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

---

## 6. ALS-PT19 Lichtsensor (RelayNode)

### 6.1 Funktionsprinzip

Der ALS-PT19 ist ein analoger Ambient Light Sensor (Umgebungslichtsensor), der Lichtintensität in eine proportionale Spannung (0-VCC) umwandelt.

**Technologie:** Photodiode mit integriertem Verstärker

**Anwendung im BioSync System:** 
- Erkennung von LED-Zuständen an Schrack-Relays
- Unterscheidung zwischen LED AUS, LED AN, LED BLINKEND
- Event-basierte Statusmeldungen

### 6.2 Technische Spezifikationen

| Parameter | Wert | Einheit | Bemerkung |
|-----------|------|---------|-----------|
| **Versorgungsspannung** | 3.3 - 5.0 | V | Typ. 5V für Arduino |
| **Stromaufnahme** | 0.5 | mA | Sehr niedrig |
| **Ausgangssignal** | 0 - VCC | V | Analog, proportional zur Lichtintensität |
| **Spektralbereich** | 350 - 750 | nm | Sichtbares Licht |
| **Spitzensensitivität** | 560 | nm | Grüngelber Bereich |
| **Sichtwinkel** | ±60 | ° | Breiter Erfassungsbereich |
| **Ansprechzeit** | < 1 | ms | Sehr schnell |
| **Betriebstemperatur** | -30 bis +85 | °C | Industrietauglich |
| **Genauigkeit** | ±15 | % | Relativ zur Lichtintensität |

### 6.3 Pinbelegung und Anschluss

```
ALS-PT19 (Top View)
┌─────────────┐
│   1  2  3   │
└─────────────┘
  │  │  │
  │  │  └── GND    (Pin 3)
  │  └───── OUT    (Pin 2) → Analog Pin A0-A3
  └──────── VCC    (Pin 1) → 5V
```

#### Anschlussschema für RelayNode

**4 Sensoren parallel an Arduino Nano Every:**

| Sensor # | LED | VCC | OUT → Arduino | GND |
|----------|-----|-----|---------------|-----|
| 1 | PUMP | 5V | A0 | GND |
| 2 | VENT | 5V | A1 | GND |
| 3 | LED3 | 5V | A2 | GND |
| 4 | LED4 | 5V | A3 | GND |

**Hinweis:** Alle VCC können gemeinsam an 5V-Rail, alle GND an GND-Rail.

### 6.4 Mechanische Installation

#### Positionierung relativ zur LED

**Optimaler Abstand:** 5-10 mm von LED-Oberfläche

**Ausrichtung:** Perpendikular (senkrecht) zur LED

**Befestigungsmethoden:**
1. **Heißkleber:** Einfach, schnell, repositionierbar
2. **3D-gedruckter Halter:** Professionell, präzise
3. **Kabelbinder:** Flexibel, schnelle Montage

#### Lichtabschirmung (kritisch!)

**Problem:** Umgebungslicht (Schrankbeleuchtung) kann Messung verfälschen

**Lösung:** Lichttunnel erstellen

```
┌─────────────┐
│   LED       │  ← Relay-LED
└──────┬──────┘
       │  5-10mm
┌──────▼──────┐
│ ████████████ │  ← Schwarzer Schrumpfschlauch
│  Sensor     │      (10-15mm Länge)
│  ALS-PT19   │
└─────────────┘
```

**Material:** Schwarzer Schrumpfschlauch (10-15mm)

**Anwendung:**
1. Schlauch über Sensor schieben
2. Mit Heißluft schrumpfen
3. Öffnung zur LED ausrichten
4. Zusätzlich schwarzes Klebeband bei Bedarf

### 6.5 Kalibrierung

Die Kalibrierung ist **essentiell** für zuverlässige LED-Erkennung.

#### Schritt 1: Rohwerte erfassen

