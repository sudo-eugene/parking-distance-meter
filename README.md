# parking-distance-meter

## Overview
The Parking Distance Meter is an ESP8266-based device that helps with parking by displaying the distance to nearby objects. It uses an ultrasonic sensor to measure distance and displays it on a 7-segment LED display. The device automatically turns off the display when the car is out of range or when the distance has remained static for a set period.

## Hardware Requirements
- ESP8266 board (D1 Mini recommended)
- Ultrasonic distance sensor (HC-SR04 or similar)
- TM1637 7-segment LED display
- Relay module
- Power supply

## Pin Configuration
The default pin configuration in the YAML file is:
- Echo Pin: GPIO5 (D1 on D1 Mini)
- Trigger Pin: GPIO4 (D2 on D1 Mini)
- Relay Pin: GPIO14 (D5 on D1 Mini)
- CLK Pin: GPIO12 (D6 on D1 Mini)
- DIO Pin: GPIO13 (D7 on D1 Mini)

## Configuration File
The `parking-distance-meter-{id}.yaml` file is an ESPHome configuration file that defines how the device works. To use it:

1. Replace `{id}` in the filename with your unique device ID (e.g., `parking-distance-meter-k062hx.yaml`)
2. Customize the configuration parameters as needed

### Key Configuration Parameters

#### Device Identification
- `id`: Unique identifier for the device (e.g., "k062hx")
- `name`: Base name for the device (default: "parking-sensor")
- `friendly_name`: Human-readable name (default: "Parking Distance Meter v3")

#### Sensor Configuration
- `update_interval`: How often the sensor takes measurements (default: 0.15s)
- `max_distance`: Maximum valid distance in meters (default: 1.5m)
- `pulse_duration`: Duration of ultrasonic pulse (default: 20μs)

#### Display Configuration
- `display_update_interval`: How often the display updates (default: 0.15s)

#### Relay Configuration
- `debounce_time`: Time in milliseconds to debounce sensor readings (default: 1000ms)
- `static_timeout`: Time in milliseconds after which the display turns off if the distance remains static (default: 300000ms = 5 minutes)
- `static_diff`: Minimum difference in cm to consider the distance as changed (default: 2.0cm)

#### Network Configuration
- `wifi_fallback_password`: Password for the fallback AP mode
- `ota_password`: Password for OTA updates
- `encryption_key`: Key for encrypted communication with Home Assistant

## How It Works

### Distance Measurement
The device uses an ultrasonic sensor to measure the distance to objects. The measurements are:
- Taken at regular intervals (defined by `update_interval`)
- Filtered using a median filter to reduce noise
- Converted from meters to centimeters for display

### Display Logic
The 7-segment display shows the current distance in centimeters. The display behavior is controlled by several mechanisms:

1. **Maximum Distance Threshold**:
   - When the measured distance exceeds `max_distance`, the display turns off after `debounce_time` milliseconds
   - When the distance returns below `max_distance`, the display turns back on

2. **Static Distance Detection**:
   - The device tracks if the distance remains stable (changes less than `static_diff` cm)
   - If the distance remains static for longer than `static_timeout` (5 minutes by default), the display turns off
   - Any significant movement resets this timer and turns the display back on

### Network Connectivity
The device connects to your home network and integrates with Home Assistant:
- Supports OTA updates
- Provides a web server interface
- Creates a fallback access point if WiFi connection fails
- Integrates with Home Assistant via the ESPHome API

## Home Assistant Integration
The device exposes:
- A sensor entity for the measured distance
- A switch entity to control the relay/display manually

## Customization
To customize the device for your needs, modify the substitution variables at the top of the YAML file before flashing it to your ESP8266 device.