# BioSync - Vollständige Projektdokumentation / Complete Project Documentation

## 🇩🇪 Deutsche Version

### Inhaltsverzeichnis
1. Projektübersicht
2. Systemarchitektur
3. Hardware-Komponenten
4. Installation und Verdrahtung
5. Software-Setup
6. Inbetriebnahme
7. Wartung und Fehlerbehebung
8. Erweiterungsmöglichkeiten

---

## 1. Projektübersicht

BioSync ist ein modulares Monitoring-System zur Überwachung von Klärgruben. Es erfasst kontinuierlich Füllstand, Temperatur, Wasserqualität (TDS) und Trübung und visualisiert die Daten auf einem Touch-Display.

**Hauptmerkmale:**
- Robuste RS-485 Datenübertragung über CAT7-Kabel
- Modularer Aufbau (Sensor-Node + Display-Node)
- Wasserdichte Sensoren für raue Umgebungen
- Echtzeit-Visualisierung auf Nextion-Display
- Wartungsarm und energieeffizient

---

## 2. Systemarchitektur

### 2.1 Sensor-Node (im Schacht)
- **Mikrocontroller:** Arduino Nano Every
- **Aufgaben:** Sensorwerte erfassen, aufbereiten und über RS-485 senden
- **Sensoren:**
  - JSN-SR04T: Ultraschall-Füllstandsmessung
  - MICREEN-DTSMK: DS18B20 Temperaturmessung
  - TSW-20M: Optische Trübungsmessung
  - CQRSENTDS01: TDS-Messung (gelöste Stoffe)

