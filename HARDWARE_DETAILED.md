# BioSync - Detaillierte Hardware-Dokumentation
## Vollständige Hardware-Spezifikation und Schaltungsdesign

**Version:** 2.0  
**Datum:** November 2025  
**Zielgruppe:** Hardware-Ingenieure, Elektronik-Techniker, Systemintegratoren

---

## Inhaltsverzeichnis

1. [Hardware-Übersicht](#1-hardware-übersicht)
2. [Schaltpläne und Verbindungen](#2-schaltpläne-und-verbindungen)
3. [Komponenten-Spezifikationen](#3-komponenten-spezifikationen)
4. [Stromversorgungssystem](#4-stromversorgungssystem)
5. [RS-485 Bus-Konfiguration](#5-rs-485-bus-konfiguration)
6. [CAT7-Kabel: Adernbelegung](#6-cat7-kabel-adernbelegung)
7. [PCB-Layout-Empfehlungen](#7-pcb-layout-empfehlungen)
8. [Gehäuseauswahl und IP-Schutz](#8-gehäuseauswahl-und-ip-schutz)
9. [Elektromagnetische Verträglichkeit (EMV)](#9-elektromagnetische-verträglichkeit-emv)
10. [Thermisches Design](#10-thermisches-design)
11. [Testpunkte und Messpunkte](#11-testpunkte-und-messpunkte)
12. [Bill of Materials (BOM)](#12-bill-of-materials-bom)

---

## 1. Hardware-Übersicht

### 1.1 Systemarchitektur

Das BioSync-System besteht aus zwei Hardware-Knoten:

1. **Sensor Node:** Datenerfassung im Schacht (IP67-geschützt)
2. **Display Node:** Datenverarbeitung und Anzeige im Haus

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    HARDWARE SYSTEM ÜBERSICHT                             ║
╚══════════════════════════════════════════════════════════════════════════╝

    ┌──────── SENSOR NODE (Schacht) ────────┐
    │                                        │
    │  ┌──────────────────────────────────┐ │
    │  │   Sensors (4x)                   │ │
    │  │   - JSN-SR04T (Distance)         │ │
    │  │   - DS18B20 (Temperature)        │ │
    │  │   - TSW-20M (Turbidity)          │ │
    │  │   - CQRSENTDS01 (TDS)            │ │
    │  └───────────┬──────────────────────┘ │
    │              │ Analog/Digital         │
    │  ┌───────────▼──────────────────────┐ │
    │  │   Arduino Nano Every             │ │
    │  │   (ATmega4809, 20MHz, 5V)        │ │
    │  └───────────┬──────────────────────┘ │
    │              │ SoftwareSerial         │
    │  ┌───────────▼──────────────────────┐ │
    │  │   RS-485 Transceiver             │ │
    │  │   (MAX485 / SN75176)             │ │
    │  └───────────┬──────────────────────┘ │
    │              │ A/B Differential       │
    │  ┌───────────▼──────────────────────┐ │
    │  │   LM2596 Buck Converter          │ │
    │  │   12V → 5V @ 3A                  │ │
    │  └───────────┬──────────────────────┘ │
    │              │ 12V Input              │
    └──────────────┼────────────────────────┘
                   │
                   │ CAT7 Cable
                   │ (Power + Data)
                   │
    ┌──────────────▼── DISPLAY NODE (Haus) ─┐
    │              │ 12V Distribution        │
    │  ┌───────────▼──────────────────────┐ │
    │  │   LM2596 Buck Converter          │ │
    │  │   12V → 5V @ 3A                  │ │
    │  └───────────┬──────────────────────┘ │
    │              │ 5V Rail                │
    │  ┌───────────▼──────────────────────┐ │
    │  │   RS-485 Transceiver             │ │
    │  │   (MAX485 / SN75176)             │ │
    │  └───────────┬──────────────────────┘ │
    │              │ Hardware Serial1       │
    │  ┌───────────▼──────────────────────┐ │
    │  │   Arduino Mega 2560              │ │
    │  │   (ATmega2560, 16MHz, 5V)        │ │
    │  └───────────┬──────────────────────┘ │
    │              │ Hardware Serial2       │
    │  ┌───────────▼──────────────────────┐ │
    │  │   Nextion NX4024T032 Display     │ │
    │  │   (3.2" TFT, 320x240, Touch)     │ │
    │  └──────────────────────────────────┘ │
    │                                        │
    │  ┌──────────────────────────────────┐ │
    │  │   12V/2A AC/DC Power Supply      │ │
    │  │   (Meanwell, TDK-Lambda, etc.)   │ │
    │  └──────────────────────────────────┘ │
    └────────────────────────────────────────┘
```

### 1.2 Power Distribution Übersicht

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    POWER DISTRIBUTION DIAGRAM                            ║
╚══════════════════════════════════════════════════════════════════════════╝

                    230V AC (Haus)
                         │
                         │
                    ┌────▼────────┐
                    │  AC/DC PSU  │
                    │  12V / 2A   │
                    └────┬────────┘
                         │ 12V DC
              ┌──────────┼──────────┐
              │          │          │
        ┌─────▼────┐     │     ┌────▼──────┐
        │ Display  │     │     │  Sensor   │
        │ Node     │     │     │  Node     │
        │ LM2596   │     │     │  LM2596   │
        │ 12V→5V   │     │     │  12V→5V   │
        └─────┬────┘     │     └────┬──────┘
              │ 5V       │          │ 5V
              │          │ (via CAT7)
    ┌─────────┴──────────┴──────────┴──────────┐
    │                                           │
┌───▼─────────┐   ┌───────────┐   ┌───────────▼────┐
│ Arduino     │   │ RS-485    │   │ Arduino        │
│ Mega        │   │ Modules   │   │ Nano Every     │
│ ~50mA       │   │ ~2mA      │   │ ~30mA          │
└─────────────┘   └───────────┘   └────────────────┘
┌───────────┐                     ┌────────────────┐
│ Nextion   │                     │ All Sensors    │
│ Display   │                     │ ~25mA total    │
│ ~85mA     │                     └────────────────┘
└───────────┘

POWER BUDGET:
─────────────
Display Node @ 5V:
  - Arduino Mega:      50 mA
  - Nextion Display:   85 mA
  - RS-485 Transceiver: 1 mA
  - TOTAL:            136 mA × 5V = 0.68W

Sensor Node @ 5V:
  - Arduino Nano:      30 mA
  - JSN-SR04T:         15 mA (peak during measurement)
  - DS18B20:            1 mA
  - TSW-20M:            5 mA
  - CQRSENTDS01:        5 mA
  - RS-485 Transceiver: 1 mA
  - TOTAL:             57 mA × 5V = 0.285W

Total 12V Load:
  - Display LM2596 Input: 0.68W / 0.92 (eff) = 0.74W = 62mA @ 12V
  - Sensor LM2596 Input:  0.30W / 0.92 (eff) = 0.33W = 27mA @ 12V
  - TOTAL 12V:            1.07W = 89mA @ 12V

Safety Margin: 2A / 89mA = 22.5x (excellent)
```

---

## 2. Schaltpläne und Verbindungen

### 2.1 Sensor Node - Vollständiger Schaltplan (Textform)

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    SENSOR NODE SCHALTPLAN                                ║
╚══════════════════════════════════════════════════════════════════════════╝

POWER SUPPLY SECTION:
─────────────────────
                                    ┌──────────────────────┐
  CAT7 Pin 1+2 (12V+) ──────────┬──┤ IN+     LM2596   OUT+├───┬── 5V Rail
                                │  │                      │   │
                                │  │ Potentiometer        │   │
                               [C1] │ (Voltage Adjust)     │  [C3]
                               100μ │                      │  100μ
                                │  │                      │   │
  CAT7 Pin 3+4 (GND)  ──────────┴──┤ IN-              OUT-├───┴── GND Rail
                                    └──────────────────────┘

  C1: 100μF/25V Electrolytic (Input filter)
  C3: 100μF/16V Electrolytic (Output filter)

  LM2596 Configuration:
  - Input Voltage Range: 4.5V to 40V
  - Output Voltage: 5.0V (adjusted via potentiometer)
  - Output Current: Up to 3A
  - Switching Frequency: 150 kHz
  - Efficiency: ~92% @ 12V in, 5V out


MICROCONTROLLER SECTION:
────────────────────────
                  ┌─────────────────────────────┐
                  │   Arduino Nano Every        │
                  │   (ATmega4809)              │
                  │                             │
   5V Rail ───────┤ VIN                     GND ├──── GND Rail
                  │                             │
                  │ D2 ──────────────────────────┼──── TRIG (JSN-SR04T)
                  │ D3 ──────────────────────────┼──── ECHO (JSN-SR04T)
                  │ D4 ──────┬───────────────────┼──── DATA (DS18B20)
                  │          │                   │
                  │          │ [R1]              │
                  │          │ 4.7kΩ             │
                  │          │                   │
   5V Rail ───────┴──────────┘                   │
                  │ A0 ──────────────────────────┼──── OUT (TSW-20M)
                  │ A1 ──────────────────────────┼──── OUT (CQRSENTDS01)
                  │                             │
                  │ D6 ──────────────────────────┼──── TX (to RS-485 DI)
                  │ D7 ──────────────────────────┼──── RX (from RS-485 RO)
                  │ D5 ──────────────────────────┼──── DE/RE (RS-485 control)
                  │                             │
                  └─────────────────────────────┘

  R1: 4.7kΩ ±5% 1/4W (Pull-up for DS18B20 OneWire)


SENSOR CONNECTIONS:
───────────────────

JSN-SR04T (Ultrasonic Distance):
  ┌──────────────────┐
  │ JSN-SR04T Board  │
  │                  │
  │ VCC ────────────────── 5V Rail
  │ TRIG ───────────────── D2 (Nano)
  │ ECHO ───────────────── D3 (Nano)
  │ GND ────────────────── GND Rail
  └──────────────────┘
       │
       │ 2.5m Cable (4-wire)
       │
  ┌────▼─────────────┐
  │ Ultrasonic       │
  │ Transducer       │
  │ (Waterproof)     │
  └──────────────────┘

DS18B20 / MICREEN-DTSMK (Temperature):
  ┌──────────────────┐
  │ DS18B20 Sensor   │
  │ (TO-92 or Probe) │
  │                  │
  │ VCC (Red) ──────────── 5V Rail
  │ DATA (Yellow) ──┬─────── D4 (Nano)
  │                 │
  │                 │ [R1] 4.7kΩ Pull-up
  │                 │
  5V Rail ──────────┘
  │ GND (Black) ────────── GND Rail
  └──────────────────┘

TSW-20M (Turbidity):
  ┌──────────────────┐
  │ TSW-20M Sensor   │
  │                  │
  │ VCC ────────────────── 5V Rail
  │ OUT ────────────────── A0 (Nano)
  │ GND ────────────────── GND Rail
  └──────────────────┘

CQRSENTDS01 (TDS):
  ┌──────────────────┐
  │ CQRSENTDS01      │
  │                  │
  │ VCC ────────────────── 5V Rail
  │ OUT ────────────────── A1 (Nano)
  │ GND ────────────────── GND Rail
  └──────────────────┘


RS-485 COMMUNICATION:
─────────────────────
  ┌──────────────────────────┐
  │  MAX485 / SN75176        │
  │  RS-485 Transceiver      │
  │                          │
  │ VCC ────────────────────────── 5V Rail
  │ GND ────────────────────────── GND Rail
  │                          │
  │ RO (Receiver Out) ──────────── D7 (Nano RX)
  │ DI (Driver In) ─────────────── D6 (Nano TX)
  │ DE (Driver Enable) ──┬──────── D5 (Nano)
  │ RE (Receiver Enable) ┘         (Combined control)
  │                          │
  │ A ──────────────────────────── CAT7 Pin 5 (Blue-White)
  │ B ──────────────────────────── CAT7 Pin 6 (Green)
  └──────────────────────────┘

  Optional (for cable >50m):
    A ────[120Ω]──── B  (Termination resistor)


COMPLETE CONNECTION MATRIX:
───────────────────────────

Component Pin    → Arduino Pin  → Function
─────────────────────────────────────────────────────────────
LM2596 OUT+      → VIN          → 5V Power Supply
LM2596 OUT-      → GND          → Ground Reference
JSN-SR04T VCC    → 5V           → Power
JSN-SR04T TRIG   → D2           → Trigger Output
JSN-SR04T ECHO   → D3           → Echo Input
JSN-SR04T GND    → GND          → Ground
DS18B20 VCC      → 5V           → Power
DS18B20 DATA     → D4 + 4.7kΩ   → OneWire Data + Pull-up
DS18B20 GND      → GND          → Ground
TSW-20M VCC      → 5V           → Power
TSW-20M OUT      → A0           → Analog Input
TSW-20M GND      → GND          → Ground
CQRSENTDS01 VCC  → 5V           → Power
CQRSENTDS01 OUT  → A1           → Analog Input
CQRSENTDS01 GND  → GND          → Ground
MAX485 VCC       → 5V           → Power
MAX485 RO        → D7           → Serial RX
MAX485 DI        → D6           → Serial TX
MAX485 DE/RE     → D5           → TX Enable Control
MAX485 GND       → GND          → Ground
MAX485 A         → CAT7 Pin 5   → RS-485 A Line
MAX485 B         → CAT7 Pin 6   → RS-485 B Line
```

### 2.2 Display Node - Vollständiger Schaltplan (Textform)

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    DISPLAY NODE SCHALTPLAN                               ║
╚══════════════════════════════════════════════════════════════════════════╝

POWER SUPPLY SECTION:
─────────────────────
                                ┌──────────────────────┐
  12V PSU (+) ────────────┬─────┤ IN+     LM2596   OUT+├───┬── 5V Rail
                          │     │                      │   │
                          │     │ Potentiometer        │   │
                         [C1]   │ (Voltage Adjust)     │  [C3]
                         100μ   │                      │  100μ
                          │     │                      │   │
  12V PSU (-) ────────────┴─────┤ IN-              OUT-├───┴── GND Rail
                                └──────────────────────┘
          │
          └───────────────────────────────── To Sensor Node (via CAT7)
                              CAT7 Pin 1+2: 12V+
                              CAT7 Pin 3+4: GND

  C1: 100μF/25V Electrolytic (Input filter)
  C3: 100μF/16V Electrolytic (Output filter)


MICROCONTROLLER SECTION:
────────────────────────
                  ┌─────────────────────────────────────────────┐
                  │   Arduino Mega 2560                         │
                  │   (ATmega2560)                              │
                  │                                             │
   5V Rail ───────┤ 5V                                      GND ├──── GND Rail
                  │                                             │
                  │ D19 (RX1) ───────────────────────────────────┼──── RO (RS-485)
                  │ D18 (TX1) ───────────────────────────────────┼──── DI (RS-485)
                  │ D10       ───────────────────────────────────┼──── DE/RE (RS-485)
                  │                                             │    (or to GND)
                  │ D17 (RX2) ───────────────────────────────────┼──── TX (Nextion)
                  │ D16 (TX2) ───────────────────────────────────┼──── RX (Nextion)
                  │                                             │
                  │ D0 (RX0) ─┐                                 │
                  │ D1 (TX0) ─┴───── USB (Debug, not connected) │
                  │                                             │
                  └─────────────────────────────────────────────┘


RS-485 COMMUNICATION:
─────────────────────
  ┌──────────────────────────┐
  │  MAX485 / SN75176        │
  │  RS-485 Transceiver      │
  │                          │
  │ VCC ────────────────────────── 5V Rail
  │ GND ────────────────────────── GND Rail
  │                          │
  │ RO (Receiver Out) ──────────── D19 (Mega RX1)
  │ DI (Driver In) ─────────────── D18 (Mega TX1)
  │ DE (Driver Enable) ──┬──────── D10 (Mega) OR
  │ RE (Receiver Enable) ┘         GND (RX-only mode)
  │                          │
  │ A ──────────────────────────── CAT7 Pin 5 (from Sensor Node)
  │ B ──────────────────────────── CAT7 Pin 6 (from Sensor Node)
  └──────────────────────────┘

  Optional (for cable >50m):
    A ────[120Ω]──── B  (Termination resistor)


NEXTION DISPLAY:
────────────────
  ┌──────────────────────────────────────┐
  │  Nextion NX4024T032                  │
  │  3.2" TFT Touch Display              │
  │                                      │
  │ VCC (5V) ────────────────────────────── 5V Rail
  │ GND      ────────────────────────────── GND Rail
  │                                      │
  │ TX ───────────────────────────────────── D17 (Mega RX2)
  │ RX ───────────────────────────────────── D16 (Mega TX2)
  │                                      │
  │ [Note: TX of Display goes to RX      │
  │  of Mega, and vice versa]            │
  └──────────────────────────────────────┘

  Nextion Power Consumption:
  - Typical: 85mA @ 5V = 0.425W
  - Peak (backlight max): 100mA @ 5V = 0.5W
  - Standby (screen off): 30mA @ 5V = 0.15W


12V POWER SUPPLY:
─────────────────
  Input:  100-240V AC, 50/60Hz
  Output: 12V DC, 2A (24W)

  Recommended Specifications:
  - Universal Input (100-240V AC)
  - Output: 12V / 2A minimum
  - Protection: Over-Voltage, Over-Current, Short-Circuit
  - Efficiency: >80% (preferably >85%)
  - Safety: UL/CE/TÜV certified
  - Connector: 5.5mm × 2.1mm barrel jack (standard)
  
  Suggested Models:
  - Meanwell GST25E12-P1J
  - TDK-Lambda LS25-12
  - Traco Power TMDC 30-2412


COMPLETE CONNECTION MATRIX:
───────────────────────────

Component Pin       → Mega Pin    → Function
─────────────────────────────────────────────────────────────
12V PSU (+)         → LM2596 IN+  → 12V Power Input
12V PSU (-)         → LM2596 IN-  → Ground Input
12V PSU (+)         → CAT7 1+2    → 12V to Sensor Node
12V PSU (-)         → CAT7 3+4    → GND to Sensor Node
LM2596 OUT+         → 5V          → 5V Power Rail (Mega, Nextion, RS-485)
LM2596 OUT-         → GND         → Ground Rail
MAX485 VCC          → 5V          → Power
MAX485 RO           → D19 (RX1)   → Serial Receive from Sensor Node
MAX485 DI           → D18 (TX1)   → Serial Transmit (optional)
MAX485 DE/RE        → GND or D10  → RX Mode (GND) or Controlled (D10)
MAX485 GND          → GND         → Ground
MAX485 A            → CAT7 Pin 5  → RS-485 A Line
MAX485 B            → CAT7 Pin 6  → RS-485 B Line
Nextion VCC         → 5V          → Power
Nextion GND         → GND         → Ground
Nextion TX          → D17 (RX2)   → Display → Mega
Nextion RX          → D16 (TX2)   → Mega → Display
```

---

## 3. Komponenten-Spezifikationen

### 3.1 LM2596 Buck Converter (Spannungswandler)

**Vollständige Technische Spezifikation:**

| Parameter | Min | Typ | Max | Einheit | Notizen |
|-----------|-----|-----|-----|---------|---------|
| Eingangsspannung | 4.5 | 12 | 40 | V | BioSync: 12V |
| Ausgangsspannung | 1.25 | 5.0 | 37 | V | Einstellbar via Poti |
| Ausgangs strom | - | 3 | 3 | A | Mit Kühlkörper |
| Schaltfrequenz | - | 150 | - | kHz | Fixed |
| Wirkungsgrad | - | 92 | 96 | % | @ 12V→5V, 0.5A load |
| Ripple-Spannung | - | 30 | 50 | mV | @ 3A load |
| Line Regulation | - | 0.2 | 0.5 | % | VIN 8-18V |
| Load Regulation | - | 0.5 | 1.0 | % | IOUT 0-3A |
| Betriebstemperatur | -40 | 25 | 125 | °C | Junction temp |

**Pinbelegung LM2596-Modul:**

```
  ┌─────────────────────────┐
  │      LM2596 Module      │
  │                         │
  │  IN+  ○  (12V Input)    │
  │  IN-  ○  (GND Input)    │
  │                         │
  │  OUT+ ○  (5V Output)    │
  │  OUT- ○  (GND Output)   │
  │                         │
  │      [Potentiometer]    │
  │       (Voltage Adj.)    │
  │                         │
  │      [Inductor L1]      │
  │                         │
  └─────────────────────────┘
```

**Einstellprozedur (VOR Installation):**

1. **Vorbereitung:**
   - Multimeter auf DC Voltage (20V Bereich)
   - Kleiner Schlitzschraubendreher
   - LM2596-Modul ohne Last
   
2. **Anschluss:**
   - IN+ an 12V Quelle (z.B. Labornetzteil)
   - IN- an GND
   - Multimeter: Red probe to OUT+, Black to OUT-
   
3. **Einstellung:**
   - Potentiometer langsam drehen (Uhrzeigersinn = höher)
   - Beobachten Multimeter-Anzeige
   - Ziel: **5,00V** (±0.05V tolerierbar)
   - WICHTIG: Nie über 5,3V einstellen!
   
4. **Verifikation:**
   - Spannungsquelle trennen, 10 Sekunden warten
   - Erneut anschließen und Spannung messen
   - Sollte stabil 5,00V ±0.02V sein
   
5. **Markierung:**
   - Position des Potentiometers mit wasserfestem Marker markieren
   - Modul beschriften: "5V - SensorNode" oder "5V - DisplayNode"

**Thermische Überlegungen:**

- **Verlustleistung bei BioSync:**
  ```
  P_loss = (V_in × I_in) - (V_out × I_out)
  
  Sensor Node:
  P_loss = (12V × 0.027A) - (5V × 0.057A)
         = 0.324W - 0.285W = 0.039W
  
  Display Node:
  P_loss = (12V × 0.062A) - (5V × 0.136A)
         = 0.744W - 0.680W = 0.064W
  ```

- **Temperaturanstieg:**
  ```
  θ_ja (junction-to-ambient) ≈ 50°C/W (ohne Kühlkörper)
  
  ΔT = P_loss × θ_ja
  
  Sensor Node:  ΔT = 0.039W × 50 = 2°C
  Display Node: ΔT = 0.064W × 50 = 3°C
  
  → Kühlkörper NICHT erforderlich bei BioSync-Last
  ```

### 3.2 MAX485 / SN75176 RS-485 Transceiver

**Vollständige Technische Spezifikation:**

| Parameter | Min | Typ | Max | Einheit | Notizen |
|-----------|-----|-----|-----|---------|---------|
| Versorgungsspannung | 4.75 | 5.0 | 5.25 | V | |
| Stromaufnahme (RX) | - | 0.3 | 1.0 | mA | Receive Mode |
| Stromaufnahme (TX) | - | 3.0 | 5.0 | mA | Transmit Mode |
| Datenrate | 0 | - | 2.5 | Mbps | BioSync: 9600 bps |
| Kabellänge | - | - | 1200 | m | @ 100 kbps |
| Empfänger-Eingangs-Impedanz | - | 12 | - | kΩ | High-Z |
| Differenzielle Ausgangsspannung | 1.5 | - | 5.0 | V | VOD |
| Common-Mode Spannung | -7 | - | +12 | V | Toleriert |
| Slew Rate Limit | - | - | 300 | V/μs | Reduziert EMI |
| ESD-Schutz | - | - | ±15 | kV | Human Body Model |

**Pinbelegung MAX485-Modul:**

```
  ┌─────────────────────────┐
  │     MAX485 Module       │
  │     (8-Pin DIP/SO)      │
  │                         │
  │  1 RO ○  Receiver Out   ├─── To MCU RX
  │  2 RE ○  Receiver Enable├─┐
  │  3 DE ○  Driver Enable  ├─┴─ To MCU DE (or GND for RX-only)
  │  4 DI ○  Driver In      ├─── From MCU TX
  │                         │
  │  5 GND○  Ground         ├─── GND Rail
  │  6 A  ○  Non-inverting  ├─── To CAT7 Pin 5
  │  7 B  ○  Inverting      ├─── To CAT7 Pin 6
  │  8 VCC○  +5V            ├─── 5V Rail
  │                         │
  └─────────────────────────┘
```

**Truth Table:**

| DE | RE | Driver | Receiver | Typical Application |
|----|----|--------|----------|---------------------|
| 0  | 1  | OFF    | OFF      | Shutdown (not used) |
| 0  | 0  | OFF    | ON       | **Receive Mode** (Display Node) |
| 1  | 0  | ON     | ON       | Loopback (testing) |
| 1  | 1  | ON     | OFF      | **Transmit Mode** (Sensor Node) |

**Biasing und Termination:**

Für Kabelstrecken > 50m empfohlen:

```
  Node A (Sensor)              Cable              Node B (Display)
  ┌──────────────┐                                ┌──────────────┐
  │              │                                │              │
  │         A ○──┴────────────────────────────────┴──○ A        │
  │         B ○──┬────────────────────────────────┬──○ B        │
  │              │                                │              │
  │        [120Ω]                                 │ [120Ω]      │
  │              │                                │              │
  └──────────────┘                                └──────────────┘
  
  Optional Biasing (für sehr lange Kabel oder hohe EMI):
  
    5V ────[560Ω]──── A
    
    GND───[560Ω]──── B
```

**Hinweise:**
- Biasing stabilisiert Bus im Idle-State
- Termination verhindert Signalreflexionen
- Für BioSync (typisch <50m): Termination optional


### 3.3 Nextion NX4024T032 Display

**Vollständige Technische Spezifikation:**

| Parameter | Wert | Einheit | Notizen |
|-----------|------|---------|---------|
| Modell | NX4024T032 | - | 3.2" Basic Series |
| Display-Größe | 3.2 | Zoll | Diagonal |
| Auflösung | 320 × 240 | px | QVGA |
| Farbtiefe | 65K | Farben | 16-bit RGB565 |
| Touch-Typ | Resistive | - | 4-Wire resistive |
| Controller | STM32F030 | - | ARM Cortex-M0 |
| Flash Storage | 16 MB | Byte | Für HMI-Projekt |
| Kommunikation | UART TTL | - | 3.3V/5V kompatibel |
| Standard-Baudrate | 9600 | bps | Einstellbar: 2400-921600 |
| Versorgungsspannung | 4.5 - 5.5 | V DC | Typ: 5.0V |
| Stromaufnahme (Typ) | 85 | mA | @ 5V, 50% Brightness |
| Stromaufnahme (Max) | 100 | mA | @ 5V, 100% Brightness |
| Abmessungen | 99.4 × 60 × 9.7 | mm | W×H×D |
| Montage | 4× M3 Löcher | - | Befestigung |
| Betriebstemperatur | -10 bis +70 | °C | |
| Gewicht | ~80 | g | |

**Pinbelegung (4-Pin Connector):**

```
  Nextion Rear View:
  
  ┌─────────────────────────────────┐
  │     Nextion NX4024T032          │
  │     3.2" TFT Display            │
  │                                 │
  │         [Display Area]          │
  │                                 │
  │                                 │
  │  ┌───┐                          │
  │  │1 2│ ← 4-Pin Connector        │
  │  │3 4│                          │
  │  └───┘                          │
  └─────────────────────────────────┘
  
  Pin 1: TX    (Yellow) → To MCU RX (Mega D17)
  Pin 2: RX    (Blue)   → To MCU TX (Mega D16)
  Pin 3: VCC   (Red)    → 5V Rail
  Pin 4: GND   (Black)  → GND Rail
  
  WICHTIG: TX vom Display geht zu RX am MCU!
```

**Kommunikationsprotokoll:**

Nextion verwendet ein einfaches ASCII-Protokoll mit 3-Byte-Terminator:

```
Command Format:
  <ASCII Command String><0xFF><0xFF><0xFF>

Beispiel: Text-Update
  Arduino → Nextion:
  "tDist.txt=\"125.3 cm\"" + 0xFF + 0xFF + 0xFF
  
  Byte-Sequenz:
  74 44 69 73 74 2E 74 78 74 3D 22 31 32 35 2E 33 20 63 6D 22 FF FF FF
  t  D  i  s  t  .  t  x  t  =  "  1  2  5  .  3     c  m  "  [3xFF]

Touch-Event vom Display:
  Nextion → Arduino:
  65 00 01 00 FF FF FF
  │  │  │  │  └──┴──┴─ Terminator
  │  │  │  └─ Component ID
  │  │  └─ Page ID
  │  └─ Event Type (0x00 = Touch Press)
  └─ Return Code (0x65 = Touch Event)
```

---

## 4. Stromversorgungssystem

### 4.1 Power Budget Analyse (Detailliert)

**Sensor Node Power Consumption:**

| Komponente | Mode | Spannung | Strom | Leistung | Duty Cycle | Ø Leistung |
|------------|------|----------|-------|----------|------------|------------|
| ATmega4809 | Active | 5V | 30 mA | 0.150 W | 100% | 0.150 W |
| JSN-SR04T | Standby | 5V | 2 mA | 0.010 W | 98% | 0.010 W |
| JSN-SR04T | Measuring | 5V | 15 mA | 0.075 W | 2% | 0.002 W |
| DS18B20 | Idle | 5V | 0.1 mA | 0.0005 W | 85% | 0.0004 W |
| DS18B20 | Conversion | 5V | 1 mA | 0.005 W | 15% | 0.0008 W |
| TSW-20M | Continuous | 5V | 5 mA | 0.025 W | 100% | 0.025 W |
| CQRSENTDS01 | Continuous | 5V | 5 mA | 0.025 W | 100% | 0.025 W |
| MAX485 | Receive | 5V | 0.5 mA | 0.0025 W | 99% | 0.0025 W |
| MAX485 | Transmit | 5V | 5 mA | 0.025 W | 1% | 0.0003 W |
| **TOTAL** | | **5V** | **≈57 mA** | **0.285 W** | | **0.216 W avg** |

**LM2596 (Sensor Node) Input Power:**
```
Efficiency: 92%
Input Power = Output Power / Efficiency
            = 0.285 W / 0.92
            = 0.310 W @ 12V
            = 25.8 mA @ 12V
```

**Display Node Power Consumption:**

| Komponente | Mode | Spannung | Strom | Leistung | Duty Cycle | Ø Leistung |
|------------|------|----------|-------|----------|------------|------------|
| ATmega2560 | Active | 5V | 50 mA | 0.250 W | 100% | 0.250 W |
| Nextion | 50% Brightness | 5V | 85 mA | 0.425 W | 90% | 0.383 W |
| Nextion | Screensaver | 5V | 30 mA | 0.150 W | 10% | 0.015 W |
| MAX485 | Receive | 5V | 1 mA | 0.005 W | 100% | 0.005 W |
| **TOTAL** | | **5V** | **≈136 mA** | **0.680 W** | | **0.653 W avg** |

**LM2596 (Display Node) Input Power:**
```
Efficiency: 92%
Input Power = 0.680 W / 0.92 = 0.739 W @ 12V = 61.6 mA @ 12V
```

**System Total @ 12V:**
```
12V PSU Total Load:
  Display Node:   61.6 mA
  Sensor Node:    25.8 mA
  ─────────────────────────
  TOTAL:          87.4 mA @ 12V = 1.049 W

Safety Margin: 2000 mA / 87.4 mA = 22.9x
```

### 4.2 Spannungsabfall im CAT7-Kabel

**Kabelwiderstand Berechnung:**

CAT7-Kabel: AWG 23 Kupferleiter
- Widerstand: ≈0.057 Ω/m pro Ader bei 20°C
- BioSync nutzt 2 Adern parallel für 12V+ und 2 für GND
- Effektiver Widerstand: R_eff = R_ader / 2

```
Für 50m Kabel:
  R_single_wire = 0.057 Ω/m × 50 m = 2.85 Ω
  R_parallel = 2.85 Ω / 2 = 1.425 Ω (für 2 parallele Adern)
  R_total = R_12V+ + R_GND = 1.425 Ω + 1.425 Ω = 2.85 Ω

Spannungsabfall:
  ΔV = I × R_total
     = 0.0258 A × 2.85 Ω
     = 0.074 V = 74 mV

Spannung am Sensor Node:
  V_sensor = 12.0 V - 0.074 V = 11.926 V
  
  → Immer noch weit über LM2596 Minimum (4.5V)
  → Kein Problem für Betrieb
```

**Für 100m Kabel (Worst Case):**
```
  R_total = 5.7 Ω
  ΔV = 0.0258 A × 5.7 Ω = 0.147 V = 147 mV
  V_sensor = 12.0 V - 0.147 V = 11.853 V
  
  → Immer noch ausgezeichnet
```

### 4.3 Inrush Current (Einschaltstrom)

Bei Systemstart können kurzzeitig höhere Ströme fließen:

**Komponenten mit Inrush:**
- LM2596 Input-Kondensatoren (100μF): ~200mA für 1-2ms
- Nextion Display: ~150mA für 100-200ms (Backlight start)
- Sensor Initialization: ~100mA für 50ms

**Peak Inrush (geschätzt):**
```
  I_inrush_peak ≈ 400-500 mA @ 12V für < 200ms
  
  12V/2A PSU kann dies problemlos handhaben
```

**Soft-Start (optional, für höhere Zuverlässigkeit):**
- NTC-Thermistor (10Ω, 2A) in series mit 12V+
- Begrenzt Inrush auf ~1.2A @ 12V
- Widerstand fällt nach Erwärmung auf <0.5Ω

---

## 5. RS-485 Bus-Konfiguration

### 5.1 RS-485 Spezifikation (TIA/EIA-485-A)

**Standard-Parameter:**

| Parameter | Wert | Einheit | BioSync Setting |
|-----------|------|---------|-----------------|
| Baudrate | 2400 - 10 Mbps | bps | 9600 |
| Topologie | Multi-Drop Bus | - | Point-to-Point |
| Max. Nodes | 32 (Standard), 256 (Erweitert) | - | 2 |
| Max. Kabellänge | 1200 m @ 100 kbps | m | typ. <100m |
| Signalpegel (VOD) | 1.5 - 5 V | V differential | Typ. 3V |
| Common-Mode Range | -7 to +12 | V | |
| Impedanz | 120 Ω | Ω differential | Matched |
| Data Format | Async Serial | - | 8N1 |

**Differentielles Signal:**

```
  Differential Voltage (VOD) = V(A) - V(B)
  
  Logic "1" (MARK):   V(A) > V(B)    →  VOD > +200 mV
  Logic "0" (SPACE):  V(A) < V(B)    →  VOD < -200 mV
  
  Idle State: Bus floating or biased
  
  Signalform:
  
       V(A)    ────┐     ┌────┐     ┌────
                   │     │    │     │
       V(B)        └─────┘    └─────┘
  
                  "0"  "1" "0" "1"  "0"
  
  VOD = V(A)-V(B)
       
       +5V  ────┐     ┌────┐     ┌────
                │     │    │     │
         0      ┼─────┼────┼─────┼
                │     │    │     │
       -5V      └─────┘    └─────┘
```

### 5.2 Kabelspezifikationen für RS-485

**CAT7 S/FTP (Shielded Foiled Twisted Pair):**

| Eigenschaft | Wert | Notizen |
|-------------|------|---------|
| Kategorie | CAT7 / Class F | ISO/IEC 11801 |
| Frequenzbereich | bis 600 MHz | Überqualifiziert für 9600 bps |
| Aufbau | 4× Twisted Pairs | Jedes Paar einzeln geschirmt |
| Gesamtschirmung | Ja (Geflecht) | Zusätzlich zur Paarschirmung |
| Impedanz | 100 Ω @ 100 MHz | Für Ethernet; RS-485: 120Ω nominal |
| Dämpfung | <2.0 dB/100m @ 100 MHz | Sehr niedrig |
| Verdrillungs rate | >50 Twists/m | Reduziert Crosstalk |
| Außendurchmesser | ~7-8 mm | Robust |
| Adernquerschnitt | AWG 23 | 0.57 mm² |
| Max. Zugkraft | ~80 N | Für Installation |
| Temperaturbereich | -40 to +75°C | Outdoor-tauglich |

**Warum CAT7 für BioSync?**

1. **Einzelne Paarschirmung:** Jedes Twisted Pair hat eigene Folie
   - RS-485 A/B Paar ist EMI-geschützt
   - Power-Adern verursachen keine Störungen auf Daten-Adern

2. **Gesamtschirmung:** Zusätzliche Abschirmung gegen externe EMI
   - Schutz vor Blitzeinschlägen (nahfeld)
   - Schutz vor Motoren, Pumpen (Klärgrube)

3. **Niedrige Kapazität:** ~55 pF/m
   - Erlaubt höhere Datenraten (falls später benötigt)
   - Weniger Signalverzerrung

4. **Mechanische Robustheit:** PVC-Mantel
   - UV-beständig (für Außenverlegung)
   - Widerstandsfähig gegen Feuchtigkeit

5. **Zukunftssicher:** Überqualifiziert
   - Spätere Erweiterungen möglich (z.B. Ethernet über gleiche Leitung)
   - Langlebigkeit (>20 Jahre erwartete Lebensdauer)

### 5.3 Schirmung und Erdung

**Schirm-Anschluss (wichtig für EMV):**

```
  Best Practice: Schirm an EINEM Ende erden
  
  Display Node (Haus):         Sensor Node (Schacht):
  ┌──────────────────┐         ┌──────────────────┐
  │                  │  CAT7   │                  │
  │  RS-485 Modul    │  Cable  │  RS-485 Modul    │
  │    A ────────────┼─────────┼──── A            │
  │    B ────────────┼─────────┼──── B            │
  │                  │         │                  │
  │  Gehäuse-GND ────┼─┐       │                  │
  │  (an PE)         │ │       │  (floating)      │
  │                  │ │       │                  │
  └──────────────────┘ │       └──────────────────┘
                       │
                   Schirm
                   (geerdet)

  Wichtig:
  - Schirm an Display Node (Haus) erden
  - Sensor Node Schirm NOT verbunden (vermeidet Ground Loop)
  - Wenn beide Enden geerdet: Risiko von Ausgleichsströmen
```

**Ground Loop Vermeidung:**

Ground Loops entstehen wenn mehrere Erdungspunkte existieren:
```
  Display Node GND ─────┐
                        │ ΔV (z.B. 0.5V)
  Sensor Node GND ──────┘
  
  Wenn Schirm beide Enden verbindet:
    I_loop = ΔV / R_schirm
           = 0.5V / 0.1Ω  (beispiel)
           = 5A !!! (Kurzschluss)
           
  Lösung: Schirm nur einseitig erden
```

---

## 6. CAT7-Kabel: Adernbelegung

### 6.1 Detaillierte Adernzuordnung

**Standard CAT7 Farb-Code (TIA/EIA-568-B):**

| Adernpaar | Ader | Farbe | BioSync Funktion | Begründung |
|-----------|------|-------|------------------|------------|
| Paar 1 | 1 | Orange-Weiß | **12V+** | Parallel mit Ader 2 |
| Paar 1 | 2 | Orange | **12V+** | Verdoppelt Stromkapazität |
| Paar 2 | 3 | Grün-Weiß | **GND** | Parallel mit Ader 4 |
| Paar 2 | 4 | Blau | **GND** | Reduziert Spannungsabfall |
| Paar 3 | 5 | Blau-Weiß | **RS-485 A** | Twisted Pair für Daten |
| Paar 3 | 6 | Grün | **RS-485 B** | Minimiert EMI |
| Paar 4 | 7 | Braun-Weiß | **Reserve** | Zukünftige Erweiterung |
| Paar 4 | 8 | Braun | **Reserve** | Z.B. I2C, SPI, GPIO |

**Rationale:**

1. **Adern 1+2 (12V+):**
   - Parallel geschaltet verdoppelt Stromtragfähigkeit
   - AWG 23: Single = 0.57 mm² → Parallel = 1.14 mm²
   - Max. Strom: Single ~1A, Parallel ~2A (sicher)
   - BioSync braucht nur 87 mA → Safety Factor 23x

2. **Adern 3+4 (GND):**
   - Gleiche Vorteile wie 12V+
   - Reduziert Spannungsabfall auf Rückleitung
   - Wichtig für stabilen Referenzpotential

3. **Adern 5+6 (RS-485 A/B):**
   - MÜSSEN Twisted Pair sein (EMI-Immunität)
   - Paar 3 ist optimal (zentral im Kabel)
   - Verdrillung: >50 Twists/m reduziert Abstrahllung

4. **Adern 7+8 (Reserve):**
   - Paar 4 für zukünftige Erweiterungen
   - Mögliche Nutzung:
     - I2C (SDA/SCL) für zusätzliche Sensoren
     - SPI für SD-Karten-Logger
     - GPIO für Relais-Steuerung (Pumpe)
     - Analog Input für weiteren Sensor

### 6.2 Kabelverlegung Best Practices

**Mechanischer Schutz:**

1. **Indoor (im Haus):**
   - Kabelkanal entlang Wand
   - Befestigung alle 50 cm (Kabelbinder)
   - Biegeradius: min. 4× Kabeldurchmesser (≥30 mm)
   - Keine scharfen Kanten

2. **Outdoor (Haus → Schacht):**
   - **Option A: Erdverlegung**
     - Schutzrohr (z.B. Wellrohr DN25)
     - Verlegetiefe: 60-80 cm (frostfrei)
     - Sandbett oben/unten
     - Warnband "Achtung Kabel" 20 cm über Rohr
   
   - **Option B: Oberirdisch**
     - Kabelbrücke oder Kabelrinne
     - UV-beständiges Kabel (CAT7 ist PVC)
     - Befestigung alle 1 m
     - Zugentlastung an Durchführungen

3. **Durchführung Gehäuse:**
   - Kabelverschraubung M12 oder M16
   - Dichtung aus Gummi/Silikon
   - Tropfschlaufe vor Eingang (Wasser läuft ab)

**Elektrischer Schutz:**

- **Blitzschutz (optional, empfohlen):**
  ```
  Grobschutz (Gebäude-Eingang):
    12V+ ────[Gas Discharge Tube 90V]──── GND
    
  Feinschutz (vor LM2596):
    12V+ ────[TVS-Diode P6KE18A]──── GND
    RS485-A ─[TVS-Diode SMCJ5.0A]─── GND
    RS485-B ─[TVS-Diode SMCJ5.0A]─── GND
  ```

---

## 7. PCB-Layout-Empfehlungen

### 7.1 Single-Layer PCB Design (für Breadboard-Prototyp)

Für permanenten Aufbau: Lochrasterplatine (Perfboard) oder Custom PCB

**Sensor Node Layout (Beispiel):**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  SENSOR NODE PCB LAYOUT (Top View)                      │
│                  Board Size: 100mm × 70mm                               │
└─────────────────────────────────────────────────────────────────────────┘

     5V Rail ═══════════════════════════════════════════════════════════
                       ║         ║         ║         ║
     ┌────────────────┐║         ║         ║         ║
     │ LM2596         │║         ║         ║         ║
     │ Buck Conv.     │║    ┌────┴─────┐  ║         ║
     │                │║    │ Arduino  │  ║    ┌────┴────┐
     │ IN+ ← (12V)    │║    │ Nano     │  ║    │ MAX485  │
     │ IN- ← (GND)    │║    │ Every    │  ║    │ RS-485  │
     │ OUT+→──────────║║    │          │  ║    │         │
     │ OUT-→──────┐   │║    │ VIN ←────║  ║    │ VCC ←───║
     └────────────┼───┘║    │ GND ←────║  ║    │ GND ←───║
                  │    ║    │ D2  ─────║──║────┼→ Sensor │
                  │    ║    │ D3  ─────║──║────┼→ Connct │
  GND Rail ═══════╧════║════│ D4  ─────║──║────┼→ 1-Wire │
                       ║    │ A0  ─────║──║────┼→ Turbid │
                       ║    │ A1  ─────║──║────┼→ TDS    │
                       ║    │ D6  ─────║──║────→ DI      │
                       ║    │ D7  ─────║──║────→ RO      │
                       ║    │ D5  ─────║──║────→ DE/RE   │
                       ║    │          │  ║    │ A  →CAT7│
                       ║    │          │  ║    │ B  →CAT7│
                       ║    └──────────┘  ║    └─────────┘
                       ║                  ║
     Sensor            ║                  ║
     Connectors:       ║                  ║
     ┌──────────────┐  ║                  ║
     │ JSN-SR04T    │  ║                  ║
     │ VCC ←────────║──╝                  ║
     │ TRIG ←───────║───── D2             ║
     │ ECHO ←───────║───── D3             ║
     │ GND ←────────║─────────────────────╝
     └──────────────┘
     
     ┌──────────────┐       ┌ 4.7kΩ Pull-up ┐
     │ DS18B20      │       │                │
     │ VCC ←────────║───────┴─── 5V          │
     │ DATA ←───────║──────────── D4         │
     │ GND ←────────║──────────── GND        │
     └──────────────┘
     
     [Similar for TSW-20M and CQRSENTDS01]

LAYOUT RULES:
─────────────
1. Power Rails: Top/Bottom, wide traces (2mm)
2. Signal Traces: 0.5mm minimum
3. Sensor Cables: Twisted, keep short (<20cm)
4. Ground Plane: Maximum copper pour
5. Decoupling Caps: Close to IC VCC pins (<5mm)
```

### 7.2 PCB Design Guidelines

**Trace Width Calculation:**

```
For Current Capacity:
  I = k × ΔT^0.44 × A^0.725
  
  Where:
    I = Current (A)
    k = Constant (0.048 for external, 0.024 for internal layers)
    ΔT = Temperature rise (°C)
    A = Cross-sectional area (mil²)
  
  For BioSync (External layer, 1oz copper):
    Max current on 5V rail: 150mA
    Allowed temp rise: 10°C
    
    Width = 0.5mm (20 mil) is SAFE (can handle >1A)
    
  For power rails (12V, 5V), use wider traces:
    Recommended: 2mm (80 mil) for aesthetics and low resistance
```

**Ground Plane:**

- Pour ground plane on Bottom Layer
- Connect all GND pins with vias
- Minimize ground loops (star topology from LM2596 OUT-)

**Component Placement:**

1. **Power first:** LM2596 near 12V input
2. **MCU central:** Arduino in center
3. **Peripherals around:** RS-485, sensor connectors at edges
4. **Decoupling caps:** 100nF ceramic caps at each VCC pin
5. **Minimize trace lengths:** Especially for high-speed (RS-485)

### 7.3 EMI/EMC Considerations

**Noise Sources:**

- LM2596 Switching (150 kHz) → Radiated EMI
- RS-485 Edges (fast rise/fall) → Conducted EMI
- Sensors (pumps, motors nearby) → External EMI

**Mitigation:**

1. **Filtering on Power Input:**
   ```
   12V Input ───┬──[100μF]──┬─── To LM2596 IN+
                │           │
              [Ferrite]   [TVS]
              Bead 100Ω   P6KE18A
                │           │
   GND ─────────┴───────────┴─── GND
   ```

2. **Shielding:**
   - Metal enclosure for both nodes
   - Connect enclosure to PE (Protective Earth) at Display Node
   - Sensor Node enclosure floating (avoid ground loops)

3. **Cable Routing:**
   - Keep power and data cables separated (min. 5cm)
   - Cross at 90° if unavoidable
   - Use shielded cable (CAT7) for data

