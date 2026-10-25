# Connected pet feeder

A kibble dispenser I designed, printed and wired myself, driven by an ESP8266 running
[ESPHome](https://esphome.io/) and controlled from Home Assistant. Built in November 2025.

<img src="docs/feeder.webp" alt="The printed feeder: brown and green case, hopper on top, double bowl below" width="360">

## What it does

- **Two scheduled meals a day.** The times live in Home Assistant (`input_datetime` helpers) and are
  pushed to the board, which keeps the last values in flash. If Home Assistant is down, the feeder still
  runs on its own clock (SNTP).
- **Adjustable portion.** The portion is the motor run time, set with a slider from 0 to 240 s.
- **Manual feeding** from a Home Assistant button, or from the physical button on the case.
- **Auto mode switch** to pause the schedule (holidays, vet diet), switched back on at every boot.
- **"Last feeding" sensor**, so anyone at home can see in the app whether the pet has eaten.
- Updates over Wi-Fi (OTA), local web page with a password, fallback access point if the Wi-Fi is gone.

## Hardware

| Part | Role |
|---|---|
| Wemos D1 mini (ESP8266) | runs the ESPHome firmware |
| MOSFET on `D1` | switches the DC motor of the dispenser |
| Push button on `D2` (internal pull-up, 50 ms debounce) | manual feeding |
| 3D-printed case, hopper and double bowl | my own print |

## Run it

1. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your Wi-Fi and keys
   (`secrets.yaml` is git-ignored).
2. Flash once over USB, then over the air:

   ```bash
   esphome run distributeur.yaml
   ```

3. In Home Assistant, add the device (ESPHome integration, with the API encryption key) and create two
   time-only `input_datetime` helpers: `heure_repas_matin` and `heure_repas_soir`.

Entity names are in French because the household uses the French UI
(*Distribuer maintenant* = feed now, *Mode Automatique* = auto mode, *Durée de distribution* = portion time).

## How the schedule works

Every minute the SNTP clock fires a lambda that compares the current time with the two stored meal times.
On a match, and if auto mode is on, it runs the `dispense_food` script: motor on, wait, motor off, timestamp.
The script runs in `mode: single`, so a button press during a meal cannot start a second one.
