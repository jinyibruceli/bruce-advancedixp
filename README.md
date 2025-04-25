# Bubble Memo Clock Documentation

## Introduction

**Bubble Memo Clock** is a handcrafted interactive hardware inspired by the **Bubble Memo App**, extending the digital experience of task management into a tangible, playful form. This clock functions not just as a timepiece but as an **interactive reminder system**, blending light, touch, and sound to encourage users to complete tasks. 

It evolved through multiple **design** and **code iterations**, refining both the **form** and **behavior** of the device. The clock allows users to input a to-do list, interact with tasks using a knob and button, and receive reminders via light animations.
<img width="1470" alt="截屏2025-04-25 12 46 55" src="https://github.com/user-attachments/assets/91222a67-4bf8-43c8-aa80-802a93f58f25" />

---

## Project Timeline & Iterations

### 1. Concept Formation

- Origin: Based on the **Bubble Memo App** — a soft, friendly UI for task reminders.
- Idea: Extend this interaction to a **physical object** using LED lights, sensors, and manual input (knob, button).

### 2. First Hardware Prototype (3D Printed)

- Attempted to create a **3D printed shell**.
- Problems:
  - Fragile material.
  - Inaccurate prints, poor fit for electronics.
  - Difficulties with integrating LED strip cleanly.
