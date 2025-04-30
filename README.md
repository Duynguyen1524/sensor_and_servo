# 🔧 Sensor and Servo Control System

This embedded system project demonstrates real-time control of a servo motor and an LED based on environmental sensor input — specifically **temperature** and **light**. The project showcases sensor integration and actuator response in a reactive IoT-like setup.

## 🧠 Project Summary

### Behavior Overview

- A **temperature sensor** controls the **servo motor**.
- A **light sensor (e.g., photoresistor)** controls the **LED**.
- When **temperature increases**, the **servo rotates**.
- When **ambient light is low**, the **LED turns on**.
- If **heat increases and then light drops**, **both actions occur sequentially**:
  - The **servo turns**
  - Then the **LED turns on** while the **servo rotates again** for confirmation

This models a scenario like a fire hazard system: movement happens with rising heat, and lighting turns on if it’s dark.

---

## 🛠️ Components Used

- Microcontroller: [e.g., STM32F4 / Arduino Uno / MSP432]
- Temperature Sensor: e.g., LM35 or DHT11
- Light Sensor: Photoresistor (LDR)
- Servo Motor: SG90 or equivalent
- LED: Standard red/green
- Resistors (10kΩ for LDR voltage divider, 220Ω for LED)
- Power Supply / USB
- Breadboard and jumper wires

---

## ⚙️ System Logic Flow

```text
If Temperature > Threshold:
    Turn Servo

If Light < Threshold:
    Turn LED ON

If Temperature > Threshold AND then Light < Threshold:
    Turn Servo
    Turn LED ON
    Turn Servo again (confirmation/motion response)
