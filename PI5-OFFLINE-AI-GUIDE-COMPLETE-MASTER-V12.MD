# 🚀 STAR TREK: THE NEXT GENERATION
# Offline AI Companion – Complete Master Build Guide

**Stardate 2026.06**  
**Personal Log: Captain Ever**  
*"Computer... Make it so."*

**Version:** V12 - Complete Master Compilation  
**Build Goal:** Rugged, fully offline Star Trek-style AI companion for overlanding in a 2023 Lexus GX460 using a Pelican 1170 case.

This document is the **complete master guide** combining all previous versions, detailed instructions, code samples, hardware recommendations, and troubleshooting notes from the entire project.

---

## 1. Engineering Hardware Manifest (Final Simplified Build)

**Core Systems (Required)**
- Raspberry Pi 5 16GB
- 1TB NVMe SSD (M.2 2280, e.g. AG474E1TB-SI or similar)
- Pimoroni NVMe Base Duo
- Elecrow 8 Inch IPS HDMI Touchscreen (1280x800)
- ReSpeaker 2-Mics Pi HAT (for voice input + Majel Barrett voice output)
- GeeekPi UPS Gen 6 (with Aluminum Heatsink) **or** your preferred power solution
- Pelican 1170 Case
- Anker Prime 26,250mAh 300W Power Bank (recommended for extended runtime)

**Optional but Recommended**
- Mini Bluetooth Keyboard with Trackpad (e.g. Bewinner or Rii)
- Raspberry Pi Camera Module 3 (Autofocus) – for plant/survival identification
- Small 8Ω 3W speakers with **JST-XH 2.54mm** connectors (for ReSpeaker HAT)

**Removed from Final Build**
- Raspberry Pi AI HAT+ 2 (no longer used – running LLMs directly on Pi 5 + NVMe)

**Total Estimated Cost:** $850 – $1,050 (depending on power solution and speakers)

---

## 2. HAT Stacking Order (Simplified – No AI HAT)

**Recommended Order (Bottom → Top):**

```
ReSpeaker 2-Mics Pi HAT          ← Top (best mic placement)
          ▲
Pimoroni NVMe Base Duo           ← Storage
          ▲
Raspberry Pi 5                   ← Main Board
          ▲
GeeekPi UPS Gen 6                ← Bottom (power + safe shutdown)
```

**Standoff Recommendation:** Use **M2.5 brass standoffs 25–30mm long** (female-to-male) between each layer for proper clearance, especially around the UPS heatsink.

**Mounting Tip:** Mount the entire stack on a 6mm ABS or 1/4" acrylic plate inside the Pelican case using VHB tape or screws into the case ribs.

---

## 3. Pelican 1170 Case Assembly & Mounting

### Screen Mounting (Elecrow 8")
- Cut a window in the lid or mount internally.
- Use **pick-and-pluck foam** to create a recessed pocket.
- Secure with **3M VHB double-sided tape** + small L-brackets or corner braces.
- Back the screen with a thin piece of 1/8" or 3/16" acrylic/ABS for rigidity.

### Main Electronics Mounting
- Cut a base plate from **6mm ABS plastic** or **1/4 inch (6mm) acrylic**.
- Recommended size: ~150mm x 110mm (adjust to fit your layout).
- Mount the Pi + HAT stack on the plate using long standoffs.
- Secure the plate to the case bottom with VHB tape or small screws into the case ribs.
- Leave space for the Anker power bank in a separate foam pocket.

### Cable Routing
- Route HDMI + USB touch cables along the **hinge side**.
- Use coiled cables or add a small rubber grommet near the hinge.
- Secure all cables with Velcro or zip ties.

### Speaker Mounting
- Mount small 8Ω speakers inside the case lid or sides using hot glue or small brackets.
- Connect using **JST-XH 2.54mm** connectors to the ReSpeaker HAT’s SPEAKER terminals (LEFT and RIGHT).

---

## 4. Base System Setup (Raspberry Pi OS)

```bash
sudo apt update && sudo apt upgrade -y
sudo raspi-config
# Enable: SSH, Camera (Legacy), I2C, SPI
sudo reboot
```

---

## 5. ReSpeaker 2-Mics Pi HAT Setup (Voice Input + Output)

```bash
cd ~
git clone https://github.com/respeaker/seeed-voicecard.git
cd seeed-voicecard
sudo ./install.sh
sudo reboot
```

After reboot, test:
```bash
arecord -D hw:1,0 -f S16_LE -r 16000 -c 2 test.wav
aplay test.wav
```

**Speaker Connection:**
- Use speakers with **JST-XH 2.54mm 2-pin** connectors.
- Plug into the white **SPEAKER** terminals labeled LEFT and RIGHT on the HAT.
- Alternative: Use the 3.5mm HEADSET jack for quick testing.

---

## 6. Ollama + Local LLM Setup

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
ollama pull gemma2:2b
ollama pull qwen2.5:3b
ollama pull llava:7b          # For camera / plant identification
```

Test:
```bash
ollama run gemma2:2b "Hello, I am Captain Ever. What is the mission today?"
```

---

## 7. LCARS Interface Setup (Web-Based)

### 7.1 Install Web Server & Browser
```bash
sudo apt update
sudo apt install apache2 chromium unclutter -y
```

### 7.2 Clone & Setup Aricwithana LCARS-SDK
```bash
cd ~
git clone https://github.com/Aricwithana/LCARS-SDK.git
cd LCARS-SDK