- Result: Abandoned 3D printing.
![421744780570_ pic](https://github.com/user-attachments/assets/d1f2b0cb-92c2-495b-80b3-1766a7ef7d3a)
![image](https://github.com/user-attachments/assets/0ffc215e-6d3e-49de-86e4-99a3341e6736)
<img width="612" alt="截屏2025-04-25 12 37 56" src="https://github.com/user-attachments/assets/bb9c8510-fcbe-42c8-a7e8-d49e19391847" />

### 3. Second Hardware Prototype (Handcrafted)

- Switched to **handmade construction** for flexibility.
- Used **2cm LED strip** around the circular edge.
- Maintained Bubble Memo's **playful aesthetic**.

<img width="458" alt="截屏2025-04-25 12 48 09" src="https://github.com/user-attachments/assets/953c97ac-b629-4b0d-af6e-091283f4fdf0" />

![IMG_6411](https://github.com/user-attachments/assets/8dc4543e-e9e4-4c7a-b5c1-96cf18893bfc)
process testing demo video:
https://drive.google.com/file/d/1bsgtXWc7cvYt24mycBALA3lmW_aHk7Q8/view?usp=drive_link

---

## Functional Overview

- **Input**: To-do list provided via interface (linked to app conceptually).
- **Interaction**:
  - **Knob (Angle Sensor)**: Select specific task zones (mapped 1-5).
  - **Button**: Confirm or complete tasks.
- **Feedback**:
  - **LED Visuals**: Color-coded states.
  - **Flashing Notifications**: For reminders.
  - **Sound Effects** (Planned): In future iterations.
- **Idle Mode**: Shows time, monitors light level (day/night detection).

---

## State Machine & Logic Updates

### States:
1. **Idle**:
   - Clock shows time.
   - Light sensor detects day/night (prints 'light' or 'dark').
   - LED stays **pink**.
   - Any knob movement triggers **Select** state.

2. **Select**:
   - User selects tasks (1-5) by rotating knob.
   - LED turns **light green**.
   - Press button → enter **Remind**.

3. **Remind**:
   - LED turns off.
   - Every 10s → flash **light blue**, print 'notification'.
   - Button press → enter **Complete**.

4. **Complete**:
   - LED **light orange**.
   - If no interaction for **5s** → return to **Select**.
   - Button press → return to **Idle**.

---

### Updated Logic Compared to First Version:
- Added **Idle state** with **light detection** (day/night).
- Refined LED colors: **higher brightness**, **lower saturation**, **softer tones**.
- **Notification now flashes twice**, instead of steady light.
- Added **auto-return** logic:
  - Remind: re-triggers every 10s until user responds.
  - Complete: auto-back to Select after 5s idle.

---

## Flowchart Evolution

### First Version:
- No **Idle** state, task flow started from **Select**.
- **Notification** was just a print, no flash.
- **Complete** didn’t auto-return.

### Refined Version:
- Added **Idle** as default.
- Added **light sensor interaction**.
- Improved transitions and visual feedback.

<img width="1494" alt="截屏2025-04-25 12 22 44" src="https://github.com/user-attachments/assets/0ee8b308-95bb-487c-ac2f-c7e66653b75d" />

---

## Full Code (Updated)

```python
from machine import Pin, ADC
from neopixel import NeoPixel
import time, utime

# ───────────────────────────────────────────────
# 1. NeoPixel Setup
# ───────────────────────────────────────────────
LED_PIN   = 7
LED_COUNT = 30
LED       = NeoPixel(Pin(LED_PIN, Pin.OUT), LED_COUNT)

# ───────────────────────────────────────────────
# 2. Hardware Mapping
# ───────────────────────────────────────────────
ANGLE = ADC(Pin(1));  ANGLE.atten(ADC.ATTN_11DB)
LIGHT = ADC(Pin(2));  LIGHT.atten(ADC.ATTN_11DB)
BTN   = Pin(6, Pin.IN, Pin.PULL_UP)

# ───────────────────────────────────────────────
# 3. Color Palette (Soft, High Brightness, Low Saturation)
# ───────────────────────────────────────────────
COL_IDLE       = (255, 190, 210)  # Pink
COL_SELECT     = (190, 255, 190)  # Light Green
COL_COMPLETE   = (255, 235, 180)  # Light Orange (updated for better contrast)
COL_NOTIFICATION = (190, 200, 255)  # Light Blue for flashing

# ───────────────────────────────────────────────
# 4. State Machine Constants & Runtime Params
# ───────────────────────────────────────────────
IDLE, SELECT, REMIND, COMPLETE = range(4)
state            = IDLE
zone             = -1
last_btn_level   = 1
last_adc         = ANGLE.read()
DEADBAND         = 200
DAY_TH           = 1500
last_light_state = None
REMIND_INTERVAL  = 10_000
remind_timer_ms  = 0
complete_timer_ms= 0

def knob_zone(val):            # Map 0-4095 into 5 zones
    return 1 + int((val / 4095) * 5)

def set_led(rgb):              # Steady light
    LED.fill(rgb)
    LED.write()

def flash_led(rgb, times=2, on_ms=200, off_ms=200):  # Flash effect
    for _ in range(times):
        LED.fill(rgb); LED.write()
        time.sleep_ms(on_ms)
        LED.fill((0,0,0)); LED.write()
        time.sleep_ms(off_ms)

# ───────────────────────────────────────────────
# 5. Boot → Idle State
# ───────────────────────────────────────────────
set_led(COL_IDLE)
print('idle')

# ───────────────────────────────────────────────
# 6. Main Loop
# ───────────────────────────────────────────────
while True:
    now_ms  = time.ticks_ms()
    adc_val = ANGLE.read()
    zone_new = knob_zone(adc_val)

    btn_level   = BTN.value()
    btn_pressed = (last_btn_level == 1 and btn_level == 0)
    last_btn_level = btn_level

    # ───────── Idle ─────────
    if state == IDLE:
        lux = LIGHT.read()
        light_state = 'light' if lux > DAY_TH else 'dark'
        if light_state != last_light_state:
            print(light_state)
            last_light_state = light_state

        if zone_new != zone and abs(adc_val - last_adc) > DEADBAND:
            zone, state = zone_new, SELECT
            last_adc = adc_val
            set_led(COL_SELECT)
            print('select')

    # ───────── Select ─────────
    elif state == SELECT:
        if zone_new != zone and abs(adc_val - last_adc) > DEADBAND:
            zone     = zone_new
            last_adc = adc_val
            print(zone)                       # Print 1-5
        if btn_pressed:
            state = REMIND
            remind_timer_ms = now_ms
            LED.fill((0,0,0)); LED.write()   # LEDs off during remind
            print('remind')

    # ───────── Remind ─────────
    elif state == REMIND:
        if time.ticks_diff(now_ms, remind_timer_ms) >= REMIND_INTERVAL:
            print('notification')
            flash_led(COL_NOTIFICATION)      # Flash twice
            remind_timer_ms = now_ms
        if btn_pressed:
            state = COMPLETE
            complete_timer_ms = now_ms
            set_led(COL_COMPLETE)
            print('complete')

    # ───────── Complete ─────────
    elif state == COMPLETE:
        if time.ticks_diff(now_ms, complete_timer_ms) >= 5_000:
            state = SELECT
            set_led(COL_SELECT)
            print('select')
        if btn_pressed:
            state = IDLE
            zone  = -1
            set_led(COL_IDLE)
            print('idle')

    time.sleep_ms(60)


## Final outcome
![IMG_4761](https://github.com/user-attachments/assets/6eb5e163-f079-4079-9759-e656d4c2d717)
![IMG_4760](https://github.com/user-attachments/assets/642bc3f8-cb46-470c-85d1-061f1bdd3200)
final outcome video:
https://drive.google.com/file/d/1-BwaicnPxT2rzJI9xhWnfyN2QR30XX0n/view?usp=drive_link

## Future Improvements

### Infrared Sensor:
- Detect user presence, trigger lighting or sound effects.
- Enable more immersive, automatic interaction.

### Sound Integration:
- Add sound modules for more engaging notifications.

### Material Refinement:
- Use better light diffusion for LED strip.
- Refine handcraft details for durability.

---

## Reflections

- Learned importance of **iteration** in both hardware and code.
- Moving from **3D printing to handcrafting** improved design flexibility.
- Realized how **physical interaction** changes how users engage with tasks.
- This project showed me how **digital ideas** can come alive through simple, yet meaningful **embedded design**.

