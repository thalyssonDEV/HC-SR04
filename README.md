# HC-SR04 Ultrasonic Sensor

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

## Example Code
Below is the complete example code provided in this repository. It includes functions to control the sensor, measure the echo duration, calculate distance, and display the results through a serial terminal.

```c
#include <stdio.h>
#include "pico/stdlib.h"

// Hardware pin definitions
#define TRIG_PIN 17           /**< Trigger pin of the ultrasonic sensor */
#define ECHO_PIN 16           /**< Echo pin of the ultrasonic sensor */

/**
 * @brief Controls the TRIG pin of the ultrasonic sensor.
 * 
 * This function sets the TRIG pin to either HIGH or LOW, 
 * enabling the sensor to send trigger pulses.
 * 
 * @param state The state to be set for the pin (HIGH or LOW).
 */
void write_trig_pin(bool state) {
    gpio_put(TRIG_PIN, state);  /**< Sends the state to the TRIG pin */
}

/**
 * @brief Reads the distance measured by the ultrasonic sensor.
 * 
 * This function emits a trigger pulse, waits for the echo signal, 
 * and calculates the distance based on the time taken for the echo 
 * to return, using the speed of sound formula. A timeout mechanism 
 * is included to prevent infinite loops in case of no echo detection.
 * 
 * @return The measured distance in centimeters. Returns -1.0 if no 
 * valid measurement is obtained (timeout or out-of-range reading).
 */
float get_reading_sensor() {
    const uint32_t timeout = 38000; /**< Timeout threshold to prevent infinite loops (~65ms -> ~4m max range) */

    // Trigger pulse to initiate measurement
    write_trig_pin(0);
    sleep_us(2);
    write_trig_pin(1);
    sleep_us(10);
    write_trig_pin(0);

    // Wait for the echo signal to start, with timeout
    uint32_t start_time = time_us_32();
    while (!gpio_get(ECHO_PIN)) {
        if ((time_us_32() - start_time) > timeout) {
            return -1.0;  /**< Return -1 to indicate timeout (no object detected) */
        }
    }

    // Capture the start time
    absolute_time_t start = get_absolute_time();

    // Wait for the echo signal to end, with timeout
    start_time = time_us_32();
    while (gpio_get(ECHO_PIN)) {
        if ((time_us_32() - start_time) > timeout) {
            return -1.0;  /**< Return -1 to indicate timeout */
        }
    }

    // Capture the end time
    absolute_time_t end = get_absolute_time();

    // Compute the duration of the echo pulse
    int64_t duration = absolute_time_diff_us(start, end);

    // Convert duration to distance in cm
    float distance = duration / 58.0;

    // Optional: Ignore unrealistic readings (e.g., too close or too far)
    if (distance < 1.0 || distance > 400.0) {
        return -1.0;  /**< Invalid reading */
    }

    return distance;
}

/**
 * @brief Main function to initialize the Pico and run the ultrasonic sensor.
 * 
 * This function initializes the GPIO pins, configures the TRIG pin as an output 
 * and the ECHO pin as an input, and then enters an infinite loop to continuously 
 * read and display the measured distance.
 * 
 * @return This function does not return (infinite loop).
 */
int main() {
    stdio_init_all();
    sleep_ms(200); /**< Ensures enough time for the serial interface to initialize */

    // Initialize GPIO pins
    gpio_init(TRIG_PIN);
    gpio_set_dir(TRIG_PIN, GPIO_OUT);
    gpio_put(TRIG_PIN, 0);

    gpio_init(ECHO_PIN);
    gpio_set_dir(ECHO_PIN, GPIO_IN);

    printf("Ultrasonic Sensor HC-SR04\n");

    while (true) {
        float distance = get_reading_sensor();  /**< Retrieves the distance measurement from the sensor */
        printf("Distance: %.2f cm\n", distance);  /**< Displays the measured distance */
        sleep_ms(1000);  /**< Waits 1 second before the next reading */
    }
}
```
## How to Use

### Setup Hardware
- Wire the HC-SR04 sensor to the Raspberry Pi Pico according to the pin configuration above.
- Ensure the sensor's power supply is correctly connected.

### Install Pico SDK
- Follow the official [Raspberry Pi Pico SDK documentation](https://www.raspberrypi.com/documentation/microcontrollers/c_sdk.html) to set up your development environment.

### Build the Project
- Place the code in your project directory.
- Create a CMake configuration and build the project following the Pico SDK guidelines.

## Additional Information

### Timeout Mechanism
- The `get_reading_sensor` function includes a timeout mechanism to prevent infinite loops if no echo is received. This ensures that the program remains responsive even if the sensor does not detect an object.

### Range and Accuracy
- The HC-SR04 sensor typically works within a range of 1 cm to 400 cm. The code filters out readings outside this range as invalid.


### Flash the Firmware
- Once built, flash the firmware onto the Pico using the appropriate tools (e.g., `picotool` or drag-and-drop programming).

### Monitor the Output
- Open a serial terminal (e.g., PuTTY, minicom, or screen) to view the distance measurements being printed every second.
