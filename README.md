# Parking Distance Meter v4

This project is an intelligent ultrasonic parking sensor built with a WEMOS D1 Mini, an HC-SR04 ultrasonic sensor, a TM1637 7-segment display, and a relay. It is designed to provide accurate distance measurements and intelligently control a connected device (e.g., a light or buzzer) via the relay.

## Hardware

*   **Microcontroller:** WEMOS D1 Mini (ESP8266)
*   **Distance Sensor:** [HC-SR04 Ultrasonic Sensor](https://esphome.io/components/sensor/ultrasonic.html)
*   **Display:** [TM1637 7-Segment Display](https://esphome.io/components/display/tm1637.html)
*   **Switch:** A 5V Relay Module

## ESPHome Configuration Explained

This project is configured using a single YAML file for [ESPHome](https://esphome.io/).

### Core Components

*   **`substitutions`**: This section defines variables for GPIO pins and other settings, making the configuration clean and easy to modify.
*   **`esp8266`**: Configures the project for the WEMOS D1 Mini board.
*   **`api`**, **`ota`**, **`wifi`**: Standard ESPHome components for connecting to Home Assistant, enabling over-the-air updates, and connecting to your Wi-Fi network.

### Sensor: Ultrasonic Distance (`HC-SR04`)

This is the primary input for the system.

*   **`platform: ultrasonic`**: Defines the sensor type.
*   **`update_interval: 0.15s`**: The sensor takes a new distance measurement every 150 milliseconds, ensuring high responsiveness.
*   **`filters`**: A `multiply: 100` filter converts the sensor's output from meters to centimeters.
*   **`on_value`**: This is the core automation block that runs every time the sensor produces a new value. It contains the logic for the intelligent relay control.

### Display: 7-Segment (`TM1637`)

This provides visual feedback to the user.

*   **`platform: tm1637`**: Defines the display type.
*   **`update_interval: 0.15s`**: The display refreshes at the same rate as the sensor, ensuring the displayed value is always current.
*   **`lambda`**: A small C++ script controls what is shown on the display. It checks if the sensor reading is valid and within the 150cm range. If it is, the distance is displayed; otherwise, it shows "----" to indicate an out-of-range or invalid reading.

### Switch: Relay Control

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
