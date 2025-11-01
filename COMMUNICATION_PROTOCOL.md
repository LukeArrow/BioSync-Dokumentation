# BioSync - RS-485 Kommunikationsprotokoll-Spezifikation
## Vollständige Protokoll-Dokumentation

**Version:** 2.0  
**Datum:** November 2025  
**Status:** Implementiert und getestet

---

## Inhaltsverzeichnis

1. [Protokoll-Übersicht](#1-protokoll-übersicht)
2. [RS-485 Physical Layer](#2-rs-485-physical-layer)
3. [Data Link Layer](#3-data-link-layer)
4. [Nachrichtenformat](#4-nachrichtenformat)
5. [Fehlerbehandlung](#5-fehlerbehandlung)
6. [Timing-Spezifikation](#6-timing-spezifikation)
7. [Implementierung](#7-implementierung)
8. [Protokoll-Erweiterungen](#8-protokoll-erweiterungen)
9. [Testing und Validation](#9-testing-und-validation)

---

## 1. Protokoll-Übersicht

### 1.1 Design-Philosophie

Das BioSync-Kommunikationsprotokoll ist bewusst einfach gehalten:

- **Unidirektional:** Nur Sensor Node → Display Node (kein ACK erforderlich)
- **ASCII-basiert:** Human-readable für Debugging
- **Selbst-synchronisierend:** Start/End-Marker für Robustheit
- **Erweiterbar:** Neue Felder leicht hinzufügbar

**Warum unidirektional?**

- Sensordaten werden periodisch gesendet (5s Intervall)
- Display Node muss nicht antworten (Fire-and-Forget)
- Vereinfacht Software (keine Timeout-Behandlung)
- Reduziert Bus-Traffic
- Sensor Node kann autonom arbeiten

### 1.2 Protokoll-Stack

```
╔════════════════════════════════════════════════════════════════════════╗
║                    BIOSYNC PROTOCOL STACK                              ║
╚════════════════════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────────────────┐
│  Application Layer                                                   │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ SensorNode: Measure → Format → Send                          │   │
│  │ DisplayNode: Receive → Parse → Update Display                │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
         │                                      ▲
         │ String Message                       │ String Message
         ▼                                      │
┌─────────────────────────────────────────────────────────────────────┐
│  Presentation Layer (Message Format)                                 │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Format: <SENSOR;DIST=xx.x;TMP=xx.x;TUR=xxx;TDS=xxx>\n       │   │
│  │ - Start Marker: '<'                                          │   │
│  │ - Type: "SENSOR"                                             │   │
│  │ - Fields: Key=Value pairs, semicolon-separated              │   │
│  │ - End Marker: '>'                                            │   │
│  │ - Terminator: '\n' (LF)                                      │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
         │                                      ▲
         │ Byte Stream                          │ Byte Stream
         ▼                                      │
┌─────────────────────────────────────────────────────────────────────┐
│  Data Link Layer (UART)                                              │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Serial Format: 8N1 (8 data, No parity, 1 stop)              │   │
│  │ Baudrate: 9600 bps                                           │   │
│  │ No hardware flow control                                     │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
         │                                      ▲
         │ RS-485 Control                       │ RS-485 Signal
         ▼                                      │
┌─────────────────────────────────────────────────────────────────────┐
│  Physical Layer (RS-485)                                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Differential Signaling on Twisted Pair                       │   │
│  │ - Line A: Non-inverting                                      │   │
│  │ - Line B: Inverting                                          │   │
│  │ - V(A) - V(B) = Signal                                       │   │
│  │ - Common-Mode rejection: >50dB                               │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. RS-485 Physical Layer

### 2.1 Elektrische Spezifikation

**Signal-Levels:**

| Parameter | Min | Typ | Max | Einheit | Notizen |
|-----------|-----|-----|-----|---------|---------|
| Differential Output Voltage (VOD) | 1.5 | 3.0 | 5.0 | V | V(A) - V(B) |
| Common-Mode Output Voltage | -7 | 2.5 | +12 | V | (V(A) + V(B)) / 2 |
| Receiver Input Threshold | -200 | - | +200 | mV | Hysteresis |
| Input Impedance | 12 | - | - | kΩ | High-Z, multiple receivers |
| Termination Impedance | 120 | 120 | 120 | Ω | Matched to cable |

**Signal Timing:**

```
  Bit Time (9600 bps):
    T_bit = 1 / 9600 = 104.17 μs
  
  Rise/Fall Time:
    T_r, T_f < 30% of T_bit = 31 μs (MAX485 typical: 10 μs)
  
  Jitter Tolerance:
    ±5% of bit time = ±5.2 μs
```

### 2.2 Bus Topology

**BioSync: Point-to-Point Configuration**

```
╔════════════════════════════════════════════════════════════════════════╗
║                    RS-485 BUS TOPOLOGY                                 ║
╚════════════════════════════════════════════════════════════════════════╝

Sensor Node (Schacht)                          Display Node (Haus)
┌───────────────────┐                          ┌────────────────────┐
│ Arduino Nano      │                          │ Arduino Mega       │
│                   │                          │                    │
│ TX (D6) ──┐       │                          │       ┌── RX (D19) │
│ RX (D7) ──┼───┐   │                          │   ┌───┼── TX (D18) │
│ DE (D5) ──┼─┐ │   │                          │   │ ┌─┼── DE (D10) │
│           │ │ │   │                          │   │ │ │            │
└───────────┼─┼─┼───┘                          └───┼─┼─┼────────────┘
            │ │ │                                  │ │ │
        ┌───▼─▼─▼────┐                        ┌───▼─▼─▼────┐
        │  MAX485    │                        │  MAX485    │
        │ Transceiver│                        │ Transceiver│
        │            │                        │            │
        │  A ●───────┼────────────────────────┼─────● A    │
        │  B ●───────┼────────────────────────┼─────● B    │
        │            │                        │            │
        │ [120Ω]     │   CAT7 Cable           │  [120Ω]    │
        │ (optional) │   Twisted Pair         │  (optional)│
        └────────────┘   Length: <100m        └────────────┘
                         Adern 5+6 (verdrilled)

Termination:
  - Für Kabel <50m: Nicht erforderlich
  - Für Kabel >50m: 120Ω an beiden Enden
  
  Termination Connection:
    A ────[120Ω resistor]──── B
```

**Advantages Point-to-Point:**

- Keine Bus-Arbitrierung erforderlich
- Keine Adressierung erforderlich
- Minimale Latenz
- Einfache Software

**Bus-Zustände:**

| Zustand | Sensor Node DE | Display Node DE | Bus Zustand | Beschreibung |
|---------|----------------|-----------------|-------------|--------------|
| Idle    | LOW (RX)       | LOW (RX)        | Floating/Biased | Kein Traffic |
| TX      | HIGH (TX)      | LOW (RX)        | Driven by Sensor | Sensor sendet |
| RX      | LOW (RX)       | LOW (RX)        | Idle | Display empfängt |

### 2.3 Grounding und Shielding

**Best Practices:**

```
Ground Reference:
  - Common ground durch GND-Adern in CAT7 (Adern 3+4)
  - Verhindert Common-Mode-Probleme
  - Max. Potentialdifferenz: <1V (typisch <100mV)

Shield Connection:
  Display Node (Haus):              Sensor Node (Schacht):
  ┌──────────────────┐              ┌──────────────────┐
  │ Gehäuse (Metall) │              │ Gehäuse (Metall) │
  │        │         │              │                  │
  │        └─ PE     │              │    (floating)    │
  │                  │              │                  │
  │ CAT7 Shield ─────┼──────────────┼─ CAT7 Shield    │
  │   connected      │              │   NOT connected  │
  └──────────────────┘              └──────────────────┘
       │                                     │
       └─── To Protective Earth              └─── Isolated

Reason:
  - Single-point grounding prevents ground loops
  - Shield connected at Display Node (mains-powered end)
  - Sensor Node floating avoids circulating currents
```

---

## 3. Data Link Layer

### 3.1 UART Configuration

**Serial Parameters:**

| Parameter | Value | Notizen |
|-----------|-------|---------|
| Baudrate | 9600 bps | Standard, robust |
| Data Bits | 8 | Standard |
| Parity | None | Error detection via message format |
| Stop Bits | 1 | Standard |
| Flow Control | None | Not needed for unidirectional |

**Frame Format (UART):**

```
Single Character Transmission:
  
  ┌────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬──────┐
  │    │  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │      │
  │ ST │ LSB │     │     │     │     │     │     │ MSB │ STOP │
  └────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴──────┘
   1 bit  8 data bits (LSB first)                        1 bit
   LOW                                                    HIGH
  
  Total Frame Time: 10 bits × (1/9600) = 1.042 ms per character
```

**Example: Character 'A' (0x41 = 0b01000001):**

```
  Idle (High) ────┐           ┌───────────────┐
                  │           │               │
  Start Bit ──────┘  Data Bits └── Stop Bit ──┘
  
  Timing:
    Start: 104 μs
    D0(1): 104 μs
    D1(0): 104 μs
    D2(0): 104 μs
    D3(0): 104 μs
    D4(0): 104 μs
    D5(0): 104 μs
    D6(1): 104 μs
    D7(0): 104 μs
    Stop:  104 μs
    ─────────────────
    Total: 1042 μs
```

### 3.2 Byte Transmission Overhead

**For BioSync Message:**

```
Example: <SENSOR;DIST=125.3;TMP=18.2;TUR=450;TDS=320>\n

Length: 51 characters

Transmission Time:
  51 characters × 10 bits/char × (1/9600) = 53.1 ms

Including Gaps:
  - Pre-transmission settling: 2 ms (DE signal)
  - Post-transmission settling: 2 ms (DE signal)
  - Inter-character gap: ~1 ms (typical)
  
  Total: 53.1 + 2 + 2 + 1 = 58.1 ms
```

---

## 4. Nachrichtenformat

### 4.1 Message Structure

**General Format:**

```
  <TYPE;FIELD1=VALUE1;FIELD2=VALUE2;...;FIELDn=VALUEn>\n
  
  Components:
    '<'         : Start Marker (1 byte)
    'TYPE'      : Message Type (variable length)
    ';'         : Field Separator (1 byte)
    'FIELDx'    : Field Name (variable length, uppercase)
    '='         : Key-Value Separator (1 byte)
    'VALUEx'    : Field Value (variable length, ASCII)
    '>'         : End Marker (1 byte)
    '\n'        : Line Terminator (1 byte, 0x0A)
```

### 4.2 Sensor Data Message

**Format:**

```
<SENSOR;DIST=xxx.x;TMP=xx.x;TUR=xxxx;TDS=xxxx>\n

Fields:
  DIST : Distance in cm, float with 1 decimal place
  TMP  : Temperature in °C, float with 1 decimal place
  TUR  : Turbidity, integer 0-1023 (raw ADC)
  TDS  : Total Dissolved Solids, integer 0-1023 (raw ADC)
```

**Examples:**

```
Normal Operation:
  <SENSOR;DIST=125.3;TMP=18.2;TUR=450;TDS=320>\n
  
Low Water Level:
  <SENSOR;DIST=380.5;TMP=15.8;TUR=120;TDS=95>\n
  
High Water Level:
  <SENSOR;DIST=45.0;TMP=22.1;TUR=780;TDS=650>\n
  
Sensor Error (Distance timeout):
  <SENSOR;DIST=-1.0;TMP=19.3;TUR=440;TDS=310>\n
  
Sensor Error (Temperature disconnected):
  <SENSOR;DIST=130.2;TMP=-127.0;TUR=425;TDS=305>\n
```

### 4.3 Field Specifications

**DIST (Distance):**

| Aspect | Specification |
|--------|---------------|
| Type | Float |
| Unit | cm |
| Range | 25.0 to 450.0 (valid), -1.0 (error) |
| Precision | 1 decimal place |
| Error Code | -1.0 = Timeout (no echo received) |
| Example | "DIST=125.3" |

**TMP (Temperature):**

| Aspect | Specification |
|--------|---------------|
| Type | Float |
| Unit | °C (Celsius) |
| Range | -55.0 to 125.0 (valid), -127.0 (error) |
| Precision | 1 decimal place |
| Error Code | -127.0 = Sensor disconnected (DEVICE_DISCONNECTED_C) |
| Example | "TMP=18.2" |

**TUR (Turbidity):**

| Aspect | Specification |
|--------|---------------|
| Type | Integer |
| Unit | Raw ADC value (unitless) |
| Range | 0 to 1023 |
| Precision | Integer (no decimals) |
| Error Code | None (always valid ADC reading) |
| Example | "TUR=450" |

**TDS (Total Dissolved Solids):**

| Aspect | Specification |
|--------|---------------|
| Type | Integer |
| Unit | Raw ADC value (can be converted to ppm) |
| Range | 0 to 1023 |
| Precision | Integer (no decimals) |
| Error Code | None (always valid ADC reading) |
| Example | "TDS=320" |

### 4.4 Message Size

**Size Analysis:**

```
<SENSOR;DIST=999.9;TMP=-127.0;TUR=1023;TDS=1023>\n
│ 1    │ 6  │ 1 │ 4 │ 1│ 5  │ 1 │ 3 │ 1│ 6   │ 1│ 3│ 1│4   │ 1│ 3│ 1│4   │ 1│ 1
└──────┴────┴───┴───┴──┴────┴───┴───┴──┴─────┴──┴──┴──┴────┴──┴──┴──┴────┴──┴──
  1      6    1   4   1   5    1   3   1   6    1   3  1  4    1   3  1  4    1  1

Breakdown:
  '<'                 : 1 byte
  'SENSOR'            : 6 bytes
  ';DIST='            : 6 bytes
  'xxx.x'             : 5 bytes (worst case: 999.9, -1.0)
  ';TMP='             : 5 bytes
  'xx.x' or '-127.0'  : 6 bytes (worst case: -127.0)
  ';TUR='             : 5 bytes
  'xxxx'              : 4 bytes (worst case: 1023)
  ';TDS='             : 5 bytes
  'xxxx'              : 4 bytes (worst case: 1023)
  '>'                 : 1 byte
  '\n'                : 1 byte
  ──────────────────────────────────────
  Total (worst case):  55 bytes

Typical message:
  <SENSOR;DIST=125.3;TMP=18.2;TUR=450;TDS=320>\n
  = 51 bytes
```

---

## 5. Fehlerbehandlung

### 5.1 Error Detection

**No CRC/Checksum (by design):**

- Rationale: RS-485 is robust, low BER (Bit Error Rate)
- UART has framing error detection (start/stop bits)
- Message format provides structural validation
- For future: CRC can be added as optional field

**Message Validation:**

```cpp
bool validateMessage(const String &msg) {
  // 1. Check Start Marker
  if (!msg.startsWith("<")) return false;
  
  // 2. Check End Marker
  if (msg.indexOf(">") < 0) return false;
  
  // 3. Check Message Type
  if (msg.indexOf("SENSOR") < 0) return false;
  
  // 4. Check Field Presence
  if (msg.indexOf("DIST=") < 0) return false;
  if (msg.indexOf("TMP=") < 0) return false;
  if (msg.indexOf("TUR=") < 0) return false;
  if (msg.indexOf("TDS=") < 0) return false;
  
  // 5. All checks passed
  return true;
}
```

### 5.2 Error Handling Strategies

**At Sensor Node (Transmitter):**

| Error Condition | Handling | Example |
|----------------|----------|---------|
| Sensor timeout (DIST) | Send error code (-1.0) | "DIST=-1.0" |
| Sensor disconnected (TMP) | Send error code (-127.0) | "TMP=-127.0" |
| Analog sensor fault | Send raw ADC (always valid) | "TUR=0" or "TUR=1023" |
| RS-485 TX failure | Log error, continue next cycle | Serial.println("TX Error") |

**At Display Node (Receiver):**

| Error Condition | Handling | User Feedback |
|----------------|----------|---------------|
| Malformed message | Discard, wait for next | (No update) |
| Missing fields | Discard message | (No update) |
| Invalid value range | Display "---" or last valid | "DIST: ---" |
| No messages received (timeout) | Display "Offline" after 30s | "Sensor Offline" |
| Partial message (buffer overflow) | Clear buffer, resync | (Wait for next '<') |

**Timeout Detection:**

```cpp
// Display Node:
unsigned long lastMessageTime = 0;
const unsigned long MESSAGE_TIMEOUT = 30000; // 30 seconds

void loop() {
  // ... receive messages ...
  
  if (millis() - lastMessageTime > MESSAGE_TIMEOUT) {
    // No message for 30 seconds
    nextion_displayError("Sensor Offline");
  }
}

// When valid message received:
void onMessageReceived() {
  lastMessageTime = millis();
  nextion_clearError();
}
```

### 5.3 Graceful Degradation

**Partial Sensor Failure:**

If one sensor fails, system continues with remaining sensors:

```
Scenario: JSN-SR04T fails (distance sensor)
  Transmitted: <SENSOR;DIST=-1.0;TMP=18.5;TUR=440;TDS=310>\n
  Display: 
    Distance: ERROR / ---
    Temperature: 18.5°C (OK)
    Turbidity: 440 (OK)
    TDS: 310 (OK)

Scenario: DS18B20 fails (temperature sensor)
  Transmitted: <SENSOR;DIST=130.2;TMP=-127.0;TUR=425;TDS=305>\n
  Display:
    Distance: 130.2 cm (OK)
    Temperature: ERROR / ---
    Turbidity: 425 (OK)
    TDS: 305 (OK)
```

---

## 6. Timing-Spezifikation

### 6.1 Transmission Timing

**Sensor Node Timing:**

```
┌─────────────────── 5000 ms (SEND_INTERVAL) ────────────────────┐
│                                                                 │
│  ┌─ Measure All Sensors (~850ms) ─┐  ┌─ Format & Send (~60ms) ┐│
│  │                                 │  │                         ││
│  │ Distance:  ~800ms (DS18B20)     │  │ Format String: ~10ms    ││
│  │ Temp:      ~30ms  (conversion)  │  │ RS-485 TX:     ~50ms    ││
│  │ Turbidity: ~10ms  (analogRead)  │  │                         ││
│  │ TDS:       ~10ms  (analogRead)  │  │                         ││
│  └─────────────────────────────────┘  └─────────────────────────┘│
│                                                                  │
│                   Idle (~4100ms)                                 │
└──────────────────────────────────────────────────────────────────┘

Duty Cycle:
  Active Time: 850 + 60 = 910 ms
  Total Time:  5000 ms
  Duty:        910 / 5000 = 18.2%
  
  Power Savings: CPU could sleep 81.8% of the time (future optimization)
```

**RS-485 Transmission Timing:**

```
  DE/RE Control:
  
  Before TX:
    1. Set DE = HIGH (enable driver)
    2. Wait 2 ms (settling time)
    
  During TX:
    3. Send message bytes
    4. Wait for TX complete (flush())
    
  After TX:
    5. Wait 2 ms (last byte propagation)
    6. Set DE = LOW (disable driver)
  
  Timing Diagram:
  
  DE Signal:        ┌───────────────┐
                    │   TX Active   │
         ───────────┘               └────────────
                    │               │
                    ├─ 2ms ─┤       ├─ 2ms ─┤
  
  RS-485 Bus:          ┌───────┐
                       │ Data  │
         ──────────────┘       └────────────────
                       │       │
                       ├───────┤
                       ~53 ms
                     (51 bytes
                      @ 9600)
  
  Total TX Time: 2 + 53 + 2 = 57 ms
```

### 6.2 Display Update Timing

**Display Node Processing:**

```
Message Received → Parse → Nextion Update → Display Refresh

┌─ Parse (~2ms) ─┐ ┌─ Nextion Update (~50ms) ─┐ ┌─ Refresh (~20ms) ─┐
│                 │ │                           │ │                    │
│ Extract DIST    │ │ Send "tDist.txt=..."      │ │ Nextion renders   │
│ Extract TMP     │ │ Wait 10ms                 │ │ text objects      │
│ Extract TUR     │ │ Send "tTemp.txt=..."      │ │ to LCD            │
│ Extract TDS     │ │ Wait 10ms                 │ │                    │
│ Validate format │ │ Send "tTurb.txt=..."      │ │                    │
│                 │ │ Wait 10ms                 │ │                    │
│                 │ │ Send "tTDS.txt=..."       │ │                    │
└─────────────────┘ └───────────────────────────┘ └────────────────────┘

Total: ~72 ms
```

**End-to-End Latency:**

```
From Sensor Measurement to Display Update:

  Sensor Measurement:      850 ms (dominated by DS18B20)
  Message Formatting:       10 ms
  RS-485 Transmission:      57 ms
  Message Parsing:           2 ms
  Nextion Update:           50 ms
  Display Refresh:          20 ms
  ────────────────────────────────
  Total Latency:           989 ms ≈ 1 second

Note: This is acceptable for a monitoring system (not real-time critical)
```

---

## 7. Implementierung

### 7.1 Sensor Node Code (Transmitter)

**rs485_send() Implementation:**

```cpp
void rs485_send(const String &s) {
  // 1. Activate Driver (enter TX mode)
  digitalWrite(RS485_DE, HIGH);
  
  // 2. Wait for transceiver to settle
  delay(2);  // 2 milliseconds
  
  // 3. Transmit message
  rs485.print(s);
  
  // 4. Wait for transmission to complete
  rs485.flush();  // Blocks until TX buffer empty
  
  // 5. Wait for last byte to propagate
  delay(2);  // 2 milliseconds
  
  // 6. Deactivate Driver (enter RX mode)
  digitalWrite(RS485_DE, LOW);
}
```

**Message Formatting:**

```cpp
void loop() {
  if (millis() - lastSend >= SEND_INTERVAL) {
    lastSend = millis();
    
    // Measure sensors
    float distance = measureDistance();
    float temp = readTemperature();
    int turb = readTurbidity();
    int tds = readTDS();
    
    // Format message
    String msg = "<SENSOR;";
    msg += "DIST=" + String(distance, 1) + ";";  // 1 decimal place
    msg += "TMP=" + String(temp, 1) + ";";       // 1 decimal place
    msg += "TUR=" + String(turb) + ";";          // Integer
    msg += "TDS=" + String(tds) + ">\n";         // Integer + newline
    
    // Send via RS-485
    rs485_send(msg);
    
    // Debug output (USB Serial)
    Serial.print(F("Gesendet: "));
    Serial.print(msg);
  }
}
```

### 7.2 Display Node Code (Receiver)

**Message Reception:**

```cpp
String rs485Buffer = "";

void loop() {
  // Process incoming bytes
  while (Serial1.available()) {
    char c = (char)Serial1.read();
    
    if (c == '\n') {
      // Line complete, process message
      String line = rs485Buffer;
      rs485Buffer = "";  // Clear buffer
      line.trim();       // Remove whitespace
      
      if (line.length() > 0) {
        processMessage(line);
      }
    } else {
      // Accumulate characters
      rs485Buffer += c;
      
      // Overflow protection
      if (rs485Buffer.length() > 1024) {
        rs485Buffer = "";  // Reset on overflow
      }
    }
  }
}
```

**Message Parsing:**

```cpp
void processMessage(const String &msg) {
  // Validate message format
  if (!msg.startsWith("<")) return;
  if (msg.indexOf("SENSOR") < 0) return;
  
  // Parse sensor data
  SensorData d = parseSensorMessage(msg);
  
  // Update display
  nextion_update(d);
  
  // Debug output
  Serial.print(F("Empfangen: "));
  Serial.println(msg);
}

SensorData parseSensorMessage(const String &msg) {
  SensorData d;
  d.distance = extract(msg, "DIST");
  d.temperature = extract(msg, "TMP");
  d.turbidity = extract(msg, "TUR");
  d.tds = extract(msg, "TDS");
  
  // Add units
  if (d.distance.length()) {
    d.distance += " cm";
  } else {
    d.distance = "n/a";
  }
  
  if (d.temperature.length()) {
    d.temperature += " °C";
  } else {
    d.temperature = "n/a";
  }
  
  if (!d.turbidity.length()) d.turbidity = "n/a";
  if (!d.tds.length()) d.tds = "n/a";
  
  return d;
}

String extract(const String &src, const char *key) {
  String k = String(key) + "=";
  int idx = src.indexOf(k);
  if (idx < 0) return "";
  
  int start = idx + k.length();
  int end = src.indexOf(';', start);
  if (end < 0) {
    end = src.indexOf('>', start);
    if (end < 0) end = src.length();
  }
  
  return src.substring(start, end);
}
```

