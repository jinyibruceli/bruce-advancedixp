# Drone Impact Detection System

## Introduction
This project is a **drone impact detection system** that triggers an LED animation upon collision. The system detects collisions using **metal plates (initially copper, later switched to aluminum for increased surface area)**. When the plates make contact, they activate a programmed sequence of LED color changes.

The design includes a **large paper airplane** structure to accommodate electronic components, with the impact detection system placed at the **front** to act as a counterweight.

The development of this system went through **three iterations**, refining the impact detection mechanism and optimizing component placement.

---

## State Diagram
_(Insert state diagram here)_

This diagram outlines the interactive behavior of the prototype, illustrating how the system transitions between different states when an impact occurs.

---

## Hardware
The project consists of the following hardware components:

- **ESP32 (or compatible microcontroller)** – Handles input from impact sensor and controls LED outputs.
- **Aluminum impact plates** – Detect physical contact and trigger the system.
- **NeoPixel LED strip (30 LEDs, connected to Pin 7)** – Displays impact animations.
- **Built-in NeoPixel LED (Pin 1)** – Provides immediate visual feedback.
- **Wires and connectors** – Ensure reliable connections between components.

_(Include wiring diagram/image here)_

---

## Firmware
Below is the MicroPython code running on the microcontroller:

```python
from machine import Pin, ADC
from time import sleep, sleep_ms, ticks_ms
from neopixel import NeoPixel

# Configure impact sensor input
touch_pin = Pin(1, Pin.IN, Pin.PULL_UP)

# Configure LED outputs
np = NeoPixel(Pin(35), 1)   # Built-in LED
np7 = NeoPixel(Pin(7), 30)  # External LED strip

# Define colors
GREEN = (0, 255, 0)   # Default state
RED = (255, 0, 0)     # Impact detected
OFF = (0, 0, 0)       # LEDs off

# Function to set all LEDs
def set_all_leds(color):
    np[0] = color
    np.write()
    for i in range(30):
        np7[i] = color
    np7.write()

# Default: Green
touch_state = False
set_all_leds(GREEN)

while True:
    if touch_pin.value() == 0 and not touch_state:
        touch_state = True
        set_all_leds(RED)
        sleep(2)
        for _ in range(2):
            set_all_leds(RED)
            sleep(1)
            set_all_leds(OFF)
            sleep(0.5)
        set_all_leds(RED)
    elif touch_pin.value() == 1 and touch_state:
        touch_state = False
        set_all_leds(GREEN)
    sleep_ms(10)
```

### Key Features:
- **Default green light**
- **Impact detection (plates touch)** triggers a **red light for 2 seconds**
- **Two flashes (1s on, 0.5s off)**
- **Permanent red light after flashes**
- **Resets to green when plates separate**

---

## Physical Components
- **Paper airplane frame** – Provides aerodynamic stability and supports electronics.
- **3D-printed mount (optional)** – Houses electronics and impact plates.
- **Adhesive materials (tape/glue)** – Securely attach components to the structure.

_(Include images of components & assembly process)_

---

## Project Outcome
This system successfully detects impacts and provides clear visual feedback via an LED animation. The design has gone through multiple refinements to ensure reliability and robustness.

_(Insert final prototype images)_

_(Insert video demonstration link)_

