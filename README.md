# Home Assistant – ha-blackhawk-deercreek

**Owner:** Laurie (Blackhawk, CA)  
This repository tracks the Home Assistant `/config` directory for this site.

---

## 🧩 Overview

This Home Assistant setup automates motion-based lighting, voice announcements, and direction detection for multiple rooms using Zigbee and local TTS audio.

It includes:
- PIR-based automation for **Atrium** and **Staircase**.
- Voice feedback via **Google Translate TTS** played through the **Yamaha RX-A770** (Family Room speakers).
- Multi-sensor logic for detecting **direction of travel** on the stairs.
- Configuration files stored safely in GitHub for version control.

---

## ⚙️ Hardware / Network

| Component | Description |
|------------|--------------|
| **Host** | Raspberry Pi 4 running Home Assistant OS |
| **Audio Receiver** | Yamaha RX-A770 (Family Room) |
| **Speakers** | Family Room ceiling speakers |
| **Zigbee Coordinator** | SONOFF Dongle Plus MG24 (EFR32MG24) |
| **Thread/Matter** | Planned – MG24 firmware upgrade when stable |
| **Network** | 10.0.0.x local subnet (wired Ethernet) |

---

## 🏠 Motion Sensors Layout

### 1. **Atrium PIR**
- **Entity:** `binary_sensor.atrium_pir_occupancy`
- Controls:
  - `light.man_cave_ceiling_lights`
  - Announces when lights turn on/off.
- Automation:
  - Turns lights **on** immediately with motion.
  - Turns lights **off** after 1 minute of no motion.
  - Uses script `script.atrium_lights_on_reasoned` to speak reason.
- Example TTS message:
  > “Lights on due to motion in the atrium.”

---

### 2. **Stair PIR (Upper + Lower)**  
Used together to determine **direction of travel**.

| Sensor | Entity ID | Location | Role |
|---------|------------|----------|------|
| **Upper PIR** | `binary_sensor.stairs_motion_upper` | Top of stairs | Detects movement toward/away from top |
| **Lower PIR** | `binary_sensor.stairs_motion_lower` | Bottom of stairs | Detects movement toward/away from bottom |

**Logic:**
- **Downstairs:** Upper PIR triggers **before** Lower → “Heading Down.”
- **Upstairs:** Lower PIR triggers **before** Upper → “Heading Up.”

**Automations:**
- `Stairs UP → DOWN announce (Family Room)`  
  → Announces: “Someone is heading downstairs.”
- `Stairs DOWN → UP speak to (Family Room) audio`  
  → Announces: “Motion detected heading upstairs.”

Both use `tts.google_translate_en_com` → `media_player.family_room`.

---

## 🗣️ Voice / Audio Configuration

| Setting | Value |
|----------|--------|
| **TTS Integration** | Google Translate (`tts.google_translate_en_com`) |
| **Output Player** | `media_player.family_room` (Yamaha RX-A770 Zone 1) |
| **Announcement Source** | Home Assistant → Family Room ceiling speakers |
| **TTS Format** | MP3 (cached in `/config/tts/`) |

Sample action:
```yaml
action: tts.speak
target:
  entity_id: tts.google_translate_en_com
data:
  media_player_entity_id: media_player.family_room
  message: "Test from Home Assistant"

