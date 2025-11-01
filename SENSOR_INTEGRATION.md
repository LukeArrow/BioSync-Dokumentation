# BioSync - Sensor Integration Guide
## Detaillierte Sensor-Dokumentation und Integration

**Version:** 2.0  
**Datum:** November 2025  
**Für:** Hardware-Techniker, Entwickler, Wartungspersonal

---

## Inhaltsverzeichnis

1. [Sensor-Übersicht](#1-sensor-übersicht)
2. [JSN-SR04T Ultraschallsensor](#2-jsr-sr04t-ultraschallsensor)
3. [DS18B20 Temperatursensor](#3-ds18b20-temperatursensor)
4. [TSW-20M Trübungssensor](#4-tsw-20m-trübungssensor)
5. [CQRSENTDS01 TDS-Sensor](#5-cqrsentds01-tds-sensor)
6. [Sensor-Wartung](#6-sensor-wartung)
7. [Fehlerdiagnose](#7-fehlerdiagnose)

---

## 1. Sensor-Übersicht

### 1.1 Sensor-Matrix

| Sensor | Messgröße | Typ | Schnittstelle | Update-Rate | Genauigkeit |
|--------|-----------|-----|---------------|-------------|-------------|
| JSN-SR04T | Füllstand | Ultraschall | Digital (TRIG/ECHO) | 5s | ±1 cm |
| DS18B20 | Temperatur | Digital-Temp | 1-Wire | 5s | ±0.5°C |
| TSW-20M | Trübung | Optisch | Analog (A0) | 5s | ±5% |
| CQRSENTDS01 | TDS/Leitfähigkeit | Konduktiv | Analog (A1) | 5s | ±10% |

### 1.2 Elektrische Anforderungen

**Alle Sensoren benötigen 5V Spannungsversorgung:**

| Sensor | Versorgung | Stromaufnahme | Peak | Standby |
|--------|------------|---------------|------|---------|
| JSN-SR04T | 5V ±10% | 15 mA | 30 mA (Messung) | 2 mA |
| DS18B20 | 3-5.5V | 1 mA | 1.5 mA (Conversion) | 0.75 μA |
| TSW-20M | 5V ±5% | 5 mA | 5 mA (kontinuierlich) | - |
| CQRSENTDS01 | 5V ±5% | 5 mA | 5 mA (kontinuierlich) | - |
| **SUMME** | **5V** | **≈26 mA** | **≈46 mA** | **≈2 mA** |

---

## 2. JSN-SR04T Ultraschallsensor

### 2.1 Funktionsprinzip

Der JSN-SR04T nutzt **Ultraschall-Laufzeitmessung** zur Distanzbestimmung:

```
Arbeitsweise:
  1. Trigger: Arduino sendet 10μs HIGH-Puls an TRIG
  2. Sensor: Sendet 8× 40kHz Ultraschall-Burst
  3. Echo: Ultraschall reflektiert von Oberfläche zurück
  4. Sensor: Setzt ECHO-Pin HIGH für Dauer der Laufzeit
  5. Arduino: Misst ECHO-Puls-Dauer
  6. Berechnung: Distance = (Duration × Speed_of_Sound) / 2

Physik:
  Schallgeschwindigkeit in Luft (20°C): v = 343 m/s = 0.0343 cm/μs
  
  Für Hin- und Rückweg: d = (t × v) / 2
                          d = (t × 0.0343) / 2
                          d = t / 58.3  [cm wenn t in μs]
  
  Vereinfacht in Code: distance = duration / 58
```

**Timing-Diagramm:**

```
TRIG Pin:        ┌──┐
                 │  │ 10μs
              ───┘  └────────────────────────────────
                    
                    ↓ Sensor sendet Ultraschall-Burst
                    
ECHO Pin:               ┌────────────────┐
                        │  Dauer ∝ Dist. │
              ──────────┘                └──────────
                        
                        ← →
                     Duration (μs)
                     
Distance = Duration / 58 [cm]

Beispiel: Distanz = 100 cm
  → Duration = 100 × 58 = 5800 μs = 5.8 ms
```

### 2.2 Hardware-Integration

**Pinbelegung:**

```
JSN-SR04T → Arduino Nano Every

  VCC (Rot)    → 5V (von LM2596)
  TRIG (Braun) → D2
  ECHO (Gelb)  → D3
  GND (Schwarz)→ GND
  
Kabel: 2.5m Standardlänge, erweiterbar auf 5m
```

**Montage:**

```
Einbau-Position:

            Schacht-Deckel
   ╔════════════════════════════════╗
   ║                                ║
   ║   [Sensor Montage-Halterung]   ║
   ║            │                   ║
   ║            ▼                   ║
   ║       ┌─────────┐              ║
   ║       │ JSN-SR04T              ║
   ║       │ Transducer│            ║
   ║       └─────┬─────┘            ║
   ║             │                  ║
   ║             │ Ultraschall      ║
   ║             ▼ (75° Kegel)      ║
   ║                                ║
   ║                                ║
   ║     ≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈≈     ║
   ║     Wasseroberfläche           ║
   ╚════════════════════════════════╝

Wichtig:
  - Senkrecht nach unten ausrichten (±5°)
  - Mindestens 25 cm über höchstem Wasserstand
  - Keine Hindernisse im 75° Kegelwinkel
  - Befestigung vibrationsfrei
  - Wasserdicht montieren (Transducer ist IP68)
```

**Software-Integration:**

```cpp
// In config.h:
static const uint8_t TRIG_PIN = 2;
static const uint8_t ECHO_PIN = 3;

// In sensors.cpp:
void sensors_begin() {
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  digitalWrite(TRIG_PIN, LOW);
}

float measureDistance() {
  // Sensor reset
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  
  // Trigger pulse (10μs)
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  // Measure echo pulse duration (timeout 30ms = ~500cm)
  unsigned long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  
  // Timeout check
  if (duration == 0) {
    return -1.0;  // Error: No echo received
  }
  
  // Calculate distance: duration(μs) / 58 = distance(cm)
  float distance = duration / 58.0;
  
  return distance;
}
```

### 2.3 Kalibrierung

**Offset-Kalibrierung (Schritt-für-Schritt):**

1. **Vorbereitung:**
   - Maßband oder Zollstock (min. 2m)
   - Flache Referenzfläche (z.B. Brett)
   - Stabiler Messaufbau

2. **Durchführung:**
   ```
   Messung bei 3 bekannten Distanzen:
   
   Distanz  | Soll (cm) | Gemessen (cm) | Differenz (cm)
   ─────────┼───────────┼───────────────┼────────────────
   Nah      |   50.0    |     49.2      |    -0.8
   Mittel   |  100.0    |     99.5      |    -0.5
   Fern     |  200.0    |    199.3      |    -0.7
   ─────────┼───────────┼───────────────┼────────────────
   Mittelwert Offset:                    |    -0.67 cm
   ```

3. **Offset anwenden:**
   ```cpp
   // In sensors.cpp:
   const float DISTANCE_OFFSET = -0.67;  // Kalibrierter Offset
   
   float measureDistance() {
     // ... existing code ...
     float distance = duration / 58.0;
     distance += DISTANCE_OFFSET;  // Offset korrigieren
     return distance;
   }
   ```

4. **Validierung:**
   - Erneut messen bei 50, 100, 200 cm
   - Abweichung sollte <±0.5 cm sein

**Temperaturkompensation (Optional):**

```cpp
// Für höhere Genauigkeit: Schallgeschwindigkeit von Temperatur abhängig
float measureDistanceCompensated(float tempCelsius) {
  unsigned long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  if (duration == 0) return -1.0;
  
  // Schallgeschwindigkeit: v(T) = 331.3 + 0.606×T [m/s]
  float speedOfSound = 331.3 + 0.606 * tempCelsius;  // m/s
  float speedCmPerUs = speedOfSound / 10000.0;       // cm/μs
  
  float distance = (speedCmPerUs * duration) / 2.0;
  return distance;
}
```

### 2.4 Störeinflüsse und Vermeidung

| Störquelle | Auswirkung | Vermeidung |
|------------|------------|------------|
| Schaum auf Wasser | Dämpft Echo | Sensor höher montieren, auf Wasserspiegel zielen |
| Schräge Oberfläche | Echo wird weg reflektiert | Senkrecht montieren (±5°) |
| Hindernisse im Kegelwinkel | Falsche Messung | Freie Sicht auf Wasser, 75° Kegel beachten |
| Temperaturänderung | ±1% Fehler pro 10°C | Temperaturkompensation verwenden |
| Kondensation am Transducer | Signaldämpfung | Tropfschlaufe vor Sensor, IP68 nutzen |
| Elektrische Störungen | Sporadische Fehler | Shielded Cable, Ground Loop vermeiden |

### 2.5 Wartung

**Alle 3 Monate:**
- Transducer-Membran mit destilliertem Wasser reinigen (weiches Tuch)
- Kabel auf Beschädigungen prüfen
- Messgenauigkeit überprüfen (Referenzmessung)

**Jährlich:**
- Kalibrierung wiederholen
- Montagestabilität prüfen (Vibrationen?)
- Bei Genauigkeitsverlust: Sensor tauschen

**Lebensdauer:** ~5-7 Jahre bei richtiger Wartung

---

## 3. DS18B20 Temperatursensor

### 3.1 Funktionsprinzip

Der DS18B20 ist ein **digitaler Temperatursensor** mit **1-Wire Interface**:

```
Funktionsweise:
  1. Master (Arduino) sendet Befehl via 1-Wire: "Convert T"
  2. Sensor startet AD-Wandlung (intern)
  3. Conversion Time: 750ms @ 12-bit Auflösung
  4. Ergebnis wird in Sensor-RAM gespeichert
  5. Master liest Temperaturwert aus (9-Byte Scratchpad)
  
1-Wire Protokoll:
  - Nur 1 Datenleitung (+ GND)
  - Master sendet Reset-Puls → Slave antwortet mit Presence-Puls
  - Bidirektionales Protokoll über eine Leitung
  - Mehrere Slaves möglich (ROM-Adressierung)
  - Pull-up Widerstand erforderlich (4.7kΩ)
```

**Internes Blockdiagramm:**

```
┌────────────────────────────────────────┐
│         DS18B20 Internals              │
│                                        │
│  ┌──────────────┐   ┌──────────────┐  │
│  │  Temperature │   │   12-bit     │  │
│  │  Sensor      │──→│   ADC        │  │
│  │  (Bandgap)   │   │              │  │
│  └──────────────┘   └──────┬───────┘  │
│                             │          │
│  ┌──────────────────────────▼───────┐ │
│  │      Scratchpad Memory           │ │
│  │      (9 Bytes)                   │ │
│  │  Byte 0-1: Temperature (LSB/MSB)│ │
│  │  Byte 2-3: TH/TL Alarm Registers│ │
│  │  Byte 4:   Configuration         │ │
│  │  Byte 5-7: Reserved              │ │
│  │  Byte 8:   CRC                   │ │
│  └────────────────┬─────────────────┘ │
│                   │                   │
│  ┌────────────────▼─────────────────┐ │
│  │      1-Wire Interface            │ │
│  │      (DQ Pin)                    │ │
│  └────────────┬─────────────────────┘ │
└───────────────┼───────────────────────┘
                │
         To Arduino D4 (+ 4.7kΩ to 5V)
```

### 3.2 Hardware-Integration

**Pinbelegung:**

```
DS18B20 (TO-92 Package or MICREEN Probe):

  Viewing from flat side:
  
   ┌─────┐
   │  ●  │  Pin 1: GND (Black/Schwarz)
   │  ●  │  Pin 2: DATA (Yellow/Gelb) → D4 + 4.7kΩ Pull-up
   │  ●  │  Pin 3: VCC (Red/Rot) → 5V
   └─────┘
   
MICREEN-DTSMK Probe (3-Wire):
  Red:    VCC → 5V
  Black:  GND → GND
  Yellow: DATA → D4
```

**Pull-up Widerstand (KRITISCH!):**

```
Schaltung:

    5V ────┬─────────────── VCC (DS18B20)
           │
         [4.7kΩ]  ← WICHTIG!
           │
           ├─────────────── DATA (DS18B20)
           │                   │
           │                   └──── D4 (Arduino)
           │
    GND ───┴─────────────── GND (DS18B20)

Warum 4.7kΩ?
  - 1-Wire Bus benötigt Pull-up für Open-Drain
  - Zu groß (>10kΩ): Signale zu langsam (Rise Time)
  - Zu klein (<1kΩ): Zu viel Strom, Bus-Überlastung
  - 4.7kΩ: Standard-Wert, optimales Timing
  
Ohne Pull-up:
  → Sensor wird nicht erkannt
  → Temperatur = -127.0°C (DEVICE_DISCONNECTED_C)
  → 1-Wire Kommunikation schlägt fehl
```

**Software-Integration:**

```cpp
// In config.h:
static const uint8_t DS18B20_PIN = 4;

// Bibliotheken einbinden:
#include <OneWire.h>
#include <DallasTemperature.h>

// In sensors.cpp:
static OneWire oneWire(DS18B20_PIN);
static DallasTemperature dsSensor(&oneWire);

void sensors_begin() {
  dsSensor.begin();
  delay(50);  // Initialisierung abwarten
}

float readTemperature() {
  // Temperaturmessung starten (non-blocking möglich)
  dsSensor.requestTemperatures();
  
  // Wert lesen (Index 0 = erster Sensor auf Bus)
  float temp = dsSensor.getTempCByIndex(0);
  
  // Fehlercheck
  if (temp == DEVICE_DISCONNECTED_C) {
    return -127.0;  // Error code
  }
  
  return temp;
}
```

**Timing:**

```
requestTemperatures() Blocking Time:
  
  12-bit Resolution (default): 750 ms
  11-bit Resolution:           375 ms
  10-bit Resolution:           188 ms
   9-bit Resolution:            94 ms
  
BioSync nutzt 12-bit für maximale Genauigkeit (±0.0625°C Auflösung)

Alternative (Non-blocking):
  dsSensor.setWaitForConversion(false);
  dsSensor.requestTemperatures();  // Startet Messung, kehrt sofort zurück
  delay(750);                      // Warten auf Conversion
  float temp = dsSensor.getTempCByIndex(0);
```

### 3.3 Kalibrierung

**Zwei-Punkt-Kalibrierung:**

1. **Referenzmessungen:**
   ```
   Eiswasser (0°C):
     Referenz: 0.0°C
     DS18B20:  0.3°C
     Offset:   +0.3°C
   
   Raumtemp (20°C):
     Referenz: 20.0°C (geeichtes Thermometer)
     DS18B20:  20.2°C
     Offset:   +0.2°C
   
   Mittelwert: +0.25°C
   ```

2. **Offset anwenden:**
   ```cpp
   const float TEMP_OFFSET = -0.25;  // Korrektur
   
   float readTemperature() {
     dsSensor.requestTemperatures();
     float temp = dsSensor.getTempCByIndex(0);
     if (temp == DEVICE_DISCONNECTED_C) return -127.0;
     
     temp += TEMP_OFFSET;  // Offset anwenden
     return temp;
   }
   ```

3. **Validierung:**
   - Mehrere Temperaturpunkte messen (0°C, 10°C, 20°C)
   - Abweichung sollte <±0.5°C sein

**Werkskalibration:**

DS18B20 ist werkseitig kalibriert:
- Genauigkeit: ±0.5°C (-10°C bis +85°C)
- Zusätzliche Kalibrierung verbessert auf ±0.2°C