**Debug-Modus aktivieren:**
```cpp
// In config.h:
#define DEBUG_ENABLED 1
```

**Serial Monitor öffnen (115200 Baud) und Werte ablesen:**

| LED-Zustand | Rohwert-Bereich | Beispiel |
|-------------|-----------------|----------|
| LED AUS | 0 - 100 | 20-50 |
| LED AN | 300 - 900 | 400-700 |
| Umgebungslicht | 50 - 200 | 100-150 |

#### Schritt 2: Schwellwerte berechnen

**Empfohlene Formel:**

```
THRESHOLD_ON  = (LED_AN_min + LED_AN_avg) / 2
THRESHOLD_OFF = (LED_AUS_max + Umgebung_avg) / 2
```

**Beispiel:**
- LED_AUS: 20-50 (avg: 35)
- LED_AN: 400-700 (avg: 550)
- Umgebung: 100-150 (avg: 125)

```
THRESHOLD_ON  = (400 + 550) / 2 = 475  → 300 (mit Sicherheitsmarge)
THRESHOLD_OFF = (50 + 125) / 2 = 87    → 100 (mit Sicherheitsmarge)
```

#### Schritt 3: config.h anpassen

```cpp
// In RelayNode/config.h:
#define THRESHOLD_ON  300   // Anpassen basierend auf Kalibrierung
#define THRESHOLD_OFF 100   // Anpassen basierend auf Kalibrierung
```

#### Schritt 4: Blink-Erkennung tunen

**Bei unzuverlässiger Blink-Erkennung:**

1. **Blink-Frequenz messen:**
   - Zähle Blinks pro Sekunde (z.B. 2 Hz)
   - Berechne Periode: T = 1000ms / 2 = 500ms

2. **Parameter anpassen:**
```cpp
#define BLINK_MIN_INTERVAL 50    // ~T/10
#define BLINK_MAX_INTERVAL 1000  // ~2*T
#define BLINK_MIN_COUNT 2        // Erhöhen bei False Positives
```

### 6.6 Software-Integration

#### Analogwert lesen

```cpp
int lightValue = analogRead(A0);  // 0-1023 (10-bit ADC)
```

#### Moving Average Filter (implementiert in led_monitor.cpp)

```cpp
// Beispiel: 5-Sample Moving Average
int samples[5] = {0};
int sampleIndex = 0;

void addSample(int newValue) {
  samples[sampleIndex] = newValue;
  sampleIndex = (sampleIndex + 1) % 5;
}

int getFilteredValue() {
  int sum = 0;
  for (int i = 0; i < 5; i++) {
    sum += samples[i];
  }
  return sum / 5;
}
```

#### Hysterese-Schwellwerte

```cpp
if (value > THRESHOLD_ON) {
  state = LED_STATE_ACTIVE;
} else if (value < THRESHOLD_OFF) {
  state = LED_STATE_IDLE;
} else {
  // Hysterese-Zone: Behalte vorherigen Zustand
  state = previousState;
}
```

### 6.7 Fehlerdiagnose

#### Problem: Sensor liest immer 0

**Ursachen:**
- Sensor nicht angeschlossen
- VCC fehlt
- Defekter Sensor

**Lösungen:**
1. Multimeter: VCC-Pin auf 5V prüfen
2. Multimeter: OUT-Pin auf 0-5V prüfen (bei Licht ändern)
3. Sensor austauschen

#### Problem: Sensor liest immer 1023

**Ursachen:**
- Kurzschluss VCC→OUT
- Sensor übersättigt (zu hell)

**Lösungen:**
1. Verkabelung prüfen
2. Lichtabschirmung verbessern
3. ND-Filter vor Sensor (bei sehr hellen LEDs)

#### Problem: Schwankende Werte / False Triggers

**Ursachen:**
- Umgebungslicht
- EMI (Elektromagnetische Störungen)
- Lose Verbindungen

