# BioSync - Erweiterte Fehlerbehebung
## Systematische Fehlerdiagnose und Lösungen

**Version:** 2.0  
**Datum:** November 2025

---

## Inhaltsverzeichnis

1. [Fehlerdiagnose-Übersicht](#1-fehlerdiagnose-übersicht)
2. [Systematische Fehlersuche](#2-systematische-fehlersuche)
3. [Hardware-Probleme](#3-hardware-probleme)
4. [Software-Probleme](#4-software-probleme)
5. [Kommunikations-Probleme](#5-kommunikations-probleme)
6. [Sensor-Probleme](#6-sensor-probleme)
7. [Display-Probleme](#7-display-probleme)
8. [Entscheidungsbäume](#8-entscheidungsbäume)

---

## 1. Fehlerdiagnose-Übersicht

### 1.1 Symptom-Matrix

| Symptom | Wahrscheinliche Ursache | Priorität | Seite |
|---------|-------------------------|-----------|-------|
| Sensor Node startet nicht | Stromversorgung | HOCH | [3.1](#31-sensor-node-startet-nicht) |
| Display Node startet nicht | Stromversorgung | HOCH | [3.2](#32-display-node-startet-nicht) |
| Display schwarz | Nextion/Stromversorgung | HOCH | [7.1](#71-display-bleibt-schwarz) |
| Keine Daten auf Display | RS-485 Kommunikation | HOCH | [5.1](#51-keine-rs-485-kommunikation) |
| Falsche Sensorwerte | Sensor-Hardware/Kalibrierung | MITTEL | [6](#6-sensor-probleme) |
| System friert ein | Software/Stromversorgung | MITTEL | [4.1](#41-system-friert-ein) |
| Sporadische Fehler | EMI/Kabelprobleme | NIEDRIG | [5.3](#53-intermittierende-fehler) |

### 1.2 Diagnose-Werkzeuge

**Hardware:**
- Multimeter (Spannungsmessung, Durchgangsprüfung)
- USB-Kabel für Serial Monitor
- Ersatz-Komponenten zum Testen

**Software:**
- Arduino IDE Serial Monitor (115200 baud)
- Nextion Editor (für HMI-Debugging)

---

## 2. Systematische Fehlersuche

### 2.1 5-Schritt Diagnose-Prozess

```
┌─────────────────────────────────────────────────────────────┐
│             BIOSYNC FEHLERDIAGNOSE-PROZESS                  │
└─────────────────────────────────────────────────────────────┘

Schritt 1: STROMVERSORGUNG PRÜFEN
  ┌──────────────────────────────────────┐
  │ □ 12V Netzteil Ausgangsspannung OK?  │
  │ □ LM2596 Ausgangsspannung = 5.0V?    │
  │ □ Alle Komponenten Power-LED an?     │
  └──────────────────────────────────────┘
           │
           ▼
Schritt 2: VERKABELUNG PRÜFEN
  ┌──────────────────────────────────────┐
  │ □ Alle Verbindungen fest?            │
  │ □ Polarität korrekt (+ / -)?         │
  │ □ CAT7-Kabel durchgängig?            │
  │ □ Keine Kurzschlüsse?                │
  └──────────────────────────────────────┘
           │
           ▼
Schritt 3: KOMMUNIKATION PRÜFEN
  ┌──────────────────────────────────────┐
  │ □ Sensor Node sendet? (Serial Mon.)  │
  │ □ Display Node empfängt? (Ser. Mon.) │
  │ □ RS-485 A/B richtig verbunden?      │
  │ □ Nachrichtenformat korrekt?         │
  └──────────────────────────────────────┘
           │
           ▼
Schritt 4: SENSOREN EINZELN TESTEN
  ┌──────────────────────────────────────┐
  │ □ JSN-SR04T: Wert plausibel?         │
  │ □ DS18B20: Temperatur realistisch?   │
  │ □ TSW-20M: ADC 0-1023?               │
  │ □ CQRSENTDS01: ADC 0-1023?           │
  └──────────────────────────────────────┘
           │
           ▼
Schritt 5: DISPLAY AKTUALISIERUNG
  ┌──────────────────────────────────────┐
  │ □ Nextion reagiert auf Befehle?      │
  │ □ Textfelder werden aktualisiert?    │
  │ □ Touch funktioniert?                │
  │ □ Screensaver aktiv?                 │
  └──────────────────────────────────────┘
```

### 2.2 Messstellen-Tabelle

| Messpunkt | Soll-Wert | Gemessen | Status | Notizen |
|-----------|-----------|----------|--------|---------|
| 12V PSU Output | 12.0V ±0.5V | _____ V | ☐ OK ☐ NOK | |
| LM2596 (Display) Output | 5.0V ±0.05V | _____ V | ☐ OK ☐ NOK | |
| LM2596 (Sensor) Output | 5.0V ±0.05V | _____ V | ☐ OK ☐ NOK | |
| Nano Every VIN | 5.0V ±0.1V | _____ V | ☐ OK ☐ NOK | |
| Mega 2560 5V | 5.0V ±0.1V | _____ V | ☐ OK ☐ NOK | |
| Nextion VCC | 5.0V ±0.1V | _____ V | ☐ OK ☐ NOK | |
| CAT7 12V (am Sensor) | >11.5V | _____ V | ☐ OK ☐ NOK | Spannungsabfall |
| RS-485 A-B (Idle) | ~0V | _____ V | ☐ OK ☐ NOK | Differenz |

---

## 3. Hardware-Probleme

### 3.1 Sensor Node startet nicht

**Symptom:** Arduino Nano Power-LED leuchtet nicht

**Diagnose-Ablauf:**

```
1. Multimeter: LM2596 Output messen
   ├─ 5.0V OK → Weiter zu Schritt 2
   └─ Nicht 5.0V → LM2596 Problem

2. Multimeter: LM2596 Input messen
   ├─ 12V OK → LM2596 defekt oder falsch eingestellt
   │            → LM2596 neu einstellen oder tauschen
   └─ Nicht 12V → Kabel/Netzteil Problem

3. CAT7-Kabel durchmessen
   ├─ Adern 1+2 (12V+): Durchgang OK?
   ├─ Adern 3+4 (GND): Durchgang OK?
   └─ Kurzschluss zwischen Adern? → Kabel tauschen

4. Netzteil prüfen
   ├─ 12V am Ausgang? → OK
   └─ <12V oder 0V → Netzteil defekt
```

**Lösungen:**

| Ursache | Lösung |
|---------|--------|
| LM2596 nicht eingestellt | Mit Multimeter auf exakt 5.00V einstellen |
| LM2596 defekt | Modul tauschen (typische Lebensdauer: >10 Jahre) |
| CAT7-Kabel beschädigt | Kabel testen, ggf. tauschen |
| Netzteil defekt | 12V/2A Netzteil tauschen |
| Kurzschluss im System | Alle Verbindungen prüfen, Kurzschluss lokalisieren |

### 3.2 Display Node startet nicht

**Symptom:** Arduino Mega und/oder Nextion Display aus

**Diagnose analog zu 3.1, zusätzlich:**

```
Nextion Display getrennt testen:
  1. Nextion VCC/GND direkt an LM2596 OUT+ / OUT-
  2. Nextion sollte Bootscreen zeigen
  3. Wenn nicht: Nextion defekt oder Kabel gebrochen
```

**Spezielle Checks:**

- **Stromverbrauch:** Mega + Nextion = ~136mA @ 5V
  - LM2596 sollte min. 200mA liefern können
  - Wenn LM2596 überlastet: Spannung bricht ein (<4.5V)
  - Lösung: LM2596 mit höherer Stromkapazität (3A statt 1A)

### 3.3 Überhitzung

**Symptom:** LM2596 oder Komponenten sehr heiß (>60°C)

**Ursachen:**

| Ursache | Lösung |
|---------|--------|
| LM2596 überlastet | Stromverbrauch messen, ggf. Komponenten trennen |
| Schlechte Belüftung | Gehäuse mit Lüftungsschlitzen versehen |
| Kurzschluss | Mit Multimeter Kurzschlüsse finden und beheben |
| Falscher LM2596 (1A statt 3A) | Modul mit höherer Stromkapazität tauschen |

**Normal-Temperaturen:**

| Komponente | Normal | Kritisch |
|------------|--------|----------|
| LM2596 | 30-45°C | >70°C |
| Arduino | 25-35°C | >60°C |
| Netzteil | 30-50°C | >75°C |

---

## 4. Software-Probleme

### 4.1 System friert ein (Keine Updates mehr)

**Symptom:** Display zeigt alte Werte, keine Aktualisierung

**Diagnose:**

```
1. Serial Monitor öffnen (Sensor Node, 115200 baud)
   ├─ Nachrichten werden gesendet? → Sensor Node OK
   │    └─ Weiter zu Display Node prüfen
   └─ Keine Nachrichten? → Sensor Node Problem

2. Sensor Node Problem:
   ├─ Code hängt in Schleife (watchdog nicht implementiert)
   ├─ Sensor blockiert (z.B. DS18B20 conversion)
   └─ Lösung: Arduino Reset-Button drücken

3. Display Node Problem:
   ├─ Serial Monitor öffnen (Display Node, 115200 baud)
   ├─ Werden Nachrichten empfangen?
   │    ├─ Ja, aber Display nicht aktualisiert → Nextion Problem
   │    └─ Nein → RS-485 Problem
   └─ Lösung: siehe entsprechende Abschnitte
```

**Präventive Maßnahmen:**

```cpp
// Watchdog Timer implementieren (zukünftige Verbesserung):
#include <avr/wdt.h>

void setup() {
  wdt_enable(WDTO_8S);  // 8 Sekunden Watchdog
  // ... rest of setup ...
}

void loop() {
  wdt_reset();  // Watchdog zurücksetzen (muss alle <8s erfolgen)
  // ... rest of loop ...
}
```

### 4.2 Firmware-Upload Fehler

**Symptom:** Arduino IDE meldet Upload-Fehler

**Lösungen:**

| Fehler | Lösung |
|--------|--------|
| "Port not found" | USB-Kabel prüfen, Treiber installieren |
| "Programmer not responding" | Reset-Button während Upload drücken |
| "Out of memory" | Falsches Board gewählt (Nano Every vs. Nano) |
| "Verification error" | Kabel tauschen, anderer USB-Port |

---

## 5. Kommunikations-Probleme

### 5.1 Keine RS-485 Kommunikation

**Symptom:** Display zeigt "Sensor Offline" oder keine Daten

**Systematische Diagnose:**

```
┌──────────────────────────────────────────────────────────┐
│ RS-485 TROUBLESHOOTING DECISION TREE                     │
└──────────────────────────────────────────────────────────┘

Sensor Node sendet?
├─ JA (Serial Monitor zeigt "Gesendet: <SENSOR;...")
│   │
│   └─ Display Node empfängt?
│       ├─ JA (Serial Monitor zeigt "Empfangen: <SENSOR;...")
│       │   └─ Problem: Display Update oder Parsing
│       │       → Siehe Abschnitt 7
│       │
│       └─ NEIN
│           └─ RS-485 Verkabelung oder Modul defekt
│               ├─ A mit A verbunden? B mit B?
│               ├─ GND gemeinsam?
│               ├─ MAX485 defekt? (tauschen)
│               └─ CAT7 Adern 5+6 durchgängig?
│
└─ NEIN
    └─ Problem im Sensor Node
        ├─ Code läuft? (Power LED blinkt?)
        ├─ Timer funktioniert? (millis() overflow?)
        └─ RS-485 DE-Pin korrekt gesteuert?
```

**Hardware-Checks:**

```
Multimeter-Tests:

1. CAT7 Adern 5+6 (RS-485):
   - Sensor Node Ende: A-Pin messen → notieren: X Ω zu GND
   - Display Node Ende: A-Pin messen → sollte gleich sein
   - Wenn unterschiedlich: Kabelbruch

2. RS-485 Modul Spannung:
   - VCC-Pin: 5.0V ±0.1V
   - GND: 0V
   - Wenn nicht: Modul hat keinen Strom

3. Loopback-Test (Display Node):
   - RS-485 Modul: A und B kurzschließen (mit Jumper)
   - Serial Monitor: Gesendete = Empfangene Bytes
   - Wenn OK: Modul funktioniert, Problem ist Kabel oder Sensor
```

### 5.2 Datenmüll / Garbled Data

**Symptom:** Display zeigt Unsinn, Serial Monitor zeigt unleserliche Zeichen

**Ursachen und Lösungen:**

| Ursache | Symptom | Lösung |
|---------|---------|--------|
| Falsche Baudrate | Zufällige Zeichen | Beide Seiten auf 9600 prüfen |
| A/B vertauscht | Invertierte Daten | A mit A, B mit B verbinden |
| Kabelschaden | Sporadisch unleserlich | CAT7 tauschen |
| EMI (Störungen) | Bei Pumpe EIN/AUS | Kabel schirmen, erden |
| Ground Loop | Rauschen | Schirm nur einseitig erden |

**Baudrate verifizieren:**

```cpp
// SensorNode/config.h:
static const unsigned long RS485_BAUD = 9600UL;  // ← Prüfen!

// DisplayNode/rs485.cpp:
RS485_SERIAL.begin(9600);  // ← Muss identisch sein!
```

### 5.3 Intermittierende Fehler

**Symptom:** System funktioniert manchmal, manchmal nicht

**Typische Ursachen:**

1. **Wackelkontakt:**
   - Lösung: Alle Steckverbindungen auf festen Sitz prüfen
   - Langfristig: Löten statt Stecken

2. **Temperatur-abhängig:**
   - Komponente erwärmt sich → Fehler tritt auf
   - Lösung: Komponente identifizieren (Infrarot-Thermometer), kühlen oder tauschen

3. **Zeitgesteuert (millis() overflow):**
   - Nach ~49 Tagen (2^32 ms) kann millis() überlaufen
   - Lösung: Code auf overflow-sichere Vergleiche prüfen
   ```cpp
   // FALSCH:
   if (millis() > lastSend + 5000) { ... }
   
   // RICHTIG:
   if (millis() - lastSend >= 5000) { ... }
   ```

---

## 6. Sensor-Probleme

### 6.1 JSN-SR04T liefert -1.0 (Timeout)

**Ursachen:**

| Ursache | Test | Lösung |
|---------|------|--------|
| Zu große Distanz (>450cm) | Mit Maßband prüfen | Sensor tiefer montieren |
| Echo zu schwach (Schaum) | Visuell prüfen | Warten auf klare Oberfläche |
| ECHO-Pin nicht verbunden | Durchgangsprüfung | Kabel reparieren |
| Sensor defekt | Mit anderem Objekt testen | Sensor tauschen |

**Test-Setup:**

```
Einfacher Test:
  1. USB Serial Monitor öffnen
  2. Hand vor Sensor halten (30-50 cm Abstand)
  3. Wert sollte zwischen 30-50 cm sein
  4. Wenn Timeout: Sensor oder Verkabelung defekt
```

### 6.2 DS18B20 zeigt -127.0°C

**Ursache:** Sensor nicht erkannt (DEVICE_DISCONNECTED_C)

**Troubleshooting:**

```
1. Pull-up Widerstand vorhanden?
   ├─ 4.7kΩ zwischen DATA und 5V?
   │   └─ NEIN → Pull-up hinzufügen (KRITISCH!)
   └─ JA → Weiter

2. Verkabelung korrekt?
   ├─ VCC an 5V?
   ├─ DATA an D4?
   └─ GND an GND?

3. Sensor defekt?
   ├─ Mit Multimeter: Widerstand GND-VCC
   │   └─ Sollte >1 MΩ sein (nicht Kurzschluss)
   └─ Sensor tauschen
```

**OneWire Debug:**

```cpp
// Temporär in setup() einfügen:
void setup() {
  Serial.begin(115200);
  dsSensor.begin();
  
  Serial.print("Sensors found: ");
  Serial.println(dsSensor.getDeviceCount());
  // Sollte "1" ausgeben, wenn Sensor erkannt
}
```

### 6.3 Analog-Sensoren (TSW-20M, CQRSENTDS01) konstant 0 oder 1023

**Ursachen:**

| Symptom | Ursache | Lösung |
|---------|---------|--------|
| Konstant 0 | Pin nicht verbunden | Verkabelung prüfen |
| Konstant 0 | Sensor-VCC fehlt | VCC prüfen (5V?) |
| Konstant 1023 | Pin offen (Pull-up) | Sensor anschließen |
| Konstant 1023 | Sensor defekt | Sensor tauschen |

---

## 7. Display-Probleme

### 7.1 Display bleibt schwarz

**Ursachen:**

1. **Keine Stromversorgung:**
   - VCC: 5.0V vorhanden?
   - GND angeschlossen?

2. **Nextion nicht initialisiert:**
   - .tft-Datei nicht auf Display geladen
   - Lösung: Nextion Editor → Compile → Upload

3. **Backlight aus:**
   - Nextion-Befehl: `dim=0` wurde gesendet
   - Lösung: `dim=100` senden (volle Helligkeit)

### 7.2 Display zeigt alte Werte

**Ursachen:**

- Kommunikation unterbrochen (siehe 5.1)
- Parsing-Fehler (Nachrichtenformat geändert?)
- Nextion HMI: Falsche Komponenten-Namen

**Debug:**

```cpp
// In DisplayNode loop(), temporär:
Serial.print("Parsed: ");
Serial.print("DIST="); Serial.print(d.distance);
Serial.print(" TMP="); Serial.print(d.temperature);
// ... etc.
```

### 7.3 Touch funktioniert nicht

**Ursachen:**

- Resistive Touch-Kalibrierung notwendig
- Nextion HMI: Events nicht programmiert
- Nextion defekt (selten)

**Lösung:**

```
Nextion Editor:
  1. Tools → Touch Calibration
  2. 5 Punkte auf Display antippen
  3. Speichern und neu hochladen
```

---

## 8. Entscheidungsbäume

### 8.1 Hauptdiagnose-Baum

```
┌──────────────────────────────────────────────────────────┐
│         BIOSYNC MASTER TROUBLESHOOTING TREE              │
└──────────────────────────────────────────────────────────┘

System eingeschaltet?
├─ NEIN → Netzteil einschalten, Sicherungen prüfen
│
└─ JA
    │
    Power LEDs leuchten?
    ├─ NEIN → HARDWARE PROBLEM
    │   └─ Siehe Abschnitt 3 (Hardware-Probleme)
    │
    └─ JA
        │
        Daten auf Display?
        ├─ NEIN
        │   │
        │   Sensor Node sendet? (Serial Monitor)
        │   ├─ NEIN → SENSOR NODE PROBLEM
        │   │   └─ Code, Sensoren, Timer
        │   │
        │   └─ JA → KOMMUNIKATION oder DISPLAY PROBLEM
        │       ├─ Display Node empfängt? (Serial Monitor)
        │       │   ├─ NEIN → RS-485 PROBLEM (Abschnitt 5)
        │       │   └─ JA → DISPLAY UPDATE PROBLEM (Abschnitt 7)
        │
        └─ JA
            │
            Werte plausibel?
            ├─ NEIN → SENSOR PROBLEM (Abschnitt 6)
            │   └─ Kalibrierung, Sensor-Hardware
            │
            └─ JA → SYSTEM OK ✓
```

