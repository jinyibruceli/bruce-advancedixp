# Boxing Gloves Project Summary

## Project Overview
Boxing gloves is an interactive boxing training device that integrates **M5 StickC Plus**, an **IMU sensor**, and a **real-time UI feedback system**. The gloves detects punch movements, changes LED colors dynamically, and provides **real-time feedback** to help users track their performance. The project successfully combines hardware, software, and UI into a **seamless, engaging training experience**.

---

## From Concept to Implementation

### Initial Concept & Prototyping
The original goal was to create a simple but effective **boxing training assistant** that reacts to punches with **visual indicators** and provides feedback in real-time. The prototype focused on:

- Accurately detecting punch movements  
- Providing **LED-based feedback** (color changes)  
- Implementing **smooth UI animations** for a responsive feel  
- Ensuring **hardware and software work in sync**  

The initial **state machine logic** had a straightforward flow: **blue for punch detection, green for correct position, and red when the goal was met**. However, after testing, adjustments were necessary to improve **responsiveness and accuracy**.
<img width="662" alt="截屏2025-03-21 12 57 59" src="https://github.com/user-attachments/assets/4ef9c36d-beae-4d78-981a-a5ae8c870a58" />


---

## Hardware Integration

### Components Used
- **M5 StickC Plus** – Main controller  
- **IMU Pro Unit** – Motion detection  
- **NeoPixel LED Strip** – Visual feedback  

### How It Works
1. **Punch Detection** – The gloves detects acceleration changes and turns blue when a punch is detected.  
2. **Target Position Confirmation** – If the punch reaches the correct position, the light turns green.  
3. **Counting Punches** – The system keeps track of successful punches. Once the target number is reached, the LED flashes red.  
4. **Smooth Transitions** – A **rainbow LED animation** plays when the session is complete.  

---

## Software & Code Implementation

### State Machine Logic
| **State** | **Condition** | **LED Color** |  
|------------|--------------|--------------|  
| **Idle** (waiting for punch) | Default state | Green |  
| **Punch detected** | IMU detects high acceleration | Blue |  
| **Punch completed** | Acceleration drops, punch counted | Red |  

- **Preventing False Triggers** – A **minimum time threshold** ensures only valid punches are counted.  
- **Reset Function** – Pressing a button resets the system and restores the initial green light.

