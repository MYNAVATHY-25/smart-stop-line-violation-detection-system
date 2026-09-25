# Smart Stop-Line Violation Detection System

An Arduino-based smart traffic safety system designed to protect pedestrians at zebra crossings and automatically detect illegal stop-line violations by vehicles.

---

## 📌 Problem Statement
At traffic signals, pedestrians rely on zebra crossings to safely cross intersections. However, vehicles frequently breach the designated stop line and encroach upon the pedestrian zone while the walking signal is active. This behavior reduces safe crossing corridors, increases the likelihood of serious accidents, and leaves drivers unaware of the remaining pedestrian passage time.

---

## 🛠️ Components Required
* **Microcontroller:** Arduino UNO
* **Sensors:** 
  * HC-SR04 Ultrasonic Sensor (for pedestrian presence detection)
  * IR Sensors × 2 (for real-time stop-line vehicle tracking)
* **Visual Indicators & Display:**
  * Red LEDs × 2 (for active traffic violation visual alerts)
  * 16×2 I2C LCD Display (for countdown timer and fine allocation alerts)
* **Prototyping & Electrical:**
  * Breadboard
  * Jumper wires
  * 1Ω Resistor
* **Power:**
  * USB Cable (for programming data transmission and logic power)

---

## 📊 System Architecture & Block Diagram
```text
[ HC-SR04 Ultrasonic Sensor ] ---> ( Pedestrian Detection ) \
                                                            v
[ IR Sensors (Stop-Line)    ] ---> ( Vehicle Monitoring ) -> [ Arduino UNO ] ---> [ 16x2 I2C LCD Display ]
                                                            ^                   [ Red LEDs (Alerts)    ]
[ External Power / USB      ] -----------------------------/
```

---

## 🔄 Working Flow
1. **Pedestrian Detection:** The HC-SR04 ultrasonic sensor constantly sweeps the target zebra-crossing perimeter. Detecting a pedestrian triggers the system's safe crossing module.
2. **Countdown Timer:** The Arduino opens a **15-second crossing window**, shifting the I2C LCD to display a live countdown accessible to both crossing pedestrians and approaching motorists.
3. **Vehicle Monitoring:** While the crossing timer remains active, dual IR tracking blocks dynamically look for physical line breaches at the crosswalk boundary.
4. **Violation Detection:** If an incoming vehicle trips either IR tracking sensor before the countdown reaches zero, a system breach is registered.
5. **Warning & Penalty Indication:** 
   * Dual Red LEDs fire **ON** immediately to signal the physical violation.
   * The LCD display dynamically overrides to output a high-visibility warning notification alongside an automated fine generation warning status.

---

## 🔌 Pin Mapping Matrix

| Component Module | Component Pin | Arduino Uno Pin | Notes |
| :--- | :--- | :--- | :--- |
| **HC-SR04 Ultrasonic** | VCC | 5V | Power Supply |
| | Trig | Pin 9 | Digital Output |
| | Echo | Pin 10 | Digital Input |
| | GND | GND | Ground Reference |
| **IR Sensor 1** | OUT | Pin 2 | Stop-Line Track 1 |
| **IR Sensor 2** | OUT | Pin 3 | Stop-Line Track 2 |
| **Red LED 1** | Anode (+) | Pin 4 | Via current resistor |
| **Red LED 2** | Anode (+) | Pin 5 | Via current resistor |
| **16x2 I2C LCD** | SDA | A4 | Hardware I2C Data |
| | SCL | A5 | Hardware I2C Clock |

---

## 💻 Source Code Implementation (`System_Firmware.ino`)

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// --- Pin Configurations ---
// HC-SR04 Ultrasonic Sensor
const int TRIG_PIN = 9;
const int ECHO_PIN = 10;

// IR Sensors (Stop-Line Monitoring)
const int IR_SENSOR_1 = 2;
const int IR_SENSOR_2 = 3;

// Alert Indicators
const int RED_LED_1 = 4;
const int RED_LED_2 = 5;

// --- System Thresholds & Variables ---
const int PEDESTRIAN_DISTANCE_THRESHOLD = 50; // Capture zone in cm
const unsigned long COUNTDOWN_DURATION = 15;  // Crossing timeline window

// Initialize 16x2 LCD via I2C (Standard target address is 0x27 or 0x3F)
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(9600);

  // Initialize LCD Setup
  lcd.init();
  lcd.backlight();
  
  // Pin Configurations
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(IR_SENSOR_1, INPUT);
  pinMode(IR_SENSOR_2, INPUT);
  pinMode(RED_LED_1, OUTPUT);
  pinMode(RED_LED_2, OUTPUT);
  
  // Clear baseline alarm state
  digitalWrite(RED_LED_1, LOW);
  digitalWrite(RED_LED_2, LOW);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("System Ready...");
  delay(2000);
}

void loop() {
  // Idle State Management
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Monitoring Road");
  lcd.setCursor(0, 1);
  lcd.print("Safe Crossing");

  long distance = measureDistance();
  Serial.print("Target Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Validate pedestrian entry zone criteria
  if (distance > 0 && distance < PEDESTRIAN_DISTANCE_THRESHOLD) {
    startPedestrianCrossing();
  }

  delay(500); 
}

void startPedestrianCrossing() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Pedestrian Cross");
  
  for (int secondsLeft = COUNTDOWN_DURATION; secondsLeft >= 0; secondsLeft--) {
    lcd.setCursor(0, 1);
    lcd.print("Time Left: ");
    lcd.print(secondsLeft);
    lcd.print("s   "); 
    
    // Multi-sampling loop over a 1-second phase to catch rapid infractions
    for (int checkInterval = 0; checkInterval < 10; checkInterval++) {
      bool violation1 = (digitalRead(IR_SENSOR_1) == LOW); // LOW indicates sensor beam breach
      bool violation2 = (digitalRead(IR_SENSOR_2) == LOW);
      
      if (violation1 || violation2) {
        handleViolation();
      }
      delay(100); 
    }
  }

  // Clear system states after countdown completes
  digitalWrite(RED_LED_1, LOW);
  digitalWrite(RED_LED_2, LOW);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Crossing Ended");
  delay(2000);
}

void handleViolation() {
  digitalWrite(RED_LED_1, HIGH);
  digitalWrite(RED_LED_2, HIGH);
  Serial.println("ALARM: Violation tracked at stop-line!");
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("!! VIOLATION !!");
  lcd.setCursor(0, 1);
  lcd.print("FINE GENERATED!");
  delay(2000); 
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Pedestrian Cross");
}

long measureDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  long duration = pulseIn(ECHO_PIN, HIGH, 30000); 
  long distanceCm = duration * 0.034 / 2;
  return distanceCm;
}
```

---

## 🚀 Future Enhancements
* **Computer Vision Tracking:** Connect an onboard edge computer module (like a Raspberry Pi running OpenCV) to read license plates through automatic number plate recognition (ANPR).
* **Wireless Network Logging:** Integrate ESP8266 or ESP32 Wi-Fi microcontrollers to push infraction timestamps and vehicle profile logs straight to a localized cloud traffic server database.

---

## 📝 Conclusion
This design scales crosswalk management framework capabilities up by trading blind passive intervals for active sensor safety checks. Deploying dual stop-line crosswalk monitoring checks allows modern roadway loops to protect pedestrians while keeping lane management systems clear and operational.