**Lösungen:**
1. Lichtabschirmung verbessern
2. `SMOOTHING_SAMPLES` erhöhen (z.B. auf 10)
3. Hysterese-Zone vergrößern
4. Sensorkabel von Stromleitungen fernhalten
5. 0.1µF Kondensator zwischen OUT und GND löten

#### Problem: Blink-Erkennung funktioniert nicht

**Ursachen:**
- Blink-Frequenz außerhalb detektierbarem Bereich
- Zu wenige Samples pro Blink
- Parameter falsch eingestellt

**Lösungen:**
1. Blink-Frequenz im Serial Monitor beobachten
2. `CHECK_INTERVAL` verkleinern (z.B. 50ms statt 100ms)
3. `BLINK_MIN_INTERVAL` und `BLINK_MAX_INTERVAL` anpassen
4. `BLINK_MIN_COUNT` erhöhen (weniger False Positives)

### 6.8 Wartung

#### Wöchentlich
- Visuelle Inspektion: Sensor-Position korrekt?
- Funktion prüfen: LED-Zustandsänderungen erkannt?

#### Monatlich
- Sensor-Linse reinigen (weiches, trockenes Tuch)
- Lichtabschirmung auf Risse prüfen
- Verkabelung auf festen Sitz prüfen

#### Jährlich
- Vollständige Rekalibrierung
- Sensor-Alterung prüfen (Werte vergleichen)
- Bei Drift >20%: Sensor austauschen

### 6.9 Beispiel-Messwerte

#### Typische Schrack-Relay LEDs

| LED-Farbe | AUS | AN | Empfohlene Schwellwerte |
|-----------|-----|-----|------------------------|
| Rot | 20-60 | 350-550 | ON=300, OFF=100 |
| Grün | 25-70 | 400-650 | ON=350, OFF=120 |
| Gelb | 30-80 | 380-600 | ON=320, OFF=130 |
| Blau | 15-50 | 450-700 | ON=380, OFF=90 |

**Hinweis:** Werte können je nach LED-Hersteller, Alter und Umgebung variieren. Immer individuell kalibrieren!

---

## 7. Sensor-Wartungsplan (aktualisiert)

### Erweiterte Wartungstabelle

| Sensor | Täglich | Wöchentlich | Monatlich | Jährlich |
|--------|---------|-------------|-----------|----------|
| JSN-SR04T | - | Visuell prüfen | Reinigen | Austauschen bei Drift |
| DS18B20 | - | - | Prüfen | Kalibrierung |
| TSW-20M | - | Sichtfenster prüfen | Reinigen | Austauschen |
| CQRSENTDS01 | - | - | Elektroden reinigen | Kalibrierung |
| **ALS-PT19** | **-** | **Position prüfen** | **Linse reinigen** | **Rekalibrierung** |

---

## 8. Zusammenfassung (aktualisiert)

Das BioSync-System nutzt jetzt **9 Sensoren** über **2 Nodes**:

### SensorNode (4 Sensoren)
1. **JSN-SR04T** – Füllstand (Ultraschall)
2. **DS18B20** – Temperatur (1-Wire)
3. **TSW-20M** – Trübung (Analog)
4. **CQRSENTDS01** – TDS (Analog)

### RelayNode (4 Sensoren) - NEU
5. **ALS-PT19 #1** – PUMP LED Status
6. **ALS-PT19 #2** – VENT LED Status
7. **ALS-PT19 #3** – LED3 Status
8. **ALS-PT19 #4** – LED4 Status

**Zusätzliche Features:**
- Event-basierte Kommunikation (nur bei Zustandsänderung)
- 3 LED-Zustände: IDLE, ACTIVE, ERROR (blinkend)
- Moving Average Filter für Störunterdrückung
- Hysterese-Schwellwerte für Stabilität

---

**Dokumentversion:** 2.1  
**Letzte Aktualisierung:** 2025-11-03  
**Autor:** BioSync Project Team

