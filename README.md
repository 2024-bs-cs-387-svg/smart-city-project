


📦 What's inside

| File | Description |
|---|---|
| `smartcity_esp8266_v7.ino` | ESP8266 code — Weather, Water, Street Lights (3 zones), + Arduino bridge |
| `smartcity_arduino_v3.ino` | **Arduino UNO code** — Traffic (2 lanes) + Waste (2 bins) + Park (2 zones) |
| `ARDUINO_WIRING.md` | Complete step-by-step wiring guide |
| `scv3-modified/` | Website files — Netlify ready, mobile responsive |

---

## 🎯 Hardware Overview

### ESP8266 NodeMCU handles (3.3V sensors):
- 🌡️ **DHT22** — Temperature + Humidity (Pin D4)
- 💧 **YF-S201** — Water flow (Pin D2)
- ☀️ **LDR digital module** — Light sensor (Pin D3)
- 🏃 **PIR** — Motion sensor (Pin D1)
- 💡 **3 Street Light LEDs** — Zone A (D5), Zone B (D0), Zone C (D6)
- 🔗 **Serial bridge to Arduino** (D7, D8)

### Arduino UNO handles (5V sensors):
- 🚦 **4 HC-SR04** sensors (shared TRIG on Pin 2):
  - Lane A ECHO → Pin 3
  - Lane B ECHO → Pin 13
  - Bin 1 ECHO → A4
  - Bin 2 ECHO → A5
- 🔴🟡🟢 **6 Traffic LEDs** (2 intersections × 3 colors)
- 🌱 **2 Soil sensors** — Park Zone A (A0) + Zone B (A1)
- 💧 **2 Pump relays or LEDs** — Pin 9 + Pin 10

### Website controls everything:
- Real-time monitoring
- Manual override for all modules
- Auto sensor-based logic

---

## ⚠️ Important Notes

### 1. Traffic has 2 Intersections (not 4)

Website ab sirf **Lane A** aur **Lane B** dikhayegi — tumhare paas 2 HC-SR04 + 6 LEDs hain, to 2 real intersections hain.

### 2. Both Waste Bins are LIVE

**Bin 1** (Pin A4) aur **Bin 2** (Pin A5) — dono real HC-SR04 sensors. No simulation.

### 3. Relay Testing Mode (LED substitute)

Arduino code mein yeh line hai:

```cpp
#define RELAY_ACTIVE_LOW false   // ← set for LED testing
```

**Ab Pin 9 aur Pin 10 pe simple LEDs laga sakte ho** relay ki jagah:

```
Pin 9  → [1kΩ] → LED(+) → LED(−) → GND  (Pump A indicator)
Pin 10 → [1kΩ] → LED(+) → LED(−) → GND  (Pump B indicator)
```

**Jab real relay mil jaye:**
1. LEDs nikalo
2. Relay modules connect karo (VCC/GND/IN)
3. Code mein change karo: `#define RELAY_ACTIVE_LOW true`
4. Re-upload karo

---

## 💡 Street Light Logic (ESP8266)

### Per-Zone Independent Control:
- **Zone A** (D5) — LIVE with LDR + PIR sensors + manual
- **Zone B** (D0) — Manual control + AUTO sensor follow
- **Zone C** (D6) — 2 LEDs parallel, manual + AUTO
- **Zone D** — Website simulation only

### AUTO Sensor Logic:
- **Day** (sunlight) → OFF
- **Night + Motion** → ON
- **Night + No motion** → OFF

### Top "Force All" buttons broadcast to all zones.

---

## 🚦 Traffic Logic (Arduino UNO)

### AUTO Mode — Density-based signal timing:
- **Lane A green duration** = base 20s + (Lane A per-min × 2s), max 60s
- **Lane B green duration** = base 20s + (Lane B per-min × 2s), max 60s
- Traffic busy hone par automatically longer green

### Cycle:
```
Green A (15-60s) → Yellow A (3s) → Green B (15-60s) → Yellow B (3s) → repeat
```

### MANUAL Override from website:
- 🟢 Force Green Lane A / 🟢 Force Green Lane B
- 🔴 All Red
- 🚨 Emergency Lane A (5 min) / 🚨 Emergency Lane B

---

## ♻️ Waste Logic

### AUTO Detection:
- HC-SR04 distance reads bin fill percentage
- <30% = empty · 30-80% = ok · >80% = full · >90% = critical alert

### MANUAL Control:
- Per-bin "Collect" button → bin resets to 0%
- "Collect All" / "Collect Critical Only"
- Route generator for collection truck

---

## 🌱 Park Logic

### AUTO Mode (default):
- Soil < 30% → Pump automatically ON
- Soil > 75% → Pump automatically OFF

### MANUAL Control:
- Per-zone ▶ Start / ⏹ Stop button (disables auto for that zone)
- "Pumps All ON" / "All OFF"
- 🤖 AUTO toggle