sudo rm -rf /var/www/html/lcars
sudo mkdir -p /var/www/html/lcars
cd lcars-sdk
sudo cp -r * /var/www/html/lcars/
```

### 7.3 Create Demo index.html
```bash
cd /var/www/html/lcars
sudo nano index.html
```

**Paste the following complete demo HTML:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>LCARS - Star Trek Computer</title>
    <link rel="stylesheet" href="core/lcars.css">
    <style>
        body { background: #000000; margin: 0; padding: 20px; font-family: monospace; color: #ffcc00; }
        h1 { color: #ffcc00; text-align: center; font-size: 2.5em; }
        .lcars-panel { border: 3px solid #ffcc00; padding: 15px; margin: 10px; background: #1a1a2e; }
        .status { color: #00ffcc; }
    </style>
</head>
<body>
    <h1>COMPUTER ONLINE</h1>
    <div class="lcars-panel">
        <p class="status">Welcome, Captain Ever.</p>
        <p>Offline AI Companion Ready.</p>
        <p>Voice Command: "Computer..."</p>
    </div>
</body>
</html>
```

### 7.4 Create Kiosk Startup Script
```bash
nano ~/start-lcars.sh
```

**Paste:**
```bash
#!/bin/bash
unclutter -idle 0.5 -root &
chromium --kiosk --incognito --disable-infobars http://localhost/lcars/
```

```bash
chmod +x ~/start-lcars.sh
~/start-lcars.sh
```

---

## 8. GPS Integration (Real-time Offline Navigation)

**Recommended Hardware:** VK-162 USB GPS Dongle (U-blox chipset)

```bash
sudo apt install gpsd gpsd-clients python3-gps -y
sudo nano /etc/default/gpsd
```

Set:
```bash
START_DAEMON="true"
USBAUTO="true"
DEVICES="/dev/ttyUSB0"
GPSD_OPTIONS="-n -G"
```

```bash
sudo systemctl restart gpsd
sudo systemctl enable gpsd
cgps -s
```

Integrate with OsmAnd for live moving map on your 8" screen.

---

## 9. onX Maps + OsmAnd Offline Navigation

1. Export GPX files from onX (when online).
2. Transfer to Pi.
3. Import into OsmAnd (`~/OsmAnd/gpx/`).

**OsmAnd Setup (recommended via Waydroid for full features):**
```bash
sudo apt install waydroid -y
waydroid init
```

Pre-download large map regions before trips.

---

## 10. Kiwix Offline Wikipedia

```bash
sudo apt install kiwix-tools -y
mkdir -p ~/kiwix
```

Download `.zim` files from https://library.kiwix.org/ (recommend **Wikipedia Simple English** for size).

Run:
```bash
kiwix-serve --port=8080 ~/kiwix/wikipedia_en_simple_all_maxi.zim &
chromium http://localhost:8080
```

---

## 11. Camera + Offline Plant / Survival Identification

**Hardware:** Raspberry Pi Camera Module 3 (Autofocus)

**Setup:**
```bash
sudo raspi-config  # Enable Legacy Camera
sudo apt install libcamera-apps -y
```

**Full Python Script** (`plant_identifier.py`):

```python
#!/usr/bin/env python3
import subprocess
import requests
import time
from piper import PiperVoice

# Initialize Piper voice (Majel Barrett style)
voice = PiperVoice.load("en_US-lessac-medium.onnx")  # or your chosen voice model

def take_photo():
    subprocess.run(["libcamera-still", "-o", "/tmp/plant.jpg", "--immediate", "-t", "1"])
    return "/tmp/plant.jpg"

def analyze_plant(image_path):
    prompt = "You are a survival expert. Describe this plant in detail. Is it edible? Any medicinal uses? Any dangers or look-alikes?"
    result = subprocess.run(
        ["ollama", "run", "llava:7b", prompt],
        input=open(image_path, "rb").read(),
        capture_output=True
    )
    return result.stdout.decode()

def speak(text):
    voice.synthesize(text, wav_file="/tmp/response.wav")
    subprocess.run(["aplay", "/tmp/response.wav"])

# Main loop example
if __name__ == "__main__":
    print("Camera ready. Say 'Computer, identify this plant'")
    # Add voice trigger here using Whisper or button
    photo = take_photo()
    description = analyze_plant(photo)
    print(description)
    speak(description)
```

Run with:
```bash
python3 plant_identifier.py
```

---

## 12. NVMe SSD Formatting & Mounting

```bash
lsblk
sudo parted /dev/nvme0n1 -- mklabel gpt
sudo parted /dev/nvme0n1 -- mkpart primary ext4 1MiB 100%
sudo mkfs.ext4 -L NVMeData /dev/nvme0n1p1
sudo mkdir -p /mnt/nvme
sudo mount /dev/nvme0n1p1 /mnt/nvme
```

Add to `/etc/fstab` for auto-mount on boot (use `blkid` to get UUID).

---

## 13. Power Management

- Use **GeeekPi UPS Gen 6** for safe shutdown on low battery.
- Use **Anker Prime 26,250mAh** as hot-swap / extended runtime pack.
- Connect via high-quality USB-C PD cable.

---

## 14. Final Testing Checklist

- [ ] LCARS kiosk launches on boot
- [ ] Voice commands work ("Computer...")
- [ ] GPS shows live location in OsmAnd
- [ ] Kiwix Wikipedia loads offline
- [ ] Camera + LLaVA plant identification works
- [ ] NVMe is mounted and fast
- [ ] All cables secured for vibration (GX460 overlanding)

---

**"Live long and prosper."**  
**— Computer**

*End of Transmission – V12 Complete Master Guide*
