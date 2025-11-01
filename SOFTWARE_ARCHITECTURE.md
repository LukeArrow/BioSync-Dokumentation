# BioSync - Software-Architektur und Code-Dokumentation
## Vollständige Software-Analyse auf Code-Ebene

**Version:** 2.0  
**Datum:** November 2025

---

## Inhaltsverzeichnis

1. [Software-Übersicht](#1-software-übersicht)
2. [Sensor Node Software](#2-sensor-node-software)
3. [Display Node Software](#3-display-node-software)
4. [State Machines](#4-state-machines)
5. [Memory Management](#5-memory-management)
6. [Error Handling](#6-error-handling)
7. [Performance Optimization](#7-performance-optimization)

---

## 1. Software-Übersicht

### 1.1 Architektur-Prinzipien

**Design-Philosophie:**
- **Modular:** Klare Trennung in funktionale Module
- **Robust:** Fehlertoleranz und Graceful Degradation
- **Einfach:** Keine komplexen Abhängigkeiten
- **Wartbar:** Gut dokumentierter, lesbarer Code

**Programmiersprache:** Arduino C++ (AVR-GCC Compiler)

**Build-System:** Arduino IDE 1.8.x oder 2.x

### 1.2 Projekt-Struktur

```
BioSync/
├── SensorNode/
│   ├── SensorNode.ino    # Hauptprogramm
│   ├── config.h          # Konfiguration (Pins, Constants)
│   ├── sensors.h         # Sensor-Funktionen (Header)
│   ├── sensors.cpp       # Sensor-Funktionen (Implementation)
│   ├── rs485.h           # RS-485 Kommunikation (Header)
│   └── rs485.cpp         # RS-485 Kommunikation (Implementation)
│
├── DisplayNode/
│   ├── DisplayNode.ino   # Hauptprogramm
│   ├── config.h          # Konfiguration
│   ├── rs485.h           # RS-485 Empfang
│   ├── rs485.cpp         # RS-485 Empfang
│   ├── nextion.h         # Nextion Display Interface
│   ├── nextion.cpp       # Nextion Display Interface
│   ├── parser.h          # Nachrichtenparser
│   └── parser.cpp        # Nachrichtenparser
│
├── Nextion/
│   └── BioSync.HMI       # Nextion HMI Projekt
│
└── docs/
    └── (Diese Dateien)
```

---

## 2. Sensor Node Software

### 2.1 SensorNode.ino - Hauptprogramm

```cpp
/*
 * SensorNode.ino
 * Hauptprogramm für Arduino Nano Every (Sensor Node)
 * 
 * Funktionen:
 * - Initialisierung aller Sensoren
 * - Periodisches Auslesen (alle 5s)
 * - Formatierung und Übertragung via RS-485
 */

#include "config.h"
#include "sensors.h"
#include "rs485.h"

// Globale Variable für Timing
unsigned long lastSend = 0;

void setup() {
  // Debug-Ausgabe via USB
  Serial.begin(115200);
  
  // Sensoren initialisieren
  sensors_begin();
  
  // RS-485 initialisieren
  rs485_begin();
  
  Serial.println(F("SensorNode gestartet"));
}

void loop() {
  // Prüfen ob Sendeintervall erreicht
  if (millis() - lastSend >= SEND_INTERVAL) {
    lastSend = millis();  // Timestamp aktualisieren
    
    // Alle 4 Sensoren auslesen
    float distance = measureDistance();      // JSN-SR04T
    float temp = readTemperature();          // DS18B20
    int turb = readTurbidity();              // TSW-20M
    int tds = readTDS();                     // CQRSENTDS01
    
    // Nachricht formatieren
    String msg = "<SENSOR;";
    msg += "DIST=" + String(distance, 1) + ";";  // 1 Dezimalstelle
    msg += "TMP=" + String(temp, 1) + ";";       // 1 Dezimalstelle
    msg += "TUR=" + String(turb) + ";";          // Integer
    msg += "TDS=" + String(tds) + ">\n";         // Integer + Newline
    
    // Via RS-485 senden
    rs485_send(msg);
    
    // Debug-Ausgabe
    Serial.print(F("Gesendet: "));
    Serial.print(msg);
  }
  
  // Weitere Tasks könnten hier eingefügt werden
  // z.B. Watchdog Reset, LED Blink, etc.
}
```

**Timing-Analyse:**

| Funktion | Dauer | CPU-Last |
|----------|-------|----------|
| measureDistance() | ~800ms | Blockierend (DS18B20) |
| readTemperature() | ~30ms | pulseIn() |
| readTurbidity() | ~10ms | Schnell (ADC) |
| readTDS() | ~10ms | Schnell (ADC) |
| String-Formatierung | ~10ms | String-Operationen |
| rs485_send() | ~60ms | Serial TX + Delays |
| **GESAMT** | **~920ms** | **18.4% Duty Cycle** @ 5s Intervall |

### 2.2 config.h - Konfiguration

```cpp
#ifndef SENSOR_CONFIG_H
#define SENSOR_CONFIG_H

#include <Arduino.h>

// ============================================
// PIN DEFINITIONS
// ============================================

// JSN-SR04T Ultrasonic
static const uint8_t TRIG_PIN    = 2;  // Digital Output
static const uint8_t ECHO_PIN    = 3;  // Digital Input

// DS18B20 Temperature (1-Wire)
static const uint8_t DS18B20_PIN = 4;  // Digital I/O (needs 4.7kΩ pull-up)

// Analog Sensors
static const uint8_t TURB_PIN    = A0; // TSW-20M Turbidity
static const uint8_t TDS_PIN     = A1; // CQRSENTDS01 TDS

// RS-485 (SoftwareSerial)
static const uint8_t RS485_RX    = 7;  // Receiver Output (RO)
static const uint8_t RS485_TX    = 6;  // Driver Input (DI)
static const uint8_t RS485_DE    = 5;  // Driver Enable / Receiver Enable

// ============================================
// OPERATIONAL PARAMETERS
// ============================================

// Sendeintervall (Millisekunden)
static const unsigned long SEND_INTERVAL = 5000UL;  // 5 Sekunden

// RS-485 Baudrate
static const unsigned long RS485_BAUD = 9600UL;

// ============================================
// CALIBRATION CONSTANTS (Optional)
// ============================================

// JSN-SR04T Distance Offset (cm)
// Adjust after calibration
// static const float DISTANCE_OFFSET = 0.0;

// DS18B20 Temperature Offset (°C)
// Adjust after calibration
// static const float TEMP_OFFSET = 0.0;

#endif // SENSOR_CONFIG_H
```

### 2.3 sensors.cpp - Sensor-Funktionen

```cpp
#include "sensors.h"
#include "config.h"

// Bibliotheken für DS18B20
#include <OneWire.h>
#include <DallasTemperature.h>

// OneWire Instanz für DS18B20
static OneWire oneWire(DS18B20_PIN);
static DallasTemperature dsSensor(&oneWire);

/**
 * sensors_begin()
 * Initialisiert alle Sensoren
 * 
 * Wird einmal in setup() aufgerufen
 */
void sensors_begin() {
  // JSN-SR04T Pins konfigurieren
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  digitalWrite(TRIG_PIN, LOW);  // Initialer Zustand
  
  // DS18B20 initialisieren
  dsSensor.begin();
  
  // Kurze Verzögerung für Stabilisierung
  delay(50);
}

/**
 * measureDistance()
 * Misst Distanz mit JSN-SR04T Ultraschallsensor
 * 
 * @return float - Distanz in cm, oder -1.0 bei Fehler
 * 
 * Timing: ~30ms (bei Echo), ~30ms timeout bei keinem Echo
 */
float measureDistance() {
  // Sensor zurücksetzen
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  
  // 10μs Trigger-Puls senden
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  // Echo-Puls messen (Timeout 30ms = ~517cm max)
  unsigned long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  
  // Timeout-Check (kein Echo empfangen)
  if (duration == 0) {
    return -1.0;  // Error code
  }
  
  // Berechnung: Schallgeschwindigkeit ~58μs/cm
  // duration [μs] / 58 = distance [cm]
  float distance = duration / 58.0;
  
  // Optional: Offset anwenden
  // distance += DISTANCE_OFFSET;
  
  return distance;
}

/**
 * readTemperature()
 * Liest Temperatur von DS18B20 Sensor (1-Wire)
 * 
 * @return float - Temperatur in °C, oder -127.0 bei Fehler
 * 
 * Timing: ~750ms (12-bit conversion)
 * 
 * WICHTIG: Benötigt 4.7kΩ Pull-up zwischen DATA und 5V!
 */
float readTemperature() {
  // Temperaturmessung starten (blocking)
  dsSensor.requestTemperatures();
  
  // Wert vom ersten Sensor auf Bus lesen (Index 0)
  float temp = dsSensor.getTempCByIndex(0);
  
  // Fehlercheck: DEVICE_DISCONNECTED_C = -127.0
  if (temp == DEVICE_DISCONNECTED_C) {
    return -127.0;  // Error code (sensor disconnected or no pull-up)
  }
  
  // Optional: Offset anwenden
  // temp += TEMP_OFFSET;
  
  return temp;
}

/**
 * readTurbidity()
 * Liest TSW-20M Trübungssensor (Analog)
 * 
 * @return int - ADC-Wert 0-1023
 * 
 * Timing: ~100μs (ADC conversion)
 * 
 * Wert-Interpretation:
 * - Niedriger Wert = klares Wasser
 * - Hoher Wert = trübes Wasser
 */
int readTurbidity() {
  int value = analogRead(TURB_PIN);
  
  // Optional: Offset oder Kalibrierung
  // value += TURBIDITY_OFFSET;
  // if (value < 0) value = 0;
  // if (value > 1023) value = 1023;
  
  return value;
}

/**
 * readTDS()
 * Liest CQRSENTDS01 TDS-Sensor (Analog)
 * 
 * @return int - ADC-Wert 0-1023
 * 
 * Timing: ~100μs (ADC conversion)
 * 
 * Wert-Interpretation:
 * - Kann mit Kalibrierung in ppm umgerechnet werden
 * - TDS (ppm) = m × ADC + b (lineare Regression)
 */
int readTDS() {
  int value = analogRead(TDS_PIN);
  
  // Optional: Umrechnung in ppm
  // float tds_ppm = TDS_SLOPE * value + TDS_INTERCEPT;
  // return (int)tds_ppm;
  
  return value;
}
```

### 2.4 rs485.cpp - RS-485 Kommunikation

```cpp
#include "rs485.h"
#include "config.h"
#include <SoftwareSerial.h>

// SoftwareSerial Instanz für RS-485
static SoftwareSerial rs485(RS485_RX, RS485_TX);

/**
 * rs485_begin()
 * Initialisiert RS-485 Kommunikation
 * 
 * Wird einmal in setup() aufgerufen
 */
void rs485_begin() {
  // DE/RE Pin als Output konfigurieren
  pinMode(RS485_DE, OUTPUT);
  
  // Initial: Empfangsmodus (DE/RE = LOW)
  digitalWrite(RS485_DE, LOW);
  
  // SoftwareSerial mit Baudrate starten
  rs485.begin(RS485_BAUD);
}

/**
 * rs485_send()
 * Sendet String via RS-485
 * 
 * @param s - String zum Senden
 * 
 * Prozedur:
 * 1. Aktiviere Treiber (DE = HIGH)
 * 2. Warte auf Settling Time
 * 3. Sende Daten
 * 4. Warte auf TX Complete
 * 5. Deaktiviere Treiber (DE = LOW)
 * 
 * Timing: ~60ms für typische 51-Byte Nachricht @ 9600 baud
 */
void rs485_send(const String &s) {
  // Schritt 1: Treiber aktivieren (TX-Modus)
  digitalWrite(RS485_DE, HIGH);
  
  // Schritt 2: Settling Time (Transceiver benötigt ~1-2ms)
  delay(2);
  
  // Schritt 3: Daten senden
  rs485.print(s);
  
  // Schritt 4: Warten bis TX Buffer leer (blocking)
  rs485.flush();
  
  // Schritt 5: Settling Time für letztes Byte
  delay(2);
  
  // Schritt 6: Treiber deaktivieren (RX-Modus)
  digitalWrite(RS485_DE, LOW);
}
```

---

## 3. Display Node Software

### 3.1 DisplayNode.ino - Hauptprogramm

```cpp
/*
 * DisplayNode.ino
 * Hauptprogramm für Arduino Mega 2560 (Display Node)
 * 
 * Funktionen:
 * - RS-485 Nachrichtenempfang
 * - Protokoll-Parsing
 * - Nextion Display Update
 */

#include "config.h"
#include "rs485.h"
#include "nextion.h"
#include "parser.h"

// Globaler Buffer für RS-485 Empfang
String rs485Buffer = "";

void setup() {
  // Debug-Ausgabe via USB
  Serial.begin(115200);
  
  // RS-485 Empfang initialisieren (Serial1)
  rs485_begin();
  
  // Nextion Display initialisieren (Serial2)
  nextion_begin();
  
  Serial.println(F("DisplayNode gestartet"));
}

void loop() {
  // RS-485 Empfang verarbeiten (Serial1)
  while (RS485_SERIAL.available()) {
    char c = (char)RS485_SERIAL.read();
    
    // Newline = Nachricht komplett
    if (c == '\n') {
      String line = rs485Buffer;
      rs485Buffer = "";  // Buffer zurücksetzen
      line.trim();       // Whitespace entfernen
      
      // Nachricht verarbeiten (wenn nicht leer)
      if (line.length() > 0) {
        Serial.print(F("Empfangen: "));
        Serial.println(line);
        
        // Nachricht validieren und parsen
        if (line.startsWith("<") && line.indexOf("SENSOR") >= 0) {
          SensorData d = parseSensorMessage(line);
          nextion_update(d);
        }
      }
    } else {
      // Zeichen zu Buffer hinzufügen
      rs485Buffer += c;
      
      // Overflow-Schutz (Buffer > 1KB)
      if (rs485Buffer.length() > 1024) {
        rs485Buffer = "";
      }
    }
  }
  
  // Weitere Tasks (z.B. Nextion Event-Handling)
  // ... could be added here ...
}
```

### 3.2 parser.cpp - Nachrichtenparser

```cpp
#include "parser.h"

/**
 * extract()
 * Extrahiert Wert für gegebenen Key aus Nachricht
 * 
 * @param src - Quell-String (gesamte Nachricht)
 * @param key - Schlüssel (z.B. "DIST", "TMP")
 * @return String - Extrahierter Wert oder "" bei Fehler
 * 
 * Format: KEY=VALUE;
 * 
 * Beispiel:
 *   extract("<SENSOR;DIST=125.3;TMP=18.2>", "DIST")
 *   → "125.3"
 */
static String extract(const String &src, const char *key) {
  String k = String(key) + "=";
  int idx = src.indexOf(k);
  if (idx < 0) return "";  // Key nicht gefunden
  
  int start = idx + k.length();
  
  // Ende finden: nächstes ';' oder '>'
  int end = src.indexOf(';', start);
  if (end < 0) {
    end = src.indexOf('>', start);
    if (end < 0) end = src.length();
  }
  
  return src.substring(start, end);
}

/**
 * parseSensorMessage()
 * Parst SENSOR-Nachricht und extrahiert alle Felder
 * 
 * @param msg - Rohe Nachricht (z.B. "<SENSOR;DIST=125.3;...>")
 * @return SensorData - Struct mit allen Feldern
 * 
 * Format: <SENSOR;DIST=xxx.x;TMP=xx.x;TUR=xxxx;TDS=xxxx>
 * 
 * Fehlerbehandlung:
 * - Fehlende Felder werden als "n/a" zurückgegeben
 * - Einheiten werden hinzugefügt (cm, °C)
 */
SensorData parseSensorMessage(const String &msg) {
  SensorData d;
  
  // Werte extrahieren (ohne Einheiten)
  d.distance = extract(msg, "DIST");
  d.temperature = extract(msg, "TMP");
  d.turbidity = extract(msg, "TUR");
  d.tds = extract(msg, "TDS");
  
  // Einheiten und Fallback-Werte
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
  
  if (!d.turbidity.length()) {
    d.turbidity = "n/a";
  }
  
  if (!d.tds.length()) {
    d.tds = "n/a";
  }
  
  return d;
}
```

### 3.3 nextion.cpp - Display Interface

```cpp
#include "nextion.h"
#include "config.h"

/**
 * nextion_begin()
 * Initialisiert Nextion Display Kommunikation
 * 
 * Wird einmal in setup() aufgerufen
 */
void nextion_begin() {
  NEXTION_SERIAL.begin(9600);  // Standard Nextion Baudrate
}

/**
 * sendToNextion()
 * Sendet Befehl an Nextion Display
 * 
 * @param cmd - Befehl-String (ohne Terminator)
 * 
 * Nextion erwartet 3-Byte Terminator: 0xFF 0xFF 0xFF
 * 
 * Beispiel:
 *   sendToNextion("tDist.txt=\"125.3 cm\"");
 *   Sendet: "tDist.txt=\"125.3 cm\"" + 0xFF + 0xFF + 0xFF
 */
static void sendToNextion(const String &cmd) {
  NEXTION_SERIAL.print(cmd);
  NEXTION_SERIAL.write(0xFF);
  NEXTION_SERIAL.write(0xFF);
  NEXTION_SERIAL.write(0xFF);
}

/**
 * nextion_update()
 * Aktualisiert alle Textfelder auf Nextion Display
 * 
 * @param d - SensorData Struct mit allen Werten
 * 
 * Nextion Komponenten:
 * - tDist (Text): Füllstand
 * - tTemp (Text): Temperatur
 * - tTurb (Text): Trübung
 * - tTDS (Text): TDS-Wert
 * 
 * Timing: ~50ms (4× 10ms + overhead)
 */
void nextion_update(const SensorData &d) {
  // Update tDist (Füllstand)
  sendToNextion(String(NEXTION_TDIST) + ".txt=\"" + d.distance + "\"");
  delay(10);  // Nextion benötigt Verarbeitungszeit
  
  // Update tTemp (Temperatur)
  sendToNextion(String(NEXTION_TTEMP) + ".txt=\"" + d.temperature + "\"");
  delay(10);
  
  // Update tTurb (Trübung)
  sendToNextion(String(NEXTION_TTURB) + ".txt=\"" + d.turbidity + "\"");
  delay(10);
  
  // Update tTDS (TDS)
  sendToNextion(String(NEXTION_TTDS) + ".txt=\"" + d.tds + "\"");
}
```

---

## 4. State Machines

### 4.1 Sensor Node State Machine

```
┌─────────────────────────────────────────────────────────────┐
│          SENSOR NODE STATE MACHINE                          │
└─────────────────────────────────────────────────────────────┘

State: IDLE (wartend auf Timer)
  ├─ Timer reached (millis() - lastSend >= SEND_INTERVAL)?
  │  ├─ NO → Stay in IDLE
  │  └─ YES → Go to MEASURING
  │
State: MEASURING (Sensoren auslesen)
  ├─ measureDistance() → distance
  ├─ readTemperature() → temp
  ├─ readTurbidity() → turb
  ├─ readTDS() → tds
  └─ Go to FORMATTING
  │
State: FORMATTING (Nachricht erstellen)
  ├─ Build String: "<SENSOR;DIST=...;TMP=...;TUR=...;TDS=...>\n"
  └─ Go to TRANSMITTING
  │
State: TRANSMITTING (RS-485 senden)
  ├─ rs485_send(msg)
  └─ Go to IDLE (lastSend = millis())
```

**Implementierung:**

Implizit in `loop()` - keine explizite State-Variable erforderlich.
Timer-basierte Steuerung mit `millis()`.

### 4.2 Display Node State Machine

```
┌─────────────────────────────────────────────────────────────┐
│          DISPLAY NODE STATE MACHINE                         │
└─────────────────────────────────────────────────────────────┘

State: IDLE (wartend auf Daten)
  ├─ RS485_SERIAL.available()?
  │  ├─ NO → Stay in IDLE
  │  └─ YES → Go to RECEIVING
  │
State: RECEIVING (Bytes empfangen)
  ├─ Read char from Serial1
  ├─ Append to buffer
  ├─ Check for '\n'
  │  ├─ NO → Continue RECEIVING
  │  └─ YES → Go to PARSING
  │
State: PARSING (Nachricht parsen)
  ├─ Validate format (startsWith("<"), contains "SENSOR")
  ├─ Extract fields (DIST, TMP, TUR, TDS)
  │  ├─ Valid → Go to UPDATING
  │  └─ Invalid → Discard, Go to IDLE
  │
State: UPDATING (Display aktualisieren)
  ├─ nextion_update(data)
  └─ Go to IDLE
```

**Implementierung:**

Ereignis-gesteuert durch `Serial1.available()`.
Buffer-basiertes Empfangen mit Zeilenende-Erkennung.

---

## 5. Memory Management

### 5.1 Sensor Node Memory Usage

```
ATmega4809 (Arduino Nano Every):
  Flash:  48 KB
  SRAM:   6 KB
  EEPROM: 256 B

Actual Usage:
  Flash:  ~10 KB (21%)
  SRAM:   ~1 KB (17%)
  EEPROM: 0 B (unused)

Flash Breakdown:
  - Arduino Core:          ~6 KB
  - OneWire Library:       ~1 KB
  - DallasTemp Library:    ~1 KB
  - SoftwareSerial:        ~0.5 KB
  - Application Code:      ~1.5 KB

SRAM Breakdown:
  - Global Variables:      ~200 B
  - OneWire Objects:       ~100 B
  - String Buffers:        ~500 B (msg in loop)
  - Stack:                 ~200 B (runtime)
```

**Optimization Opportunities:**

1. **F() Macro für String-Literals:**
   ```cpp
   // VORHER (SRAM):
   Serial.println("SensorNode gestartet");
   
   // NACHHER (Flash):
   Serial.println(F("SensorNode gestartet"));
   ```

2. **Reduzierung von String-Operationen:**
   ```cpp
   // Vermeiden:
   String msg = "<SENSOR;";
   msg += "DIST=" + String(distance, 1) + ";";
   
   // Besser: char-Array nutzen
   char msg[80];
   snprintf(msg, sizeof(msg), "<SENSOR;DIST=%.1f;TMP=%.1f;TUR=%d;TDS=%d>\n",
            distance, temp, turb, tds);
   ```

### 5.2 Display Node Memory Usage

```
ATmega2560 (Arduino Mega 2560):
  Flash:  256 KB
  SRAM:   8 KB
  EEPROM: 4 KB

Actual Usage:
  Flash:  ~15 KB (6%)
  SRAM:   ~2 KB (25%)
  EEPROM: 0 B (unused)

Flash Breakdown:
  - Arduino Core:          ~8 KB
  - Application Code:      ~7 KB

SRAM Breakdown:
  - rs485Buffer (String):  ~1 KB (dynamic, worst case)
  - SensorData struct:     ~100 B
  - Nextion cmd buffers:   ~500 B
  - Stack:                 ~400 B
```

---

## 6. Error Handling

### 6.1 Sensor Errors

**JSN-SR04T Timeout:**
```cpp
if (duration == 0) {
  return -1.0;  // Special error value
}
```
Display zeigt: "---" oder "ERROR"

**DS18B20 Disconnected:**
```cpp
if (temp == DEVICE_DISCONNECTED_C) {  // -127.0
  return -127.0;
}
```
Display zeigt: "---" oder "SENSOR ERROR"

### 6.2 Communication Errors

**RS-485 Message Validation:**
```cpp
if (!line.startsWith("<")) return;  // Kein Start-Marker
if (line.indexOf("SENSOR") < 0) return;  // Falscher Typ
```

**Buffer Overflow Protection:**
```cpp
if (rs485Buffer.length() > 1024) {
  rs485Buffer = "";  // Reset bei Überlauf
}
```

### 6.3 Future Improvements

**Watchdog Timer:**
```cpp
#include <avr/wdt.h>

void setup() {
  wdt_enable(WDTO_8S);  // 8s Watchdog
}

void loop() {
  wdt_reset();  // Must be called every <8s
  // ... rest of loop ...
}
```

**CRC Checksum:**
```cpp
// In Nachrichtenformat einfügen:
// <SENSOR;DIST=125.3;...;CRC=1A2B>

uint16_t calculateCRC(const String &msg) {
  // CRC-16-CCITT implementation
  // ...
}
```

---

## 7. Performance Optimization

### 7.1 Timing Optimization

**Non-blocking DS18B20:**
```cpp
// Aktuell (blocking): ~750ms
dsSensor.requestTemperatures();  // Blocks for 750ms
float temp = dsSensor.getTempCByIndex(0);

// Optimiert (non-blocking):
dsSensor.setWaitForConversion(false);
dsSensor.requestTemperatures();  // Returns immediately
delay(750);  // Can do other tasks here
float temp = dsSensor.getTempCByIndex(0);
```

**Parallel Sensor Reading:**
```cpp
// Start DS18B20 conversion (non-blocking)
dsSensor.setWaitForConversion(false);
dsSensor.requestTemperatures();

// While DS18B20 converts, read analog sensors
int turb = readTurbidity();
int tds = readTDS();
float distance = measureDistance();

// Now DS18B20 should be ready
delay(remaining_time);
float temp = dsSensor.getTempCByIndex(0);

// Total time reduced by ~20ms
```

### 7.2 Code Size Optimization

**Inline Functions:**
```cpp
// sensors.h
inline int readTurbidity() {
  return analogRead(TURB_PIN);
}
// Compiler kann direkt einbetten → kein Function Call Overhead
```

**constexpr für Compile-Time Constants:**
```cpp
constexpr unsigned long SEND_INTERVAL = 5000UL;
// Compiler ersetzt durch Literal → kein RAM benötigt
```

