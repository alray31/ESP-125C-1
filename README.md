# ESP-125C-1

Replace your ACM-125C-1 RF433 remote with an ESP32 + Qiachip WL102-341 transmitter module to control pool lights (skimmer or return jet) using ESPHome. This project transmits the RF signals captured from an original ACM-125C-1 remote, allowing integration with Home Assistant.

Original ACM-125C-1 remote:

<img width="230" height="406" alt="image" src="https://github.com/user-attachments/assets/b28b058f-0d4f-45b9-8b9d-f4bb4036e94f" />

Used to control skimmer light:

<img width="270" height="182" alt="image" src="https://github.com/user-attachments/assets/b23a936e-f089-4bd3-bd39-f6049aea3096" />

And/or return jet light:

<img width="270" height="271" alt="image" src="https://github.com/user-attachments/assets/bb05db57-c002-4275-9b05-a271657ca118" />

---

## Part List

Here’s what you’ll need to replicate this project:

| Quantity | Component | Notes |
|----------|-----------|-------|
| 1        | ESP32 Dev Board | Any ESP32 board compatible with ESPHome |
| 1        | Qiachip WL102-341 RF433 Transmitter Module | To transmit RF 433.92 MHz codes |
| 1        | Jumper Wires | For connections between ESP32 and transmitter |
| 1        | USB Cable | To power and flash the ESP32 |
| Optional | 3D-printed enclosure | To house the electronics safely |

ESP32 Dev board:  
<img width="273" height="272" alt="image" src="https://github.com/user-attachments/assets/2b5acb89-069d-4b25-9b7b-e92e0d80850e" />

RF433 Transmitter:  
<img width="181" height="172" alt="image" src="https://github.com/user-attachments/assets/65407214-096f-4e60-aab9-93035d3a9f96" />

---
## Wiring diagram

<img width="584" height="547" alt="image" src="https://github.com/user-attachments/assets/619c494c-9e98-438e-abf1-fa1421785298" />

note: EN contact from RF transmitter is not used

---

## Proof of Concept

This project has been tested and successfully transmits the captured RF signals from an ACM-125C-1 remote. The ESP32, running ESPHome, can toggle the pool lights (skimmer or return jet) from Home Assistant.  

Steps verified in proof of concept:

1. Captured the RF codes from the original ACM-125C-1 remote using an RTL-SDR dongle.  
2. Analyzed captured codes using Universal Radio Hacker software.  
3. Programmed the ESP32 using ESPHome to transmit these codes.  
4. Verified successful control of pool lights through Home Assistant.  
5. Tested repeated transmissions for reliability.

---

## Code Analysis

The ACM-125C-1 remote transmits RF signals as a pulse train. The structure is as follows:

1. **Leading Pulse** – An initial pulse signals the start of the transmission (preamble). The pulse length is consistent for all commands except the On/Off commands.  
2. **24-bit Data Transmission** – The actual command is encoded as a 24-bit sequence of high and low pulses.  
3. **Repeated Transmission** – The 24-bit signal is repeated 10 times for all commands, except the pair command, which is repeated 110 times.  

Typical pulse train:  
<img width="874" height="149" alt="image" src="https://github.com/user-attachments/assets/ace97d7a-4200-41c9-8f98-026032245a89" />

Typical 24-bit sequence:  
<img width="958" height="155" alt="image" src="https://github.com/user-attachments/assets/4625414a-4c45-4003-9139-8389a9c9b14c" />

This structure ensures the receiver correctly interprets the command, even in a noisy RF environment. The ESP32 replicates this pattern for accurate control of the pool light(s).

---

## Installation

1. Connect the Qiachip WL102-341 transmitter to the ESP32 following the wiring diagram.  
2. Flash the ESP32 with the ESPHome configuration from this repo.  
3. Add the ESP32 as a device in Home Assistant.  
4. Control your pool lights using Home Assistant or automations.

---

## How to Use

The pool light acts as a receiver only. Therefore, no feedback is sent from the light to Home Assistant after a selection is made.

1. **Modes Dropdown – Replaces the yellow M and C buttons of the remote**  
   - Used to change the operation mode of the light(s).  
   - Seven modes can be selected (five dynamic modes, steady white, and steady color).  
   - Use the toggle buttons or switches to turn on/off the skimmer or return jet lights.

2. **Light Intensity or Effect Speed Dropdown – Replaces the yellow + and - buttons of the remote**  
   - In dynamic modes, this controls the speed of the scene mode (1 = slowest, 8 = fastest).  
   - Each dynamic mode remembers its last speed. For example, if "Jumping" mode is set to speed 8 and you switch to "Fading" mode at speed 5, returning to "Jumping" mode will still have speed 8.  
   - In steady white or steady color mode, this controls light intensity (1 = dimmest, 8 = brightest).

3. **On/Off Switch – Replaces the red power button of the remote**  
   - Turns the light(s) on and off.  
   - Works in optimistic mode: the switch status may not reflect the actual light state since the light does not send feedback. Like the original remote, you may need to press twice to ensure the desired command is sent.

4. **Pair Button – Replaces the simultaneous press of M and C buttons on the original remote**  
   - Used to pair the ESP to your light. If your lights are already paired to an ACM-125C-1 remote, pairing may not be required, as the light’s RF receiver already recognizes these RF codes.  
   - Pressing once momentarily simulates a long press of the M + C buttons to transmit the pairing code.  
   - For these pool lights, "pairing" simply tells the receiver: "Listen for these RF codes." Multiple remotes of the same type can operate the same light. However, pairing too many different remote types may cause the first type to be ignored. In this project, the ESP acts as an ACM-125C-1 remote, so your light will recognize it automatically.

---

## License

MIT License
