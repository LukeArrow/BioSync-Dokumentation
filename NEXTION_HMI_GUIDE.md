# BioSync - Nextion HMI Comprehensive Guide
## Detailed Nextion Display Programming and Integration

**Version:** 2.0  
**Date:** November 2025

---

## Table of Contents

1. [Nextion Display Overview](#1-nextion-display-overview)
2. [Hardware Setup](#2-hardware-setup)
3. [Nextion Editor Installation](#3-nextion-editor-installation)
4. [HMI Design](#4-hmi-design)
5. [Component Reference](#5-component-reference)
6. [Event Programming](#6-event-programming)
7. [Arduino Communication](#7-arduino-communication)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Nextion Display Overview

### 1.1 NX4024T032 Specifications

| Parameter | Value | Notes |
|-----------|-------|-------|
| Model | NX4024T032 | Basic Series |
| Screen Size | 3.2" | Diagonal |
| Resolution | 320 × 240 px | QVGA |
| Touch Type | Resistive | 4-wire |
| Interface | UART TTL | 3.3V/5V compatible |
| Default Baudrate | 9600 | Adjustable |
| Power | 5V, ~85mA | Typical |

### 1.2 Communication Protocol

**Nextion uses ASCII commands with 3-byte terminator:**

```
Command Format:
  <ASCII Command><0xFF><0xFF><0xFF>

Example: Update text field
  "tDist.txt=\"125.3 cm\"" + 0xFF + 0xFF + 0xFF
  
  In HEX:
  74 44 69 73 74 2E 74 78 74 3D 22 31 32 35 2E 33 20 63 6D 22 FF FF FF
  t  D  i  s  t  .  t  x  t  =  "  1  2  5  .  3     c  m  "  [3×0xFF]
```

---

## 2. Hardware Setup

### 2.1 Wiring

```
Nextion Display → Arduino Mega 2560

Pin 1: TX (Yellow)  → D17 (RX2) - Serial2 Receive
Pin 2: RX (Blue)    → D16 (TX2) - Serial2 Transmit
Pin 3: VCC (Red)    → 5V Rail
Pin 4: GND (Black)  → GND Rail

IMPORTANT: TX from display goes to RX on Arduino!
```

### 2.2 Power Considerations

- Nextion draws ~85mA @ 5V (typical)
- Peak: ~100mA @ 5V (100% backlight)
- Ensure LM2596 can supply ≥200mA total (Mega + Nextion)

---

## 3. Nextion Editor Installation

### 3.1 Download and Install

1. Visit: https://nextion.tech/nextion-editor/
2. Download latest version (v1.65 or newer)
3. Install on Windows PC
4. No Linux/Mac version (use Wine or VM)

### 3.2 Creating New Project

```
Steps:
1. Open Nextion Editor
2. File → New → Nextion Project
3. Device: Basic Series
4. Model: NX4024T032
5. Resolution: 320×240
6. Orientation: Horizontal
7. Click OK
```

---

## 4. HMI Design

### 4.1 Page Structure

BioSync uses **2 pages:**

1. **Page 0: Main Display (page `PgMain`)**
   - Shows sensor values
   - Interactive buttons
   - Idle timer for screensaver

2. **Page 1: Screensaver (page `PgSafer`)**
   - Minimal content (energy saving)
   - Touch to wake

### 4.2 Main Page Components

```
┌────────────────────────────────────────────────────────┐
│ BioSync Klärgruben-Monitoring        [Refresh Button] │
│────────────────────────────────────────────────────────│
│                                                        │
│  Füllstand:                                            │
│  ┌──────────────────────────────┐                     │
│  │  125.3 cm                    │ ← tDist (Text)      │
│  └──────────────────────────────┘                     │
│                                                        │
│  Temperatur:                                           │
│  ┌──────────────────────────────┐                     │
│  │  18.2 °C                     │ ← tTemp (Text)      │
│  └──────────────────────────────┘                     │
│                                                        │
│  Trübung:                                              │
│  ┌──────────────────────────────┐                     │
│  │  450                         │ ← tTurb (Text)      │
│  └──────────────────────────────┘                     │
│                                                        │
│  TDS:                                                  │
│  ┌──────────────────────────────┐                     │
│  │  320                         │ ← tTDS (Text)       │
│  └──────────────────────────────┘                     │
│                                                        │
│  [Optional: Pump Control Button]                      │
│                                                        │
└────────────────────────────────────────────────────────┘

Components List (Sensor Data):
  - tDist: Text component (id: 1) - Füllstand/Distance
  - tTemp: Text component (id: 2) - Temperatur/Temperature
  - tTurb: Text component (id: 3) - Trübung/Turbidity
  - tTDS: Text component (id: 4) - TDS-Wert/TDS Value
  
Components List (Relay Status) - NEU v2.0:
  - tPump: Text component (id: 7) - Pump LED Status
  - tVent: Text component (id: 8) - Ventilation LED Status
  
Optional Components:
  - btnRefresh: Button component (id: 5)
  - nIdle: Number component (id: 6, invisible, for idle counter)
  - tIdleTimer: Timer component (interval: 1000ms)
```

### 4.3 Component Placement

**Using Nextion Editor:**

1. **Add Text Components:**
   - Toolbox → Text → Drag to canvas
   - Properties:
     - objname: `tDist`, `tTemp`, `tTurb`, `tTDS`
     - txt: "---" (initial value)
     - font: Choose readable font (e.g., Font24)
     - xcen: 1 (center horizontally)
     - ycen: 1 (center vertically)
     - bco: 65535 (white background)
     - pco: 0 (black text)

2. **Add Labels (for "Füllstand:", etc.):**
   - Toolbox → Text
   - objname: `lDist`, `lTemp`, etc.
   - txt: "Füllstand:" (static text)
   - sta: solid (static, not updated)

3. **Add Button:**
   - Toolbox → Button
   - objname: `btnRefresh`
   - txt: "Aktualisieren"

4. **Add Hidden Number:**
   - Toolbox → Number
   - objname: `nIdle`
   - val: 0 (initial)
   - visible: 0 (hidden)

5. **Add Timer:**
   - Toolbox → Timer
   - objname: `tIdleTimer`
   - tim: 1000 (1 second interval)
   - en: 1 (enabled)

6. **Add Relay Status Fields (v2.0 - NEU):**
   - Toolbox → Text (2×)
   - objname: `tPump`, `tVent`
   - txt: "IDLE" (initial value)
   - font: Font20
   - xcen: 1
   - ycen: 1
   - bco: Choose based on status:
     - 63488 (red) for ERROR
     - 2016 (green) for ACTIVE
     - 65535 (white) for IDLE

---

## 5. Component Reference

### 5.1 Text Component (Sensor Data: tDist, tTemp, tTurb, tTDS)

**Properties:**

| Property | Value | Description |
|----------|-------|-------------|
| objname | `tDist` | Unique identifier |
| txt | "---" | Displayed text |
| font | Font24 | Font reference |
| xcen | 1 | Horizontal center |
| ycen | 1 | Vertical center |
| bco | 65535 | Background color (white) |
| pco | 0 | Text color (black) |
| sta | solid | Draw style |

**Update from Arduino:**

```cpp
// Format:
// <component>.txt="<value>"

nextion_send("tDist.txt=\"125.3 cm\"");
nextion_send("tTemp.txt=\"18.2 °C\"");
nextion_send("tTurb.txt=\"450\"");
nextion_send("tTDS.txt=\"320\"");
```

### 5.2 Text Component (Relay Status: tPump, tVent) - v2.0 NEU

**Properties:**

| Property | Value | Description |
|----------|-------|-------------|
| objname | `tPump` / `tVent` | Unique identifier for relay status |
| txt | "IDLE" | Initial status (IDLE/ACTIVE/ERROR) |
| font | Font20 | Font reference |
| xcen | 1 | Horizontal center |
| ycen | 1 | Vertical center |
| sta | solid | Draw style |

**Background Colors by Status:**

| Status | Background Color (bco) | Description |
|--------|------------------------|-------------|
| IDLE | 65535 (white) | Relay off, normal |
| ACTIVE | 2016 (green) | Relay on, operating |
| ERROR | 63488 (red) | Relay blinking, fault |

**Update from Arduino:**

```cpp
// Format for relay status:
// <component>.txt="<status>"
// <component>.bco=<color>

// Example: Pump becomes ACTIVE (green)
nextion_send("tPump.txt=\"ACTIVE\"");
nextion_send("tPump.bco=2016");  // Green background

// Example: Vent remains IDLE (white)
nextion_send("tVent.txt=\"IDLE\"");
nextion_send("tVent.bco=65535");  // White background

// Example: Pump ERROR (red)
nextion_send("tPump.txt=\"ERROR\"");
nextion_send("tPump.bco=63488");  // Red background
```

**Layout Suggestion:**

```
┌────────────────────────────────────┐
│  Pump Status:  [ ACTIVE ]  (green) │
│  Vent Status:  [  IDLE  ]  (white) │
└────────────────────────────────────┘
```

### 5.3 Button Component (btnRefresh)

**Properties:**

| Property | Value | Description |
|----------|-------|-------------|
| objname | `btnRefresh` | Unique identifier |
| txt | "Aktualisieren" | Button label |
| font | Font20 | Font for text |
| bco | 63488 | Background color (red) |
| pco | 65535 | Text color (white) |

**TouchPress Event Code:**

```javascript
// Reset idle timer
nIdle.val=0

// Send message to Arduino
print "BTN_REFRESH"
```

### 5.4 Timer Component (tIdleTimer)

**Properties:**

| Property | Value | Description |
|----------|-------|-------------|
| objname | `tIdleTimer` | Unique identifier |
| tim | 1000 | Interval in ms (1 second) |
| en | 1 | Enabled at startup |

**Timer Event Code:**

```javascript
// Increment idle counter
nIdle.val=nIdle.val+1

// Check if idle timeout reached (60 seconds)
if nIdle.val>60
  page Page_screensaver
  nIdle.val=0
endif
```

---

## 6. Event Programming

### 6.1 Touch Events

**For Text Components (Optional - to reset idle timer on touch):**

```javascript
// tDist TouchPress Event:
nIdle.val=0
print "TOUCH_TDIST"

// tTemp TouchPress Event:
nIdle.val=0
print "TOUCH_TTEMP"

// Similar for tTurb, tTDS...
```

### 6.2 Page Events

**Page_main - Page Initialization (Preinitialize Event):**

```javascript
// Start timer
tIdleTimer.en=1

// Reset idle counter
nIdle.val=0
```

**Page_screensaver - Touch to Wake:**

Add invisible hotspot covering entire screen:

```javascript
// Hotspot TouchPress Event:
page Page_main
nIdle.val=0
print "SCR_WAKE"
```

### 6.3 Complete Event Summary

| Component | Event | Code |
|-----------|-------|------|
| btnRefresh | TouchPress | `nIdle.val=0` <br> `print "BTN_REFRESH"` |
| tIdleTimer | Timer | `nIdle.val=nIdle.val+1` <br> `if nIdle.val>60` <br> `page Page_screensaver` <br> `nIdle.val=0` <br> `endif` |
| Page_screensaver hotspot | TouchPress | `page Page_main` <br> `nIdle.val=0` <br> `print "SCR_WAKE"` |
| tDist (optional) | TouchPress | `nIdle.val=0` <br> `print "TOUCH_TDIST"` |

---

## 7. Arduino Communication

### 7.1 Sending Commands to Nextion

```cpp
void sendToNextion(const String &cmd) {
  Serial2.print(cmd);
  Serial2.write(0xFF);
  Serial2.write(0xFF);
  Serial2.write(0xFF);
}

void nextion_update(const SensorData &d) {
  sendToNextion(String("tDist.txt=\"") + d.distance + "\"");
  delay(10);  // Allow Nextion to process
  
  sendToNextion(String("tTemp.txt=\"") + d.temperature + "\"");
  delay(10);
  
  sendToNextion(String("tTurb.txt=\"") + d.turbidity + "\"");
  delay(10);
  
  sendToNextion(String("tTDS.txt=\"") + d.tds + "\"");
}
```

### 7.2 Receiving Events from Nextion

```cpp
#define NEXTION_BUF_SIZE 256
uint8_t nxt_buf[NEXTION_BUF_SIZE];
size_t nxt_len = 0;

void nextion_process_byte(uint8_t b) {
  if (nxt_len < NEXTION_BUF_SIZE) {
    nxt_buf[nxt_len++] = b;
  } else {
    nxt_len = 0;  // Overflow protection
  }
  
  // Check for 3×0xFF terminator
  if (nxt_len >= 3 &&
      nxt_buf[nxt_len-1] == 0xFF &&
      nxt_buf[nxt_len-2] == 0xFF &&
      nxt_buf[nxt_len-3] == 0xFF) {
    
    // Extract message (without terminators)
    size_t msg_len = nxt_len - 3;
    String msg = "";
    for (size_t i = 0; i < msg_len; ++i) {
      msg += (char)nxt_buf[i];
    }
    msg.trim();
    
    if (msg.length()) {
      handle_nextion_message(msg);
    }
    
    nxt_len = 0;  // Reset buffer
  }
}

void poll_nextion() {
  while (Serial2.available()) {
    uint8_t b = Serial2.read();
    nextion_process_byte(b);
  }
}

void handle_nextion_message(const String &msg) {
  if (msg == "BTN_REFRESH") {
    // Force immediate sensor read
    forceSensorRead();
  } else if (msg == "SCR_WAKE") {
    // Screensaver woke up
    Serial.println("Display woke from screensaver");
  } else if (msg.startsWith("TOUCH_")) {
    // Touch event on sensor display
    Serial.print("Touch event: ");
    Serial.println(msg);
  }
}
```

### 7.3 In loop()

```cpp
void loop() {
  // Process RS-485 messages (existing code)
  // ...
  
  // Process Nextion events
  poll_nextion();
}
```

---

## 8. Troubleshooting

### 8.1 Display Stays Black

**Causes:**

1. No power (VCC/GND)
2. .tft file not uploaded
3. Backlight off (`dim=0`)

**Solutions:**

```cpp
// Set backlight to 100%:
sendToNextion("dim=100");

// Query display:
sendToNextion("connect");
// Should return: comok 2,32033-0,NX4024T032,...
```

### 8.2 Text Not Updating

**Causes:**

- Wrong component name (`tDist` vs `tdist` - case sensitive!)
- Command format error (missing quotes)
- Terminator missing (3×0xFF)

**Debug:**

```cpp
// Add to Serial Monitor:
Serial.print("Sending to Nextion: ");
Serial.println(cmd);
```

### 8.3 Touch Not Working

**Solutions:**

1. Calibrate touch: Nextion Editor → Tools → Touch Calibration
2. Verify events are programmed
3. Check if Nextion is receiving commands (LED on RX pin)

