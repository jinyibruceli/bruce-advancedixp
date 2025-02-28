# Drone Impact Detection System

## Introduction
This project is a **drone impact detection system** that triggers an LED animation upon collision. The system detects collisions using **metal plates (initially copper, later switched to aluminum for increased surface area)**. When the plates make contact, they activate a programmed sequence of LED color changes.

The design includes a **large paper airplane** structure to accommodate electronic components, with the impact detection system placed at the **front** to act as a counterweight.

The development of this system went through **three iterations**, refining the impact detection mechanism and optimizing component placement.

---

## State Diagram
<img width="820" alt="截屏2025-02-28 09 25 33" src="https://github.com/user-attachments/assets/ec26b086-7776-4f0f-b8ae-f7ec867dbf79" />  
This diagram outlines the interactive behavior of the prototype, illustrating how the system transitions between different states when an impact occurs.

---

## Hardware
The project consists of the following hardware components:

- **ESP32 (or compatible microcontroller)** – Handles input from impact sensor and controls LED outputs.
- **Aluminum hitting component** – Detect physical contact and trigger the system.
- **NeoPixel LED strip (30 LEDs, connected to Pin 7)** – Displays impact animations.
- **Built-in NeoPixel LED (Pin 1)** – Provides immediate visual feedback.
- **Wires and connectors** – Ensure reliable connections between components.
- **Paper made airplane (huge)** – To contain all the components.


<img width="840" alt="截屏2025-02-28 09 46 36" src="https://github.com/user-attachments/assets/55f584c8-8efe-4717-9433-358028bbe691" />

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

 **Hiting components Attempt 1** 
 <img width="389" alt="截屏2025-02-28 09 42 36" src="https://github.com/user-attachments/assets/1f7a158d-b71b-4e29-a8bc-f9a12053d2e5" />
 I first designed a circle, and imaging to put the foil at two side of it. But it keeps slides elsewhere when the plane hit something, in this case the foil paper won't touch. So I am thinking to design a new one that increase the area of foil paper.

 **Hiting components Attempt 2** 
<img width="459" alt="截屏2025-02-28 09 31 44" src="https://github.com/user-attachments/assets/db68c61d-669c-4d03-82f1-1d272e8ea083" />
My second attempt is use the foil paper to make a stick inside a circle. I am expecting to have more success rate by increase the foil paper area. But I quickly realized that I am not able to fit this huge installation onto my plane's top. So I need to design a smaller one.

 **Hiting components Attempt 3** 
<img width="696" alt="截屏2025-02-28 09 38 22" src="https://github.com/user-attachments/assets/8680dd73-0c47-4a52-a9bd-01013c73cc48" />
<img width="334" alt="截屏2025-02-28 09 40 56" src="https://github.com/user-attachments/assets/67ad9b6b-9f98-47df-a490-1c5f4da42a2c" />
In this version, finally the success rate comes up and it work as expected. It will touch while hit something, and the size is acceptable.

---

## Project Outcome
This system successfully detects impacts and provides clear visual feedback via an LED animation. The design has gone through multiple refinements to ensure reliability and robustness.

Final demo video:
(https://drive.google.com/file/d/130Ylu_2bFULq6-qVu4J6zWQZxDL2UjYT/view?usp=sharing)