### 2.2 Display-Node (im Haus)
- **Mikrocontroller:** Arduino Mega
- **Aufgaben:** RS-485 Daten empfangen, parsen und auf Display anzeigen
- **Display:** Nextion NX4024T032 (3,2" Touch)

### 2.3 Datenübertragung
- **Protokoll:** RS-485 (Half-Duplex, 9600 Baud)
- **Nachrichtenformat:** `<SENSOR;DIST=123.4;TMP=18.3;TUR=512;TDS=420>`
- **Medium:** CAT7-Kabel (störsicher, wetterfest)

---

## 3. Hardware-Komponenten

### 3.1 Sensor-Node Komponenten

| Komponente | Funktion | Signaltyp | Arduino Pin |
|------------|----------|-----------|-------------|
| Arduino Nano Every | Hauptsteuerung | - | - |
| JSN-SR04T | Füllstandsmessung | Digital (TRIG/ECHO) | D2 (TRIG), D3 (ECHO) |
| MICREEN-DTSMK | Temperaturmessung | 1-Wire (DS18B20) | D4 |
| TSW-20M | Trübungsmessung | Analog | A0 |
| CQRSENTDS01 | TDS-Messung | Analog | A1 |
| RS-485 Modul | Datenübertragung | Digital (TX/RX) | D6 (TX), D7 (RX) |
| LM2596 Step-Down | Spannungsversorgung | 12V → 5V | VIN, GND |
| IP67-Gehäuse | Schutz | - | - |

### 3.2 Display-Node Komponenten

| Komponente | Funktion | Signaltyp | Arduino Pin |
|------------|----------|-----------|-------------|
| Arduino Mega | Hauptsteuerung | - | - |
| Nextion NX4024T032 | Touch-Display | Serial | D16 (TX2), D17 (RX2) |
| RS-485 Modul | Datenempfang | Serial | D18 (TX1), D19 (RX1) |
| LM2596 Step-Down | Spannungsversorgung | 12V → 5V | 5V, GND |
| 12V DC Netzteil | Stromversorgung | - | - |

---

## 4. Installation und Verdrahtung

### 4.1 Spannungsversorgung

**Im Haus:**
1. 12V DC Netzteil (min. 2A) an Steckdose anschließen
2. LM2596 Step-Down Modul anschließen:
   - IN+ → 12V+ vom Netzteil
   - IN- → 12V- (GND) vom Netzteil
3. Ausgangsspannung mit Multimeter auf exakt 5,0V einstellen
4. Arduino Mega und Nextion Display versorgen

**Im Schacht:**
1. 12V über CAT7-Kabel empfangen (2 Adern +12V, 2 Adern GND)
2. LM2596 Step-Down Modul anschließen und auf 5,0V einstellen
3. Arduino Nano Every und alle Sensoren versorgen

### 4.2 Sensor-Node Verdrahtung

**JSN-SR04T (Ultraschall):**
```
VCC → 5V
GND → GND
TRIG → D2
ECHO → D3
```

**MICREEN-DTSMK (DS18B20 Temperatur):**
```
Rot (VCC) → 5V
Schwarz (GND) → GND
Gelb (DATA) → D4 + 4,7kΩ Pull-up zu 5V
```

**TSW-20M (Trübung):**
```
VCC → 5V
GND → GND
OUT → A0
```

**CQRSENTDS01 (TDS):**
```
VCC → 5V
GND → GND
OUT → A1
```

**RS-485 Modul:**
```
VCC → 5V
GND → GND
RO (Receiver Output) → D7 (RX)
DI (Driver Input) → D6 (TX)
DE/RE → D5 (Transmit Enable)
A, B → CAT7-Kabel zu Display-Node
```

### 4.3 Display-Node Verdrahtung

**RS-485 Modul:**
```
VCC → 5V
GND → GND
RO → D19 (RX1)
DI → D18 (TX1)
DE/RE → D10 (optional)
A, B → CAT7-Kabel von Sensor-Node
```

**Nextion Display:**
```
5V → 5V
GND → GND
TX → D17 (RX2)
RX → D16 (TX2)
```

### 4.4 CAT7-Kabel Belegung

| Ader-Paar | Funktion |
|-----------|----------|
| 1-2 | +12V Versorgung |
| 3-4 | GND (Masse) |
| 5-6 | RS-485 A/B (Daten) |
| 7-8 | Reserve |

---

## 5. Software-Setup

### 5.1 Benötigte Bibliotheken

**Arduino IDE Bibliotheken installieren:**
- OneWire
- DallasTemperature
- SoftwareSerial (meist vorinstalliert)

**Installation:**
1. Arduino IDE öffnen
2. Sketch → Include Library → Manage Libraries
3. Nach "OneWire" suchen → Installieren
4. Nach "DallasTemperature" suchen → Installieren

### 5.2 Sensor-Node programmieren

1. `SensorNode/SensorNode.ino` in Arduino IDE öffnen
2. Board auswählen: Tools → Board → Arduino Nano Every
3. Port auswählen: Tools → Port → [Dein COM-Port]
4. `config.h` prüfen und ggf. Pins anpassen
5. Sketch hochladen

**Wichtige config.h Einstellungen:**
```cpp
// Pins
static const uint8_t TRIG_PIN    = 2;
static const uint8_t ECHO_PIN    = 3;
static const uint8_t DS18B20_PIN = 4;
static const uint8_t TURB_PIN    = A0;
static const uint8_t TDS_PIN     = A1;
static const uint8_t RS485_RX    = 7;
static const uint8_t RS485_TX    = 6;
static const uint8_t RS485_DE    = 5;

// Parameter
static const unsigned long SEND_INTERVAL = 5000UL; // 5 Sekunden
static const unsigned long RS485_BAUD = 9600UL;
```

### 5.3 Display-Node programmieren

1. `DisplayNode/DisplayNode.ino` in Arduino IDE öffnen
2. Board auswählen: Tools → Board → Arduino Mega
3. Port auswählen: Tools → Port → [Dein COM-Port]
4. `config.h` prüfen und ggf. anpassen
5. Sketch hochladen

**Wichtige config.h Einstellungen:**
```cpp
#define RS485_SERIAL Serial1    // RX1=D19, TX1=D18
#define NEXTION_SERIAL Serial2  // RX2=D17, TX2=D16
static const uint8_t RS485_DE_PIN = 10;

// Nextion Komponenten-Namen
static constexpr const char* NEXTION_TDIST  = "tDist";
static constexpr const char* NEXTION_TTEMP  = "tTemp";
static constexpr const char* NEXTION_TTURB  = "tTurb";
static constexpr const char* NEXTION_TTDS   = "tTDS";
```

### 5.4 Nextion Display programmieren

**Komponenten anlegen:**

1. **Hauptseite** (`Page_main`) erstellen mit:
   - Number-Objekt: `nIdle` (val=0)
   - Timer: `tIdleTimer` (Interval=1000, En=1)
   - Textfelder: `tDist`, `tTemp`, `tTurb`, `tTDS`
   - Button: `btnRefresh`

2. **Screensaver-Seite** (`Page_screensaver`) erstellen mit:
   - Transparenter Button: `btnWake` (gesamte Seite)

**Event-Code einfügen:**

**btnRefresh - TouchPress Event:**
```
nIdle.val=0
print "BTN_REFRESH"
```

**tIdleTimer - Timer Event:**
```
nIdle.val=nIdle.val+1
if nIdle.val>60
page Page_screensaver
nIdle.val=0
endif
```

**btnWake - TouchPress Event:**
```
page Page_main
nIdle.val=0
print "SCR_WAKE"
```

3. HMI kompilieren und auf Display hochladen (via SD-Karte oder seriell)

---

## 6. Inbetriebnahme

### 6.1 Erstinbetriebnahme - Checkliste

**Vor Stromversorgung:**
- [ ] Alle Lötverbindungen visuell prüfen
- [ ] Kurzschlüsse mit Multimeter ausschließen
- [ ] Polarität aller Anschlüsse prüfen (+/-) 
- [ ] LM2596 Ausgangsspannung auf 5,0V eingestellt
- [ ] Kabelverbindungen fest und isoliert

**Sensor-Node Test:**
1. Arduino Nano über USB verbinden
2. Seriellen Monitor öffnen (115200 Baud)
3. Ausgabe prüfen: "SensorNode gestartet"
4. Sensorwerte prüfen (alle 5 Sekunden neue Werte)
5. RS-485 Message-Format pr��fen

**Display-Node Test:**
1. Arduino Mega über USB verbinden
2. Seriellen Monitor öffnen (115200 Baud)
3. Ausgabe prüfen: "DisplayNode gestartet"
4. Nextion Display zeigt Werte an
5. Touch-Interaktion testen

**System-Test:**
1. Beide Nodes über CAT7 verbinden
2. 12V Versorgung einschalten
3. Sensor-Node sendet Daten → Display-Node empfängt
4. Display zeigt aktuelle Werte an
5. Refresh-Button testen

### 6.2 Kalibrierung

**Füllstand (JSN-SR04T):**
- Null-Punkt: Sensor-Abstand zur Wasseroberfläche bei leer
- Maximum: Sensor-Abstand bei vollem Tank
- In Software umrechnen: `Füllstand (%) = (Max - Aktuell) / Max * 100`

**TDS-Sensor (CQRSENTDS01):**
- Destilliertes Wasser: ~0 ppm (Null-Punkt)
- Kalibrierungs-Lösung (z.B. 1413 µS/cm): bekannter Wert
- Faktor in Software anpassen

---

## 7. Wartung und Fehlerbehebung

### 7.1 Regelmäßige Wartung

**Monatlich:**
- Display-Werte mit manuellen Messungen vergleichen
- Sichtkontrolle aller Sensoren im Schacht
- Nextion Display reinigen

**Vierteljährlich:**
- Sensoren reinigen (besonders Trübungs- und TDS-Sensor)
- Kabelverbindungen prüfen
- Gehäuse-Dichtungen prüfen (IP67)

**Jährlich:**
- Sensoren kalibrieren
- Firmware-Updates prüfen
- Backup der Nextion-HMI erstellen

### 7.2 Häufige Probleme und Lösungen

**Problem: Display zeigt keine Werte**
- ✓ RS-485 Kabelverbindung prüfen (A ↔ A, B ↔ B)
- ✓ GND-Verbindung zwischen Nodes prüfen
- ✓ Seriellen Monitor Display-Node prüfen (Empfang?)
- ✓ Baudrate 9600 in beiden Nodes
- ✓ DE/RE Pin korrekt gesteuert

**Problem: Sensor-Werte unplausibel**
- ✓ Sensor-Verkabelung prüfen (VCC, GND, Signal)
- ✓ Pull-up Widerstand bei DS18B20 (4,7kΩ)
- ✓ Sensoren reinigen
- ✓ Kalibrierung prüfen

**Problem: Display friert ein**
- ✓ Stromversorgung stabil? (Multimeter: 5V ±0,2V)
- ✓ Nextion-Baudrate korrekt?
- ✓ Zu viele Updates pro Sekunde? (Delays erhöhen)

**Problem: RS-485 Übertragung instabil**
- ✓ Terminierungswiderstände an Busenden (120Ω)
- ✓ Kabelqualität prüfen (CAT7 empfohlen)
- ✓ Maximale Kabellänge nicht überschreiten (500m)
- ✓ Störquellen in der Nähe? (Motoren, Netzteile)

**Problem: DS18B20 zeigt -127°C**
- ✓ Sensor angeschlossen?
- ✓ Pull-up Widerstand vorhanden?
- ✓ Kabel nicht zu lang (max. 3m ohne Verstärkung)
- ✓ OneWire-Adresse in Code korrekt?

### 7.3 Debug-Tipps

**Serieller Monitor nutzen:**
```cpp
// In SensorNode.ino loop()
Serial.print("Distanz: "); Serial.println(distance);
Serial.print("Temperatur: "); Serial.println(temp);
Serial.print("Trübung: "); Serial.println(turb);
Serial.print("TDS: "); Serial.println(tds);
```

**Nextion Debug:**
- Verbinde Display per USB-TTL Adapter mit PC
- Öffne Nextion Editor → Debug → Connect
- Zeige Live-Updates im Editor

---

## 8. Erweiterungsmöglichkeiten

### 8.1 Hardware-Erweiterungen

**SD-Karten Datenlogger:**
- SD-Modul an Display-Node SPI anschließen
- Werte mit Zeitstempel speichern
- CSV-Export für Analyse

**Alarm-Ausgänge:**
- Relais für Pumpensteuerung
- Buzzer für akustische Alarme
- LED-Signalleuchten

**Zusatzsensoren:**
- pH-Wert Sensor
- Sauerstoffgehalt (DO)
- Durchflussmesser

**Netzwerkanbindung:**
- ESP32 für WiFi/MQTT
- LoRa für Fernübertragung
- GSM-Modul für SMS-Benachrichtigungen

### 8.2 Software-Erweiterungen

**Alarme und Schwellwerte:**
```cpp
// In DisplayNode.ino
if (d.distance < 10.0) {
  nextion_send_raw(Serial2, "page Page_alarm");
  // Relais schalten, Buzzer aktivieren
}
```

**Datenlogging:**
```cpp
#include <SD.h>
// SD-Karte initialisieren, Werte in CSV schreiben
File dataFile = SD.open("data.csv", FILE_WRITE);
dataFile.println(timestamp + "," + distance + "," + temp);
dataFile.close();
```

**Web-Interface:**
- ESP32 als WiFi-Bridge
- Webserver für Browser-Zugriff
- REST-API für externe Systeme

### 8.3 Nextion HMI-Erweiterungen

**Graphen/Trends:**
- Waveform-Komponente für Füllstandverlauf
- Historische Daten visualisieren

**Mehrseitige Navigation:**
- Einstellungsseite
- Statistik-Seite
- Alarm-Historie

**Benutzer-Interaktion:**
- Manuelle Pumpensteuerung
- Schwellwerte konfigurieren
- Sprache umschalten (DE/EN)

---

## Anhang / Appendix

### Lizenz / License
MIT License - siehe LICENSE Datei / see LICENSE file

### Kontakt / Contact
GitHub: LukeArrow/BioSync

### Versions-Historie / Version History
- v1.0 (2025): Initial release mit modularer Architektur / Initial release with modular architecture

---

**Letzte Aktualisierung / Last Update:** 2025-11-01
