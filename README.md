# Intelligent Ultrasonic Parking Sensor

This project transforms a WEMOS D1 Mini and an HC-SR04 ultrasonic sensor into an intelligent, robust parking assistant. It provides highly accurate distance measurements on a TM1637 7-segment display and intelligently controls a relay for devices like lights or buzzers.

The system is engineered to be highly responsive while aggressively filtering out the spurious "blip" readings and sensor timeouts that commonly affect ultrasonic sensors, ensuring extremely reliable operation.

## Hardware

*   **Microcontroller:** WEMOS D1 Mini (ESP8266)
*   **Distance Sensor:** [HC-SR04 Ultrasonic Sensor](https://esphome.io/components/sensor/ultrasonic.html)
*   **Display:** [TM1637 7-Segment Display](https://esphome.io/components/display/tm1637.html)
*   **Switch:** A 5V Relay Module

## Key Features

*   **High Responsiveness:** Sensor and display update every 100 milliseconds (`0.10s`).
*   **Advanced Noise Filtering:** A multi-stage filter pipeline provides clean, stable data.
*   **Intelligent Relay Control:** The relay activates only on significant distance changes and automatically turns off after 10 seconds of inactivity.
*   **Clear Visual Feedback:** Displays distance in centimeters or "----" for out-of-range or invalid readings.
*   **Robust Error Handling:** Correctly handles sensor timeouts and out-of-range values (>1.5m).

## ESPHome Configuration Explained

### Sensor: The Filtering Pipeline

The core of this project's reliability is its multi-stage filter pipeline, which processes sensor data in four steps:

1.  **Pre-Filter (Lambda):** Catches sensor timeouts (`nan`) and converts them to a large, placeholder distance (`3.0m`). This is critical for allowing the `median` filter to work correctly.
2.  **Smoothing (Median Filter):** Takes the median of the last 7 readings. This step effectively eliminates single, random blips. For example, a single false reading of `0.5m` in a stream of `3.0m` readings will be discarded.
3.  **Cutoff (Lambda):** Enforces a hard 1.5-meter cutoff. Any value greater than `1.5` (including the `3.0` placeholder) is converted back to `nan`, signifying an "out of range" or invalid state for the rest of the system.
4.  **Conversion (Multiply):** The final, clean, in-range value is converted from meters to centimeters by multiplying by 100.

### Intelligent Relay Control (`on_value`)

The automation logic is triggered by every new, filtered sensor value:

*   **State Tracking:** Two global variables, `g_last_distance` and `g_last_reading_is_valid`, track the system's state between readings.
*   **Activation Logic:** The relay is turned ON only under two conditions:
    1.  The sensor gets its first valid reading after being invalid (e.g., a car enters the 1.5m range).
    2.  The distance changes by more than 2cm, indicating real movement.
*   **Deactivation Logic:** A 10-second `turn_off_relay_if_idle` script is started or reset every time the relay is activated. If no new valid readings occur within 10 seconds, the relay automatically turns OFF. This ensures the relay doesn't stay on indefinitely if the car leaves the sensor's range.

### Display: 7-Segment (`TM1637`)

The display provides clear, immediate feedback:

*   It shows the filtered distance in whole centimeters (e.g., " 50").
*   If the sensor state is invalid (`nan`), it displays "----". This happens when the object is out of range (>1.5m) or the sensor times out, providing unambiguous feedback to the user.

The relay is configured as a simple [GPIO Switch](https://esphome.io/components/switch/gpio.html).

*   **`inverted: true`**: This may be required depending on your relay module's logic (whether a HIGH or LOW signal turns it on).
*   **`restore_mode: RESTORE_DEFAULT_OFF`**: Ensures the relay is in a predictable OFF state when the device boots up, allowing the automation to take full control.

### Intelligent Automation Logic

To prevent the relay from activating on momentary false readings and to turn it off when idle, we use a combination of global variables, scripts, and an `on_value` automation.

*   **`globals`**: We use three [Global Variables](https://esphome.io/components/globals.html) to track the state between sensor readings:
    *   `g_last_distance`: Stores the last valid distance.
    *   `g_last_reading_is_valid`: Remembers if the previous reading was valid.
    *   `g_consecutive_valid_readings`: Counts how many valid readings have occurred in a row.

*   **`script`**: Two [Scripts](https://esphome.io/components/script.html) handle the relay actions:
    *   `turn_on_relay`: A simple script to turn the relay on.
    *   `turn_off_relay_if_idle`: A script that waits 10 seconds and then turns the relay off. This script can be restarted, which is key to the timeout logic.

*   **`on_value` Automation Logic**:
    1.  **From Invalid to Valid:** If the display was showing "----", the system requires **3 consecutive valid readings** before it will turn the relay ON. This filters out single, inaccurate "blips".
    2.  **During Valid Readings:** If the sensor is already showing a valid distance, the relay will activate instantly if the distance changes by more than 2 cm, ensuring responsiveness to genuine movement.
    3.  **Idle Timeout:** The `turn_off_relay_if_idle` script is triggered whenever the readings become stable (no significant change) or invalid. If no new activity occurs within 10 seconds, the relay turns OFF.
