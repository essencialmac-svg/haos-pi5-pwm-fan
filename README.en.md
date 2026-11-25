# 🌀 Quiet PWM fan for Raspberry Pi 5 on Home Assistant OS

[Versión en español](README.md) · [![Donate with PayPal](https://img.shields.io/badge/Donate-PayPal-00457C?logo=paypal)](https://paypal.me/AntonioMV)

Drive a **5 V PWM fan** (e.g. **Noctua NF-A6x15 5V PWM**) wired to the GPIO header of a
**Raspberry Pi 5** running **Home Assistant OS**, based on CPU temperature.

- ✅ **One `config.txt` edit**: no add-on, no HACS, no pigpio, no scripts.
- ✅ Handled by **firmware/kernel**: keeps working while Home Assistant is stopped or restarting.
- ✅ **Cool and quiet** tested curve.
- ✅ Backup and **1-minute rollback** (even if the Pi won't boot).
- ✅ Dashboard card for CPU temperature.

> Using the official Pi 5 Active Cooler (FAN connector)? Firmware already handles it; this guide is for
> fans wired to the **GPIO pins**.

## 1. Requirements
Raspberry Pi 5 · Home Assistant OS (SD/SSD/NVMe) · 4-pin **5 V PWM** fan ·
[Advanced SSH & Web Terminal](https://github.com/hassio-addons/addon-ssh) add-on.

## 2. Wiring

| Noctua wire | Function | Physical pin |
|---|---|---|
| 🟡 Yellow | +5 V | **2** |
| ⚫ Black | GND | **6** |
| 🔵 Blue | PWM | **12** (GPIO18) |
| 🟢 Green | Tach | 16 (optional, unused) |

Diagram: [docs/wiring.md](docs/wiring.md).

## 3. Step-by-step install

### 3.1 Terminal
1. **Settings → Add-ons → Store** → install **Advanced SSH & Web Terminal**, set a password or SSH key in **Configuration**.
2. In **Info**, **turn off "Protection mode"** and **Restart** (needed to reach the boot partition — you'll turn it back on at the end).
3. **Open Web UI**.

### 3.2 Back up `config.txt`
```sh
docker run --rm -v /mnt/boot:/boot alpine sh -c 'cp -n /boot/config.txt /boot/config.txt.bak && ls -l /boot/config.txt*'
```

### 3.3 Add the fan
```sh
docker run --rm -v /mnt/boot:/boot alpine sh -c 'grep -q pwm-gpio-fan /boot/config.txt && echo "ALREADY PRESENT, nothing changed" || printf "%s\n" \
"" \
"# 5V PWM fan on GPIO18" \
"dtoverlay=pwm-gpio-fan,fan_gpio=18" \
"dtparam=fan_temp0=40000,fan_temp0_speed=100" \
"dtparam=fan_temp1=50000,fan_temp1_speed=140" \
"dtparam=fan_temp2=60000,fan_temp2_speed=190" \
"dtparam=fan_temp3=70000,fan_temp3_speed=255" >> /boot/config.txt; sync; tail -n 8 /boot/config.txt'
```

### 3.4 Reboot
```sh
ha host reboot
```

### 3.5 Check
```sh
echo "CPU: $(cat /sys/class/thermal/thermal_zone0/temp)"; for h in /sys/class/hwmon/hwmon*; do [ -e $h/pwm1 ] && echo "$(basename $(readlink -f $h/device)): pwm1=$(cat $h/pwm1)"; done
```
The **`pwm-fan`** line is your fan: **0** below 40 °C, **100+** above. `cooling_fan` is the official connector.

### 3.6 Turn "Protection mode" back on ✅

## 4. Curve

| CPU temp | Speed | Steps down below |
|---|---|---|
| < 40 °C | off | — |
| 40 °C | 39 % (100) | 35 °C |
| 50 °C | 55 % (140) | 45 °C |
| 60 °C | 75 % (190) | 55 °C |
| 70 °C | 100 % (255) | 65 °C |

Edit `fan_tempN` (millidegrees) and `fan_tempN_speed` (0–255). Optional `fan_tempN_hyst` (default 5000).

### ⚠️ Why several `dtparam=` lines
The Pi firmware **silently ignores anything past 98 characters on a `config.txt` line**. Put every
parameter on one `dtoverlay=` line and the last ones are dropped, falling back to the overlay defaults
(55/60/67.5/75 °C). `dtparam=` lines right below a `dtoverlay=` apply to that overlay — **don't put
another `dtoverlay=` in between**.

## 5. Dashboard card
Enable **System Monitor** → *Processor temperature* sensor, then add a **Manual** card with
[lovelace/cpu-temp-card.yaml](lovelace/cpu-temp-card.yaml) (needs [apexcharts-card](https://github.com/RomRider/apexcharts-card))
or [lovelace/cpu-temp-card-basic.yaml](lovelace/cpu-temp-card-basic.yaml) (no HACS).

## 6. Rollback
Pi running (protection mode off):
```sh
docker run --rm -v /mnt/boot:/boot alpine sh -c 'cp /boot/config.txt.bak /boot/config.txt && sync' && ha host reboot
```
Pi won't boot: plug the SD/SSD into a PC, open the **`hassos-boot`** drive (FAT), rename
`config.txt.bak` to `config.txt`. 🚫 Never format or delete partitions.

## 7. Troubleshooting

| Symptom | Likely cause |
|---|---|
| docker permission denied | Protection mode still on, or add-on not restarted. |
| Fan never spins | Blue wire on wrong pin, CPU below 40 °C, or 12 V fan. |
| Always full speed | PWM (blue) wire missing / not on pin 12. |
| Curve not applied | Line longer than 98 chars? Another `dtoverlay=` in between? |
| Terminal shows "403: Forbidden" after reboot | Stale session: open another HA page, then the terminal again. |
| Hum at low speed | Overlay uses 50 Hz software PWM. Raise `fan_temp0_speed` (e.g. 120). |

## 8. Credits
[`pwm-gpio-fan`](https://github.com/herbingk/pwm-gpio-fan) overlay by herbingk, shipped in the official Raspberry Pi firmware.
Tested on Raspberry Pi 5 + Home Assistant OS booting from SSD with a Noctua NF-A6x15 5V PWM.

## ☕ Support
If this saved you time: **[paypal.me/AntonioMV](https://paypal.me/AntonioMV)** — thank you!

[MIT](LICENSE) license.
