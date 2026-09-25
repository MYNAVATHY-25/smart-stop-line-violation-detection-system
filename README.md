# Smart Stop-Line Violation Detection System

## Problem Statement
At traffic signals, pedestrians use zebra crossings to cross the road. However, some vehicles cross the stop line and move into the pedestrian crossing area while pedestrians are crossing. This reduces the safe space available for pedestrians and can lead to accidents. Also, drivers may not have a clear indication of how much pedestrian crossing time is remaining.

---

## Components Required
* HC-SR04 Ultrasonic Sensor
* Arduino UNO
* IR Sensor (2)
* Red LED (2)
* 16×2 I2C LCD Display
* Bread Board
* Jumper Wires
* 1 Ohm resistor
* USB cable

---

## Block Diagram
![Block Diagram](block_diagram.png)<img width="1408" height="768" alt="Block diagram" src="https://github.com/user-attachments/assets/9ac3379b-c9b6-496b-a7d4-2cde29dd98d2" />


---

## Working Flow
* **Pedestrian Detection:** The ultrasonic sensor continuously monitors the zebra-crossing area. When a pedestrian is detected, the system activates the pedestrian crossing mode.
* **Countdown Timer:** Arduino starts a 15-second countdown. The remaining time is displayed on the LCD for drivers and pedestrians.
* **Vehicle Monitoring:** During the countdown period, the IR sensor continuously monitors the stop line.
* **Stop-Line Violation Detection:** If a vehicle crosses the stop line while the countdown is active, the IR sensor detects the violation.
* **Warning Alert:** The red LED turns ON to indicate a traffic violation. A warning message is displayed on the LCD.
* **Automatic Fine Indication:** The LCD displays an automatic fine/violation message for the vehicle. In future enhancements, a camera and OCR module can be integrated for automatic number plate recognition and fine generation.

---

## Circuit Diagram
![Circuit Diagram](circuit_diagram.png)<img width="1536" height="1024" alt="Circuit diagram" src="https://github.com/user-attachments/assets/f836bfe7-17fd-40ef-aa0e-03be42364f36" />

---

## Conclusion
This project improves pedestrian safety at traffic crossings by detecting pedestrians and providing a controlled crossing time through a countdown system. It also monitors stop-line violations using sensors and alerts drivers through visual indications. By combining pedestrian detection, countdown monitoring, and violation detection, the system helps create a safer and more disciplined traffic environment.