```python
import os, sys, io
import M5
from M5 import *
from hardware import I2C
from hardware import Pin, ADC
from unit import IMUProUnit
from time import sleep_ms
from neopixel import NeoPixel
import m5utils  # remap function
import time
import math

M5.begin()

# Configure I2C port
i2c = I2C(0, scl=Pin(1), sda=Pin(2), freq=100000)
# Configure IMU
imu = IMUProUnit(i2c)

# Configure NeoPixel
np = NeoPixel(Pin(7), 30)

###############################################################################
# State machine:
#   state = 0 => Green  (waiting to punch / initial or reset state)
#   state = 1 => Blue   (punch in progress)
#   state = 2 => Red    (punch complete)
###############################################################################
state = 0

# Thresholds (tweak as needed)
THRESHOLD_PUNCH = 2.0       # Acceleration threshold to detect punch
THRESHOLD_RESET = 0.6       # Acceleration threshold to detect punch release
MIN_BLUE_DURATION = 500     # Minimum time in blue state (ms)
PUNCH_LIMIT = 10            # Show rainbow animation after this many punches

# Punch counter
punch_count = 0

# Flags for animation/button
stop_updates = False
reset_requested = False

# Track current LED color to reduce redundant updates
current_color = None

# Initial acceleration reading
last_accel = imu.get_accelerometer()

# Track punch start time for blue state timing
punch_start_time = 0

def set_led(color):
    """Update LEDs only when color changes to avoid flicker"""
    global current_color
    if current_color != color:
        for i in range(len(np)):
            np[i] = color
        np.write()
        current_color = color

def rainbow_animation(duration=3000):
    """Rainbow animation, interruptible by reset"""
    global stop_updates, reset_requested
    stop_updates = True
    start_time = time.ticks_ms()

    while time.ticks_diff(time.ticks_ms(), start_time) < duration:
        if reset_requested:
            return
        for i in range(len(np)):
            hue = (i * 12 + time.ticks_ms() // 10) % 360
            r = int(127.5 * (1 + math.cos(math.radians(hue))))
            g = int(127.5 * (1 + math.cos(math.radians(hue + 120))))
            b = int(127.5 * (1 + math.cos(math.radians(hue + 240))))
            np[i] = (r, g, b)
        np.write()
        sleep_ms(50)

    stop_updates = False

# Initial state: green
set_led((0, 255, 0))

while True:
    # Pause main loop if rainbow animation is running
    if stop_updates:
        continue

    # Update M5 state
    M5.update()

    # Button A: Reset
    if BtnA.wasPressed():
        punch_count = 0
        reset_requested = True    # Tell animation to stop
        stop_updates = False      # Resume main loop
        state = 0                 # Back to green
        set_led((0, 255, 0))      # Set LED to green
        print("Reset triggered!")
        continue

    # Read current acceleration
    accel = imu.get_accelerometer()
    delta_x = abs(accel[0] - last_accel[0])
    delta_y = abs(accel[1] - last_accel[1])
    delta_z = abs(accel[2] - last_accel[2])
    total_change = delta_x + delta_y + delta_z

    # State machine
    if state == 0:
        # Waiting for punch (green)
        if total_change > THRESHOLD_PUNCH:
            state = 1
            punch_start_time = time.ticks_ms()
            set_led((0, 0, 255))  # Blue
            print("Punch detected")

    elif state == 1:
        # Punch in progress (blue)
        elapsed = time.ticks_diff(time.ticks_ms(), punch_start_time)
        if elapsed >= MIN_BLUE_DURATION and total_change < THRESHOLD_RESET:
            state = 2
            punch_count += 1
            print("Punch")
            set_led((255, 0, 0))  # Red

            if punch_count >= PUNCH_LIMIT:
                print("Punch end")
                rainbow_animation()
                punch_count = 0
                print("Count reset after animation")
                set_led((255, 0, 0))  # Stay red after animation

    elif state == 2:
        # Punch complete (red)
        if total_change > THRESHOLD_PUNCH:
            state = 1
            punch_start_time = time.ticks_ms()
            set_led((0, 0, 255))  # Blue
            print("Punch detected")

    # Store last acceleration
    last_accel = accel
    sleep_ms(100)
```


---

## UI & Interaction

### UI Design & Feedback
- The **central circle shrinks when punching and expands when retracting**, creating a dynamic and **fluid animation**.  
- The **real-time interaction** makes training **more immersive** and visually responsive.  
- The UI prototype was designed in **ProtoPie**, featuring a **minimalistic, clear layout**.  
<img width="1470" alt="截屏2025-03-21 12 41 26" src="https://github.com/user-attachments/assets/e1d83a07-2249-4e5f-bccf-96fa9e3ae870" />


---

## Challenges & Solutions

### Debugging Thonny Output Issues
At first, I tried **printing variables directly in Thonny**, but the results were inconsistent. To debug, I decided to trigger **different print outputs** for each event and handled them in **ProtoPie** instead. This approach solved the issue and allowed me to manage UI updates more efficiently.

### ProtoPie Variable Issues
ProtoPie's variable management was frustrating. Initially, I kept getting **"variable not found" errors**, and I couldn’t figure out why. Later, I realized it was because I **didn’t have a premium account**, which limited access to certain features. I ended up borrowing **Juni’s account** to make it work.

---

## Final Outcome

### What Worked Well
- **Accurate punch detection** without false triggers  
- **Seamless LED transitions** synced with real-time movements  
- **UI animations** that made interactions feel smooth and engaging  
- **Adjustments to state logic** improved response times and overall experience  

Dispite the challenges, the final implementation **works smoothly**. The **real-time UI animations**, especially the dynamic circle scaling, add a polished, interactive feel to the training experience. This project shows how a simple idea can evolve into an intuitive and well-integrated system when **hardware, software, and UI are carefully designed together**.

### Video:
https://drive.google.com/file/d/1YJkU27xxd4wirRr5go8Ry3cIKph-pwnK/view?usp=sharing
