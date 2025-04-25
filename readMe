# Bubble Memo Clock Documentation

## Overview

**Bubble Memo Clock** is a hardware extension of my **Bubble Memo App**, transforming digital task reminders into a physical, interactive experience. The device allows users to interact with their to-do list through a tactile interface, using light and sound feedback to guide them through their tasks.

---

## Concept

- The design stems from the need to make reminders more **intuitive**, **tangible**, and **playful**.
- Users interact via:
  - **Knob (Angle Sensor)**: to scroll through tasks.
  - **Button**: to confirm task completion.
  - **LED Ring**: provides soft visual feedback.
- The clock cycles through four states:
  - **Idle**
  - **Select**
  - **Remind**
  - **Complete**

---

## Design Iterations

### Hardware

1. **First Iteration**:
   - Tried 3D printing the case.
   - Faced issues: material fragility, complexity in print design.
   - Decision: drop 3D printing.

2. **Second Iteration**:
   - Chose **handcrafting** for flexibility and better control.
   - LED strip integrated, approx. **2cm width**.
   - Overall aesthetic kept in line with **Bubble Memo App** visuals.

   **Note**: Insert image of hardware prototype here.

### Software (Code)

1. **Original Logic**:
   - Basic state transitions based on knob rotation and button press.
   - LED colors less distinct.
   - Notification logic was simpler.

2. **Refined Logic**:
   - Updated LED colors: **higher brightness**, **lower saturation**, more **pastel-like**.
   - Added flashing effect for notifications.
   - Improved debounce and ADC deadband logic.
   - Enhanced timer handling for **REMIND** and **COMPLETE** states.

---

## User Flow

### Refined Flowchart:
1. Start in **Idle** (shows time, waiting for knob input).
2. Knob touched → enter **Select** (choose task).
3. If no button press in 10s → enter **Remind**, LED off, flash for notification.
4. Button pressed → **Complete**, LED turns orange.
5. If no button press in 5s in **Complete** → return to **Select**.
6. Button press again → back to **Idle**.

**Note**: Insert flowchart images here.

---

## Full Code

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


Reflections & Future Opportunities
Opportunity for Better Integration:

Add infrared sensors to detect user proximity (suggested during critique).

Trigger dynamic effects based on user interaction beyond button/knob.

Explore better handcraft techniques or light diffusion materials.

Learning Points:

Moving from 3D printing to handcrafting opened more flexibility.

Importance of color tuning for emotional design.

State management in hardware systems mirrors app logic, but with unique physical challenges.
