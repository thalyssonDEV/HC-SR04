# HC-SR04 Ultrasonic Sensor Documentation

This repository provides comprehensive documentation and example code for the HC-SR04 ultrasonic sensor when interfaced with a Raspberry Pi Pico. The project covers the sensor's pin configuration, wiring instructions, and step-by-step guidance on utilizing the provided code to measure distances.

## Overview

The HC-SR04 sensor is an affordable ultrasonic distance sensor that measures distances by emitting a high-frequency sound pulse and timing the echo's return. This repository includes example C code for the Raspberry Pi Pico using the Pico SDK. The code:
- Initializes the required GPIO pins.
- Sends a trigger pulse.
- Waits for the echo.
- Computes the distance based on the echo duration.

## Hardware Requirements

- **Raspberry Pi Pico** (or a compatible RP2040 board)
- **HC-SR04 Ultrasonic Sensor**
- **Jumper wires** and a **breadboard** for connections
- A **power supply** (typically 5V for the sensor, with proper level shifting if required)

## Pin Configuration

The code uses the following pin definitions:

```c
// Hardware pin definitions
#define TRIG_PIN 17           /**< Trigger pin of the ultrasonic sensor */
#define ECHO_PIN 16           /**< Echo pin of the ultrasonic sensor */
```

### Connect the sensor pins as follows:

- **VCC**: Connect to 5V (or use a voltage regulator/level shifter if required for the Pico).
- **GND**: Connect to ground.
- **TRIG**: Connect to GPIO 17 on the Pico.
- **ECHO**: Connect to GPIO 16 on the Pico (with appropriate level shifting if necessary, since the Pico GPIO is 3.3V).