---

## 🔗 ESP ↔ Arduino Serial Connection

**3 wires** required:

### Wire 1: Arduino Pin 12 ← ESP D7 (direct)
No voltage divider needed — ESP 3.3V → Arduino 5V input is safe.

### Wire 2: Arduino Pin 11 → ESP D8 (VOLTAGE DIVIDER!)

```
Arduino Pin 11 ──[1kΩ]──┬── ESP D8
                        │
                     [2kΩ]
                        │
                       GND
```

**2kΩ nahi hai?** 2 × 1kΩ series mein lagao.

### Wire 3: Common GND

```
Arduino GND ←──────→ ESP GND
```

**BILKUL ZARURI** — bina iske serial kaam nahi karegi.

---

## 🚀 Deploy Steps

### Step 1: Wiring
1. **`ARDUINO_WIRING.md`** kholo — step by step
2. Saare 11 steps follow karo
3. Test each module as you go

### Step 2: Upload Arduino code
1. **Pin 11 wire disconnect** karo ESP se (upload conflict)
2. Arduino IDE kholo
3. File → Open → `smartcity_arduino_v3.ino`
4. Tools → Board → **Arduino UNO**
5. Tools → Port → Arduino's COM port
6. Upload ⬆️
7. Serial Monitor (**9600 baud**) kholo
8. **Pin 11 wire wapas lagao** ESP D8 pe (voltage divider ke through)

### Step 3: Upload ESP code
1. File → Open → `smartcity_esp8266_v7.ino`
2. Tools → Board → **NodeMCU 1.0 (ESP-12E)**
3. Tools → Port → ESP's COM port
4. Upload ⬆️
5. Serial Monitor (**115200 baud**) kholo

### Step 4: Deploy Website
1. Update `firebase-config.js` with your Firebase credentials
2. Drag-and-drop `scv3-modified/` folder on Netlify
3. Open site in browser
4. **Ctrl + Shift + R** for hard refresh

### Step 5: Test everything
Har module test karo (detailed tests in `ARDUINO_WIRING.md`)

---

## 📡 Firebase Database Structure

```
/sensors/
  /weather/      ← DHT22 data
  /water/        ← Flow + leak status
  /street/       ← LDR, PIR, 3 zones status
  /traffic/      ← Lane A + Lane B live data
  /waste/        ← Bin 1 + Bin 2 live fill %
  /park/         ← Moisture + pump status

/commands/
  /street/
    /mode       ← on/off/auto (Zone A global)
    /zone_b     ← on/off/auto (Zone B)
    /zone_c     ← on/off/auto (Zone C)
  /traffic/
    /mode       ← auto/manual
    /signal     ← green_a/green_b/red_all/emergency_a/emergency_b
  /park/
    /mode       ← auto/manual
    /zone_a_pump ← true/false
    /zone_b_pump ← true/false
  /waste/
    /bin1_collected ← true/false
    /bin2_collected ← true/false

/system/
  /status       ← online
  /firmware     ← v7.0
  /last_seen    ← heartbeat
```

---

## 🆘 Quick Troubleshooting

| Problem | Solution |
|---|---|
| Arduino upload fails | Disconnect Pin 11 wire from ESP first |
| ESP not receiving data | Check common GND between boards |
| LED not lighting | Check polarity (+ long leg, − short leg) |
| HC-SR04 reading 0 | Check shared TRIG wire on Pin 2 |
| Pump LED always off | Check `RELAY_ACTIVE_LOW false` in Arduino code |
| Soil reading wrong | Probe in actual soil (not air) |
| Website shows red dots | Wait 20s after ESP starts, check WiFi |
| Charts not live | Wait 30-60 seconds for first data points |

---

## 🎯 Module Status Summary

| Module | Hardware | Status |
|---|---|---|
| 🌡️ Weather | DHT22 on ESP | ✅ LIVE |
| 💧 Water | YF-S201 on ESP | ✅ LIVE |
| 💡 Street Lights | LDR + PIR + 3 LEDs on ESP | ✅ LIVE (3 zones) |
| 🚦 Traffic | 2 HC-SR04 + 6 LEDs on Arduino | ✅ LIVE (2 lanes) |
| ♻️ Waste | 2 HC-SR04 on Arduino | ✅ LIVE (2 bins) |
| 🌱 Park | 2 Soil sensors + 2 LEDs/Relays | ✅ LIVE (2 zones) |
| 📢 Complaints | Firebase-based | ✅ LIVE |

---

## 📱 Mobile Responsive

Website works perfectly on:
- 💻 Desktop (>1024px)
- 💻 Tablet (768-1024px)
- 📱 Mobile (480-768px)
- 📱 Small mobile (<480px)

All stat cards, charts, buttons responsive. Touch-friendly tap targets.
