# BioSync - Detaillierte Installationsanleitung
## Schritt-für-Schritt Installation und Inbetriebnahme

**Version:** 2.0  
**Datum:** November 2025

---

## Inhaltsverzeichnis

1. [Vor der Installation](#1-vor-der-installation)
2. [Werkzeuge und Materialien](#2-werkzeuge-und-materialien)
3. [Hardware-Aufbau Sensor Node](#3-hardware-aufbau-sensor-node)
4. [Hardware-Aufbau Display Node](#4-hardware-aufbau-display-node)
5. [Kabelverlegung](#5-kabelverlegung)
6. [Software-Installation](#6-software-installation)
7. [Inbetriebnahme und Test](#7-inbetriebnahme-und-test)
8. [Finalisierung](#8-finalisierung)

---

## 1. Vor der Installation

### 1.1 Standort-Bewertung

**Sensor Node (Schacht):**
- [ ] Zugang zum Schacht vorhanden?
- [ ] Montageposition für Sensor gefunden? (über Wasseroberfläche, senkrecht)
- [ ] Kabeldurchführung möglich? (IP67 Kabelverschraubung)
- [ ] Schutz vor direkter Sonneneinstrahlung gegeben?

**Display Node (Haus):**
- [ ] Montageposition im Haus gewählt? (gut sichtbar)
- [ ] Stromversorgung 230V vorhanden?
- [ ] Kabelweg zum Schacht geplant?
- [ ] Schutz vor Feuchtigkeit gegeben?

### 1.2 Sicherheitshinweise

⚠️ **WICHTIG:**
- Netzteil VOR jeder Arbeit vom Strom trennen
- Polarität IMMER prüfen (+ und -)
- Keine losen Kabelenden (Kurzschlussgefahr)
- Arbeiten in trockener Umgebung

---

## 2. Werkzeuge und Materialien

### 2.1 Werkzeuge

- [ ] Multimeter (Spannungsmessung, Durchgangsprüfung)
- [ ] Lötkolben + Lötzinn (optional, für dauerhaften Aufbau)
- [ ] Abisolierzange
- [ ] Seitenschneider
- [ ] Schraubendreher-Set (Kreuz, Schlitz, klein)
- [ ] Kabelschneider für CAT7
- [ ] Bohrmaschine (für Gehäusemontage)
- [ ] Maßband / Zollstock

### 2.2 Materialien (siehe HARDWARE_DETAILED.md für BOM)

---

## 3. Hardware-Aufbau Sensor Node

### 3.1 LM2596 Vorbereitung und Einstellung

**KRITISCH: Vor dem Anschluss an Komponenten!**

```
Schritt 1: Multimeter vorbereiten
  - DC Voltage Bereich (20V)
  - Rote Sonde: OUT+
  - Schwarze Sonde: OUT-

Schritt 2: LM2596 an 12V anschließen (OHNE LAST)
  - IN+ an 12V Quelle
  - IN- an GND
  - NICHTS an OUT+/OUT-

Schritt 3: Spannung einstellen
  - Kleiner Schlitzschraubendreher
  - Potentiometer vorsichtig drehen
  - Ziel: Multimeter zeigt exakt 5,00V
  - ⚠️ NIE über 5,3V einstellen!

Schritt 4: Verifikation
  - Netzteil aus, 10s warten
  - Netzteil wieder ein
  - Spannung erneut messen: sollte 5,00V ±0.02V sein

Schritt 5: Beschriften
  - LM2596 mit wasserfestem Marker: "5V - Sensor Node"
  - Poti-Position markieren
```

### 3.2 Komponenten auf Breadboard/Platine

**Layout-Plan (siehe HARDWARE_DETAILED.md für Details)**

```
Reihenfolge:

1. LM2596 montieren
   - Befestigung mit M3 Schrauben
   - OUT+ an 5V Rail
   - OUT- an GND Rail

2. Arduino Nano Every einsetzen
   - VIN an 5V Rail
   - GND an GND Rail
   - Power-LED sollte leuchten (wenn LM2596 eingeschaltet)

3. Sensoren anschließen:
   a) JSN-SR04T
      - VCC → 5V, GND → GND
      - TRIG → D2, ECHO → D3
   
   b) DS18B20
      - VCC → 5V, GND → GND
      - DATA → D4
      - ⚠️ 4.7kΩ Pull-up: zwischen DATA und 5V!
   
   c) TSW-20M
      - VCC → 5V, GND → GND
      - OUT → A0
   
   d) CQRSENTDS01
      - VCC → 5V, GND → GND
      - OUT → A1

4. RS-485 Modul
   - VCC → 5V, GND → GND
   - RO → D7, DI → D6, DE/RE → D5
   - A, B: vorerst offen (später an CAT7)
```

### 3.3 Funktionstest (vor Gehäuse-Einbau)

```
1. USB-Kabel an Arduino anschließen
2. Arduino IDE öffnen
3. SensorNode.ino öffnen und hochladen
4. Serial Monitor öffnen (115200 baud)
5. Erwartete Ausgabe alle 5 Sekunden:
   "Gesendet: <SENSOR;DIST=xxx.x;TMP=xx.x;TUR=xxx;TDS=xxx>"

6. Sensoren einzeln prüfen:
   - JSN-SR04T: Hand vor Sensor → Distanz ändert sich?
   - DS18B20: Sollte Raumtemperatur zeigen (~20°C)
   - TSW-20M: Wert 0-1023
   - CQRSENTDS01: Wert 0-1023

7. Wenn alle OK: Weiter zu Gehäuse-Einbau
```

### 3.4 Gehäuse-Einbau (IP67)

```
1. Komponenten im Gehäuse anordnen
   - LM2596 nahe Kabeldurchführung (Wärmeableitung)
   - Arduino zentral
   - Sensorkabel zu separaten Durchführungen

2. Befestigung
   - Mit M3 Schrauben oder Heißkleber
   - Abstand zu Metallgehäuse (Kurzschlussgefahr)
   - Zugentlastung für Kabel

3. Kabelverschraubungen (M12)
   - 12V Einspeisung (CAT7)
   - Sensorkabel (JSN-SR04T, DS18B20, etc.)
   - Dichtungen fest anziehen

4. Silica-Gel Beutel
   - 1-2 Beutel im Gehäuse
   - Absorbiert Restfeuchtigkeit

5. Deckel schließen
   - Dichtung prüfen
   - Schrauben gleichmäßig anziehen
```

---

## 4. Hardware-Aufbau Display Node

### 4.1 LM2596 Vorbereitung (analog zu Sensor Node)

```
Wiederhole Schritte 3.1
Beschriftung: "5V - Display Node"
```

### 4.2 Komponenten aufbauen

```
1. LM2596 an 12V Netzteil anschließen (NOCH NICHT EINSTECKEN!)
   - IN+ an PSU +
   - IN- an PSU -
   - OUT+ an 5V Rail
   - OUT- an GND Rail

2. Arduino Mega einsetzen
   - 5V an 5V Rail
   - GND an GND Rail

3. RS-485 Modul
   - VCC → 5V, GND → GND
   - RO → D19 (RX1), DI → D18 (TX1)
   - DE/RE → GND (Empfangsmodus)
   - A, B: vorerst offen

4. Nextion Display
   - VCC → 5V, GND → GND
   - TX → D17 (RX2), RX → D16 (TX2)
   ⚠️ TX vom Display zu RX am Mega!
```

### 4.3 Funktionstest

```
1. Netzteil EINSTECKEN
2. Erwartung:
   - Arduino Mega Power-LED: ON
   - Nextion Display: Bootscreen, dann HMI

3. USB-Kabel an Mega
4. Arduino IDE: DisplayNode.ino hochladen
5. Serial Monitor (115200 baud)
6. Erwartung: "DisplayNode gestartet"

7. Nextion Test:
   - Display sollte Main Page zeigen
   - Textfelder: "---" (initial)
```

---

## 5. Kabelverlegung

### 5.1 CAT7-Kabel vorbereiten

```
1. Kabel ablängen
   - Benötigte Länge + 2m Reserve
   - Mit Kabelschneider sauber schneiden

2. Beide Enden abisolieren
   - Mantel: 10 cm
   - Einzelne Adern: 5 mm
   - Adern verzinnen (optional, verhindert Oxidation)

3. Adern beschriften
   - Mit Markern oder Beschriftungsband
   - 1+2: 12V+
   - 3+4: GND
   - 5+6: RS-485 A/B
   - 7+8: Reserve
```

### 5.2 Verlegung Haus → Schacht

**Option A: Erdverlegung (empfohlen)**

```
1. Graben ausheben
   - Tiefe: 60-80 cm (frostfrei)
   - Breite: 20 cm

2. Schutzrohr (Wellrohr DN25) verlegen
   - CAT7 durch Rohr ziehen
   - Sandbett (5 cm) unter und über Rohr

3. Warnband
   - "Achtung Kabel" 20 cm über Rohr

4. Graben verfüllen
```

**Option B: Oberirdisch**

```
1. Kabelbrücke oder -rinne verwenden
2. UV-beständiges CAT7 (PVC-Mantel)
3. Befestigung alle 1 m
4. Zugentlastung an Durchführungen
5. Tropfschlaufe vor Gebäude-Eingang
```

### 5.3 Anschluss im Haus (Display Node)

```
1. Adern an LM2596 und RS-485 anschließen:
   - Adern 1+2 (12V+) → zusammen an IN+ (LM2596)
   - Adern 3+4 (GND) → zusammen an IN- (LM2596)
   - Ader 5 (Blau-Weiß) → RS-485 A
   - Ader 6 (Grün) → RS-485 B

2. 12V Netzteil anschließen:
   - PSU + → LM2596 IN+ (parallel zu CAT7 1+2)
   - PSU - → LM2596 IN- (parallel zu CAT7 3+4)

3. Durchgang prüfen (Multimeter):
   - Zwischen CAT7 1+2 und PSU +: <1Ω
   - Zwischen CAT7 3+4 und PSU -: <1Ω
```

### 5.4 Anschluss im Schacht (Sensor Node)

```
1. CAT7 durch Kabelverschraubung ins Gehäuse führen

2. Adern an LM2596 und RS-485 anschließen:
   - Adern 1+2 (12V+) → zusammen an IN+ (LM2596)
   - Adern 3+4 (GND) → zusammen an IN- (LM2596)
   - Ader 5 (Blau-Weiß) → RS-485 A
   - Ader 6 (Grün) → RS-485 B

3. Zugentlastung anbringen
   - Kabel mit Kabelbinder an Gehäuse-Innenseite fixieren
   - Verhindert Zug an Lötstellen

4. Gehäuse verschließen
```

---

## 6. Software-Installation

### 6.1 Arduino IDE Setup

```
1. Download Arduino IDE (https://www.arduino.cc/en/software)
2. Install

3. Bibliotheken installieren:
   Tools → Manage Libraries
   - OneWire (by Paul Stoffregen)
   - DallasTemperature (by Miles Burton)

4. Board-Paket (für Nano Every):
   Tools → Board → Boards Manager
   - "megaAVR" by Arduino
   - Install
```

### 6.2 SensorNode Firmware Upload

```
1. File → Open → SensorNode/SensorNode.ino
2. Tools → Board → Arduino Nano Every
3. Tools → Port → (Select COM port)
4. Sketch → Upload
5. Wait for "Done uploading"
6. Open Serial Monitor (115200 baud)
7. Verify: Messages every 5 seconds
```

### 6.3 DisplayNode Firmware Upload

```
1. File → Open → DisplayNode/DisplayNode.ino
2. Tools → Board → Arduino Mega 2560
3. Tools → Port → (Select COM port)
4. Sketch → Upload
5. Wait for "Done uploading"
6. Serial Monitor: Should show received messages
```

### 6.4 Nextion HMI Upload

```
1. Compile .tft file in Nextion Editor
2. Copy .tft to microSD card (FAT32 formatted)
3. Power off Nextion
4. Insert microSD card
5. Power on Nextion
6. Display shows "Update Success" after ~10s
7. Power off, remove microSD, power on
```

---


### 6.5 RTC DS3231 Setup ⏰

**Bibliothek installieren:**

```
1. Arduino IDE → Tools → Manage Libraries
2. Suche nach "RTClib"
3. Install "RTClib by Adafruit" (Version 2.x oder höher)
```

**Initiale Zeit setzen:**

```cpp
// Code-Beispiel zum Setzen der RTC-Zeit (einmalig ausführen)
#include <RTClib.h>

RTC_DS3231 rtc;

void setup() {
  Serial.begin(115200);
  
  if (!rtc.begin()) {
    Serial.println("ERROR: RTC not found!");
    while (1) delay(10);
  }
  
  // Setze Zeit auf Compile-Zeit (einmalig beim Upload)
  rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  
  // ODER: Manuelle Zeit setzen
  // rtc.adjust(DateTime(2025, 11, 3, 9, 0, 0)); // Jahr, Monat, Tag, Stunde, Minute, Sekunde
  
  Serial.println("RTC time set successfully!");
}

void loop() {
  DateTime now = rtc.now();
  Serial.print(now.year());
  Serial.print('-');
  Serial.print(now.month());
  Serial.print('-');
  Serial.print(now.day());
  Serial.print(' ');
  Serial.print(now.hour());
  Serial.print(':');
  Serial.print(now.minute());
  Serial.print(':');
  Serial.println(now.second());
  
  delay(1000);
}
```

**Zeitzone anpassen:**

```
Für Mitteleuropäische Zeit (MEZ/CET):
- Winterzeit: UTC +1 Stunde
- Sommerzeit: UTC +2 Stunden

Im Code nach Bedarf anpassen:
  DateTime adjusted = now + TimeSpan(0, 1, 0, 0); // +1 Stunde
```

**CR2032 Batterie:**

```
□ CR2032 Knopfzelle im RTC-Modul einlegen
□ Batterie hält ca. 5-10 Jahre
□ Bei Stromausfall: Zeit bleibt erhalten
□ Warnung im Serial Monitor wenn Batterie leer
```

### 6.6 SD-Karte Vorbereitung 💾

**SD-Karte formatieren:**

```
1. SD-Karte (empfohlen: 4-32 GB, Class 10)
2. Am PC anschließen
3. Formatieren als FAT32:
   - Windows: Rechtsklick → Formatieren → FAT32
   - Linux: sudo mkfs.vfat -F 32 /dev/sdX
   - Mac: Disk Utility → Erase → MS-DOS (FAT)
4. Label (optional): "BIOSYNC"
```

**Automatische Dateierstellung:**

```
Die DisplayNode-Firmware erstellt automatisch beim ersten Start:

sensor.csv:
  Timestamp,Distance_cm,Temperature_C,Turbidity,TDS_ppm
  (Header wird automatisch geschrieben)

relay.csv:
  Timestamp,Event,Pump_State,Vent_State
  (Header wird automatisch geschrieben)

Status-LED (D53):
  - Blinkt beim Schreibvorgang
  - Leuchtet dauerhaft bei erfolgreicher Initialisierung
  - Aus bei SD-Karten-Fehler
```

**Freien Speicher überwachen:**

```cpp
// Im Serial Monitor wird angezeigt:
SD Card initialized
  Card Type: SDHC
  Volume: FAT32
  Size: 7.40 GB
  Free: 7.38 GB

Bei <100 MB frei: Warnung im Serial Monitor
```

**Troubleshooting SD-Karte:**

```
Problem: "SD Card initialization failed"
Lösung:
  □ SD-Karte als FAT32 formatiert?
  □ CS-Pin korrekt (D53)?
  □ Verkabelung SPI-Bus prüfen
  □ Andere SD-Karte testen
  □ SD-Karte vollständig eingesteckt?

Problem: "File write error"
Lösung:
  □ SD-Karte voll?
  □ Schreibschutz-Schalter an Karte?
  □ SD-Karte defekt?
```

### 6.7 RelayNode Kalibrierung (falls installiert) ⚡

**Wenn RelayNode installiert ist:**

Siehe [RELAY_NODE_GUIDE.md](../RELAY_NODE_GUIDE.md) für detaillierte Kalibrierung.

**Kurzübersicht:**

```
1. RelayNode-Firmware hochladen (RelayNode/RelayNode.ino)

2. Serial Monitor öffnen (115200 baud)

3. Schwellwerte kalibrieren:
   - LED AUS: Analogwert ablesen (z.B. 50)
   - LED AN: Analogwert ablesen (z.B. 800)
   - Schwellwert = Mittelwert (z.B. 425)

4. In config.h anpassen:
   #define PUMP_THRESHOLD 425
   #define VENT_THRESHOLD 425
   // usw.

5. Firmware erneut hochladen

6. Test: LEDs ein-/ausschalten und Status im Serial Monitor prüfen
```

---

## 7. Inbetriebnahme und Test

### 7.1 Systemstart

```
Checkliste:
  [ ] Netzteil Display Node einstecken
  [ ] Mega Power-LED: ON
  [ ] Nextion Display: ON, zeigt Main Page
  [ ] Sensor Node Power-LED: ON (im Gehäuse)
  [ ] Serial Monitor Sensor Node: Sendet Nachrichten
  [ ] Serial Monitor Display Node: Empfängt Nachrichten
  [ ] Nextion Display: Zeigt Sensor-Werte
```

### 7.2 Funktionstest

**Test 1: Stromversorgung**
```
Multimeter-Messungen:
  - 12V PSU Output: 12.0V ±0.5V ✓
  - Display Node LM2596 Output: 5.0V ±0.05V ✓
  - Sensor Node LM2596 Output: 5.0V ±0.05V ✓
  - Spannung am Sensor Node (via CAT7): >11.5V ✓
```

**Test 2: Sensoren**
```
Serial Monitor Sensor Node:
  - DIST: 25-450 cm (plausibel?) ✓
  - TMP: 15-25°C (Raumtemperatur?) ✓
  - TUR: 0-1023 (nicht konstant 0 oder 1023?) ✓
  - TDS: 0-1023 (nicht konstant 0 oder 1023?) ✓
```

**Test 3: RS-485 Kommunikation**
```
Serial Monitor Display Node:
  - Empfangen: <SENSOR;...> alle 5s ✓
  - Keine Fehler (garbled data) ✓
```

**Test 4: Nextion Display**
```
Display zeigt:
  - Füllstand: xxx.x cm ✓
  - Temperatur: xx.x °C ✓
  - Trübung: xxx ✓
  - TDS: xxx ✓
  - Werte aktualisieren sich alle 5s ✓
```

### 7.3 Langzeit-Test (24h)

```
1. System 24h laufen lassen
2. Log führen (Stichproben alle 2h):
   - Timestamp
   - Alle 4 Sensorwerte
   - Auffälligkeiten?

3. Nach 24h:
   - System stabil? ✓
   - Keine Abstürze? ✓
   - Werte plausibel? ✓
   - Display reagiert auf Touch? ✓
```

---

## 8. Finalisierung

### 8.1 Kalibrierung

Siehe CALIBRATION_PROCEDURES.md für detaillierte Anleitungen

```
□ JSN-SR04T Offset-Kalibrierung
□ DS18B20 Temperatur-Offset
□ TSW-20M Null-Punkt
□ TDS Zwei-Punkt-Kalibrierung (optional)
```

### 8.2 Dokumentation

```
□ Kalibrierungsprotokoll ausfüllen
□ Kabellänge notieren
□ Firmware-Versionen notieren
□ Fotos von Installation machen
□ Kontaktdaten Techniker hinterlegen
```

### 8.3 Übergabe an Betreiber

```
Einweisung:
  □ Display-Bedienung erklären
  □ Wartungsintervalle besprechen
  □ Notfall-Kontakt hinterlassen
  □ Troubleshooting-Guide übergeben
```

---

**Installation abgeschlossen! System ist betriebsbereit. ✓**

