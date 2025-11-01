# 🧪 Testing-Guide für BioSync

## 📋 Einleitung

### Zweck des Testing-Guides
Dieser Testing-Guide führt Sie systematisch durch alle Testphasen des BioSync-Systems, von der Hardware-Prüfung bis zur vollständigen Systemintegration. Ziel ist es, ein zuverlässiges und robustes Klärgruben-Monitoring-System zu gewährleisten.

### Benötigte Werkzeuge und Materialien
- ⚡ **Multimeter** (Digital, min. 0,01V Genauigkeit)
- 🔌 **12V DC Netzteil** (min. 2A)
- 💻 **Arduino IDE** (neueste Version)
- 📡 **USB-TTL Adapter** (für RS-485 Debugging)
- 🌡️ **Referenzthermometer** (für Temperatursensor-Kalibrierung)
- 📏 **Maßband/Lineal** (für Ultraschall-Kalibrierung)
- 🔧 **Schraubendreher-Set** (für LM2596 Poti-Einstellung)
- 📱 **Serial Monitor** (Arduino IDE oder externe Tools wie PuTTY)
- 🧪 **Testbehälter mit Wasser** (für Sensor-Tests)
- 📝 **Testprotokoll** (siehe unten)

---

## ⚡ Phase 1: Hardware-Tests

### 1.1 Spannungsversorgung prüfen

#### LM2596 Step-Down Modul einstellen (Sensor-Node)

- [ ] **Schritt 1:** LM2596 vom System trennen (keine Last anschließen)
- [ ] **Schritt 2:** 12V DC Netzteil an Eingang anschließen
- [ ] **Schritt 3:** Multimeter am Ausgang anlegen (+ und GND)
- [ ] **Schritt 4:** Mit Poti (Uhrzeigersinn = höher) auf **exakt 5,00V** ± 0,05V einstellen
- [ ] **Schritt 5:** Spannung 3× prüfen und notieren

**Erwartetes Ergebnis:**
```
Eingangsspannung: 12,0V ± 0,5V
Ausgangsspannung: 5,00V ± 0,05V
```

**✅ Pass-Kriterium:** Ausgangsspannung stabil bei 5,00V

#### LM2596 Step-Down Modul einstellen (Display-Node)

Gleicher Prozess wie oben für Display-Node LM2596.

- [ ] Ausgangsspannung Display-Node: **5,00V ± 0,05V**

### 1.2 Verkabelung testen

#### Polarität prüfen
- [ ] **Plus-Leitung (rot):** Durchgängigkeit mit Multimeter (Ohm-Modus) < 1Ω
- [ ] **GND-Leitung (schwarz):** Durchgängigkeit < 1Ω
- [ ] **RS-485 A-Leitung:** Durchgängigkeit < 1Ω
- [ ] **RS-485 B-Leitung:** Durchgängigkeit < 1Ω
- [ ] **Isolation prüfen:** Widerstand zwischen + und GND > 10MΩ (Multimeter Ohm-Modus)

**⚠️ Wichtig:** Keine Kurzschlüsse zwischen Leitungen!

### 1.3 CAT7-Kabel durchmessen

- [ ] **Länge des Kabels notieren:** __________ Meter
- [ ] **Widerstand pro Ader messen:**
  - Ader 1 (12V+): ______ Ω
  - Ader 2 (GND): ______ Ω
  - Ader 3 (RS-485 A): ______ Ω
  - Ader 4 (RS-485 B): ______ Ω

**✅ Pass-Kriterium:** Widerstand < 0,2Ω pro Meter

### 1.4 Funktionstest Spannungsversorgung unter Last

- [ ] Arduino Nano Every an LM2596 (Sensor-Node) anschließen
- [ ] Spannung unter Last messen: ______ V (soll: 4,95V - 5,05V)
- [ ] Arduino Mega an LM2596 (Display-Node) anschließen
- [ ] Spannung unter Last messen: ______ V (soll: 4,95V - 5,05V)

---

## 🌊 Phase 2: Sensor-Node Tests

### 2.1 Serial Debug Output

#### Test-Setup
1. Arduino Nano Every per USB mit PC verbinden
2. Arduino IDE öffnen → Serial Monitor (115200 Baud)
3. `SensorNode.ino` hochladen

**Erwartete Ausgabe:**
```
SensorNode gestartet
Gesendet: <SENSOR;DIST=123.4;TMP=18.3;TUR=512;TDS=420>
Gesendet: <SENSOR;DIST=123.4;TMP=18.3;TUR=512;TDS=420>
...
```

- [ ] ✅ Startmeldung erscheint
- [ ] ✅ Regelmäßige Nachrichten alle 5 Sekunden
- [ ] ✅ Nachrichtenformat korrekt: `<SENSOR;...>`

### 2.2 JSN-SR04T Ultraschall-Test

#### Pinbelegung prüfen
- [ ] TRIG → Pin D2
- [ ] ECHO → Pin D3
- [ ] VCC → 5V
- [ ] GND → GND

#### Funktionstest
1. Sensor in bekannter Höhe (z.B. 30 cm über Tisch) positionieren
2. Serial Monitor beobachten

**Erwartete Ausgabe:**
```
DIST=30.2  (Distanz in cm)
```

**Tests:**
- [ ] Abstand 20 cm: Messung ______ cm (Toleranz ±2 cm)
- [ ] Abstand 50 cm: Messung ______ cm (Toleranz ±2 cm)
- [ ] Abstand 100 cm: Messung ______ cm (Toleranz ±3 cm)
- [ ] Keine Messung bei > 400 cm: Wert ______ (soll: 0 oder Fehlermeldung)

**🔧 Debugging:**
- `DIST=0`: TRIG/ECHO Pin-Verbindung prüfen, Timeout erhöhen
- Unplausible Werte: Echos/Reflexionen in der Umgebung entfernen

### 2.3 DS18B20 Temperatur-Test (MICREEN-DTSMK)

#### Pinbelegung prüfen
- [ ] DATA → Pin D4
- [ ] VCC → 5V
- [ ] GND → GND
- [ ] 4,7kΩ Pull-up Widerstand zwischen DATA und VCC vorhanden

#### Funktionstest
1. Sensor in Raumtemperatur-Wasser tauchen (ca. 20°C)
2. Mit Referenzthermometer vergleichen

**Erwartete Ausgabe:**
```
TMP=20.3  (Temperatur in °C)
```

**Tests:**
- [ ] Raumtemperatur (~20°C): Messung ______ °C (Toleranz ±0,5°C)
- [ ] Kaltes Wasser (~10°C): Messung ______ °C (Toleranz ±0,5°C)
- [ ] Warmes Wasser (~30°C): Messung ______ °C (Toleranz ±0,5°C)

**🔧 Debugging:**
- `TMP=-127.0` oder `DISCONNECTED`: OneWire-Verkabelung prüfen, Pull-up prüfen
- Falsche Werte: Sensor-Adresse prüfen (`DallasTemperature::getAddress()`)

### 2.4 Analog-Sensoren (Trübung & TDS)

#### TSW-20M Trübungssensor

**Pinbelegung:**
- [ ] Signal → Pin A0
- [ ] VCC → 5V
- [ ] GND → GND

**Funktionstest:**
1. Sensor in klares Wasser tauchen
2. Sensor in trübes Wasser (z.B. mit Milch) tauchen

**Tests:**
- [ ] Klares Wasser: Wert ______ (0-200 = klar)
- [ ] Trübes Wasser: Wert ______ (600-1023 = trüb)
- [ ] Luft (kein Wasser): Wert ______ (sehr hoch oder 1023)

**Erwartete Ausgabe:**
```
TUR=150  (klar)
TUR=750  (trüb)
```

#### CQRSENTDS01 TDS-Sensor

**Pinbelegung:**
- [ ] Signal → Pin A1
- [ ] VCC → 5V
- [ ] GND → GND

**Funktionstest:**
1. Sensor in destilliertes Wasser tauchen (TDS ≈ 0 ppm)
2. Sensor in Leitungswasser tauchen (TDS ≈ 100-400 ppm)

**Tests:**
- [ ] Destilliertes Wasser: Wert ______ ppm (< 50 ppm)
- [ ] Leitungswasser: Wert ______ ppm (100-400 ppm)

**Erwartete Ausgabe:**
```
TDS=320  (Leitungswasser)
```

### 2.5 RS-485 Übertragung

#### Hardware-Setup
- [ ] RS485-Modul mit Sensor-Node verbunden
- [ ] DE/RE Pin → D5
- [ ] TX → D6 (SoftwareSerial TX)
- [ ] RX → D7 (SoftwareSerial RX)
- [ ] A und B Leitungen korrekt verdrahtet

#### Test mit Loopback (optional)
1. A und B Leitungen kurz verbinden (Loopback)
2. Serial Monitor beobachten

**Test mit Display-Node:**
- [ ] Sensor-Node sendet regelmäßig (siehe Serial Monitor)
- [ ] Display-Node empfängt Daten (Phase 3 prüfen)

**🔧 Debugging:**
- Keine Übertragung: DE/RE Pin prüfen (HIGH beim Senden)
- Verfälschte Daten: Baudrate prüfen (9600), GND-Verbindung prüfen
- Reichweite testen: Max. 1200m, bei längeren Strecken Terminierung prüfen

---

## 🏠 Phase 3: Display-Node Tests

### 3.1 RS-485 Empfang

#### Hardware-Setup
- [ ] Arduino Mega mit RS485-Modul verbunden
- [ ] Serial1 (RX1=D19, TX1=D18) für RS-485
- [ ] DE Pin (D10) auf LOW (Empfangsmodus)

#### Test
1. `DisplayNode.ino` auf Mega hochladen
2. USB Serial Monitor öffnen (115200 Baud)
3. Sensor-Node einschalten

**Erwartete Ausgabe (Serial Monitor):**
```
DisplayNode gestartet
Empfangen: <SENSOR;DIST=123.4;TMP=18.3;TUR=512;TDS=420>
Parsed: DIST=123.4, TMP=18.3, TUR=512, TDS=420
```

- [ ] ✅ Empfangsmeldung erscheint
- [ ] ✅ Nachrichten werden korrekt geparst
- [ ] ✅ Alle 4 Werte (DIST, TMP, TUR, TDS) erkannt

**🔧 Debugging:**
- Keine Nachrichten: RS-485 Verkabelung prüfen, A/B tauschen
- Verfälschte Zeichen: Baudrate prüfen, GND-Verbindung sicherstellen
- Nur Teildaten: Parser-Logik prüfen (`parser.cpp`)

### 3.2 Nextion Display Updates

#### Hardware-Setup
- [ ] Nextion Display mit Serial2 (TX2=D16, RX2=D17) verbunden
- [ ] Nextion Baudrate: 9600
- [ ] Nextion HMI hochgeladen (mit Komponenten: `tDist`, `tTemp`, `tTurb`, `tTDS`)

#### Test
1. Display einschalten
2. Sensor-Node Daten senden lassen
3. Display beobachten

**Erwartete Anzeige:**
- [ ] Textfeld `tDist` zeigt Distanz: "123.4 cm"
- [ ] Textfeld `tTemp` zeigt Temperatur: "18.3 °C"
- [ ] Textfeld `tTurb` zeigt Trübung: "512"
- [ ] Textfeld `tTDS` zeigt TDS: "420 ppm"

**Refresh-Rate:** Aktualisierung alle 5 Sekunden

**🔧 Debugging:**
- Display zeigt nichts: Baudrate prüfen, TX/RX Pins prüfen
- Nur Teildaten: Nextion-Komponentennamen prüfen (`config.h`)
- Flackern: Delays zwischen Nextion-Befehlen erhöhen (10-20ms)
- Display friert ein: Nextion Reset-Pin (falls vorhanden) prüfen

### 3.3 Serial Debugging

#### Vollständiger Debug-Output Beispiel:

```
DisplayNode gestartet
Empfangen: <SENSOR;DIST=123.4;TMP=18.3;TUR=512;TDS=420>
Parsed: DIST=123.4, TMP=18.3, TUR=512, TDS=420
Nextion: tDist.txt="123.4 cm"
Nextion: tTemp.txt="18.3 °C"
Nextion: tTurb.txt="512"
Nextion: tTDS.txt="420 ppm"
```

- [ ] Debug-Output vollständig und plausibel

---

## 🔗 Phase 4: Systemintegration

### 4.1 End-to-End Tests

#### Vollständiger Systemtest

**Setup:**
1. Sensor-Node im Testbehälter positionieren (mit Wasser)
2. Display-Node im "Haus" (separater Raum oder Tisch) aufstellen
3. CAT7-Kabel verlegen (min. 10m für realistischen Test)
4. Beide Nodes einschalten

**Tests:**
- [ ] **Füllstandsänderung:** Wasser im Behälter hinzufügen/entfernen
  - Display aktualisiert Distanzwert innerhalb 10 Sekunden
- [ ] **Temperaturänderung:** Warmes/kaltes Wasser hinzugeben
  - Display zeigt neue Temperatur innerhalb 10 Sekunden
- [ ] **Trübungsänderung:** Milch/Erde ins Wasser geben
  - Display zeigt höheren Trübungswert
- [ ] **TDS-Änderung:** Salz ins Wasser geben
  - Display zeigt höheren TDS-Wert

**✅ Pass-Kriterium:** Alle Sensorwerte werden korrekt übertragen und angezeigt

### 4.2 Langzeittest (24h)

#### Test-Protokoll
1. System für 24h durchlaufen lassen
2. Alle 2h Werte notieren (automatisches Logging empfohlen)
3. Nach 24h System prüfen

**Checkliste:**
- [ ] System läuft 24h ohne Neustart
- [ ] Keine Datenverluste (alle Nachrichten kommen an)
- [ ] Keine Speicherlecks (Serial Monitor bleibt stabil)
- [ ] Display zeigt durchgehend aktuelle Werte

**Langzeittest Tabelle:**

| Zeit | DIST (cm) | TMP (°C) | TUR | TDS (ppm) | Bemerkung |
|------|-----------|----------|-----|-----------|-----------|
| 0h   |           |          |     |           |           |
| 2h   |           |          |     |           |           |
| 4h   |           |          |     |           |           |
| 6h   |           |          |     |           |           |
| 8h   |           |          |     |           |           |
| 10h  |           |          |     |           |           |
| 12h  |           |          |     |           |           |
| 14h  |           |          |     |           |           |
| 16h  |           |          |     |           |           |
| 18h  |           |          |     |           |           |
| 20h  |           |          |     |           |           |
| 22h  |           |          |     |           |           |
| 24h  |           |          |     |           |           |

### 4.3 Störfestigkeit

#### Elektromagnetische Verträglichkeit (EMV)

**Tests:**
- [ ] **Netzteil Ein/Aus:** System neustart ohne Fehler
- [ ] **Handy in der Nähe:** Keine Störungen im Display
- [ ] **LED-Lampe neben Kabel:** Keine Datenverfälschung
- [ ] **Motor/Pumpe in der Nähe:** RS-485 bleibt stabil

#### Mechanische Tests
- [ ] **Kabel bewegen:** Verbindung bleibt stabil
- [ ] **Vibration am Sensor-Node:** Messwerte bleiben plausibel
- [ ] **Display antippen:** Nextion reagiert, Daten laufen weiter

---

## 📝 Testprotokoll-Vorlage

### Allgemeine Informationen

| **Feld** | **Wert** |
|----------|----------|
| **Datum:** | ________________ |
| **Tester:** | ________________ |
| **Firmware-Version Sensor-Node:** | ________________ |
| **Firmware-Version Display-Node:** | ________________ |
| **Hardware-Revision:** | ________________ |
| **Testumgebung:** | Labor / Feld (bitte ankreuzen) |

---

### Checkliste zum Ausdrucken

#### Phase 1: Hardware
- [ ] LM2596 Sensor-Node auf 5,00V eingestellt
- [ ] LM2596 Display-Node auf 5,00V eingestellt
- [ ] CAT7-Kabel durchgemessen (alle Adern < 0,2Ω/m)
- [ ] Polarität korrekt
- [ ] Isolation > 10MΩ

#### Phase 2: Sensor-Node
- [ ] Serial Debug funktioniert
- [ ] JSN-SR04T misst korrekt (±2 cm bei 50 cm)
- [ ] DS18B20 misst korrekt (±0,5°C)
- [ ] Trübungssensor reagiert auf Änderungen
- [ ] TDS-Sensor reagiert auf Änderungen
- [ ] RS-485 sendet Daten

#### Phase 3: Display-Node
- [ ] RS-485 empfängt Daten
- [ ] Parser extrahiert alle 4 Werte
- [ ] Nextion Display zeigt alle Werte
- [ ] Display aktualisiert sich regelmäßig

#### Phase 4: System
- [ ] End-to-End Test erfolgreich
- [ ] Langzeittest 24h bestanden
- [ ] Störfestigkeit geprüft

---

### Unterschriftsfeld

**Test abgeschlossen am:** ________________

**Tester (Name, Unterschrift):** ________________________________

**Freigabe (Name, Unterschrift):** ________________________________

**Bemerkungen:**
```
_________________________________________________________________

_________________________________________________________________

_________________________________________________________________
```

---

## 🔧 Debugging-Tipps

### Häufige Probleme und Lösungen

#### Problem: Sensor-Node startet nicht
**Symptome:** Keine LED-Aktivität, kein Serial Output  
**Lösungen:**
1. ✅ Spannungsversorgung prüfen (5V am Nano)
2. ✅ USB-Kabel tauschen
3. ✅ Bootloader prüfen (Nano Every manchmal zurücksetzen)

#### Problem: Display zeigt nur schwarzen Bildschirm
**Symptome:** Nextion Display bleibt schwarz  
**Lösungen:**
1. ✅ Spannungsversorgung prüfen (5V am Display)
2. ✅ Helligkeit im HMI erhöhen (`dim=100`)
3. ✅ HMI neu hochladen (über SD-Karte oder seriell)

#### Problem: Sensorwerte sind unplausibel
**Symptome:** DIST=0, TMP=-127, TUR=1023 konstant  
**Lösungen:**
1. ✅ Sensor-Verkabelung prüfen
2. ✅ Pull-up Widerstände prüfen (DS18B20)
3. ✅ Sensor austauschen (defekt?)

---

### RS-485 Troubleshooting

| **Problem** | **Mögliche Ursache** | **Lösung** |
|-------------|----------------------|------------|
| Keine Daten empfangen | A/B vertauscht | A und B Leitungen tauschen |
| Verfälschte Zeichen | Baudrate falsch | Beide Nodes auf 9600 Baud einstellen |
| Verbindungsabbrüche | Kabel zu lang | Terminierung 120Ω an beiden Enden |
| Sender kann nicht senden | DE/RE Pin falsch | DE Pin muss HIGH beim Senden |
| Nur Rauschen | Kein gemeinsamer GND | GND-Verbindung zwischen Nodes prüfen |

**RS-485 Debug-Code (Display-Node):**
```cpp
// In loop() hinzufügen für Diagnose:
if (RS485_SERIAL.available()) {
  char c = RS485_SERIAL.read();
  Serial.print(c, HEX); // Zeige Bytes in Hex
  Serial.print(" ");
}
```

---

### Nextion Probleme

#### Display reagiert nicht auf Befehle
**Debug-Schritte:**
1. Baudrate prüfen (Nextion und Arduino gleich)
2. TX/RX vertauscht? (Arduino TX → Nextion RX)
3. Nextion Debug-Modus aktivieren (im HMI Editor)
4. Komponenten-Namen überprüfen (`tDist` vs. `t_Dist`)

**Nextion Test-Befehl (manuell über Serial):**
```cpp
Serial2.print("tTemp.txt=\"TEST\"");
Serial2.write(0xFF); Serial2.write(0xFF); Serial2.write(0xFF);
```

#### Display zeigt alte Werte
**Ursache:** Nextion aktualisiert nicht schnell genug  
**Lösung:**
- Delay zwischen Updates erhöhen (min. 10ms)
- `page 0` Befehl senden, um Seite zu aktualisieren

---

### Sensor-Fehler

#### JSN-SR04T liefert nur 0 oder unendlich
**Checkliste:**
- [ ] TRIG/ECHO Pin-Nummern korrekt?
- [ ] Sensor richtig ausgerichtet? (keine schräge Montage)
- [ ] `pulseIn()` Timeout erhöhen (z.B. 30000 µs)
- [ ] Sensor in Wasser getaucht? (JSN-SR04T ist wasserdicht, aber nicht für dauerhaftes Untertauchen)

#### DS18B20 zeigt -127°C
**Checkliste:**
- [ ] 4,7kΩ Pull-up zwischen DATA und VCC?
- [ ] OneWire-Adresse korrekt? (mit `DallasTemperature::getAddress()` prüfen)
- [ ] Mehrere DS18B20? (Adressen eindeutig zuweisen)
- [ ] Kabel zu lang? (max. 20m ohne Repeater)

#### Analog-Sensoren zeigen konstante Werte
**Checkliste:**
- [ ] Sensor ins Wasser getaucht?
- [ ] VCC und GND korrekt angeschlossen?
- [ ] Analog-Pin korrekt? (A0, A1)
- [ ] ADC-Referenz korrekt? (`analogReference(DEFAULT)`)

---

## 🧮 Kalibrier-Anleitung

### Ultraschall (JSN-SR04T) kalibrieren

#### Ziel
Genaue Distanzmessung für Füllstandsberechnung.

#### Benötigte Materialien
- Maßband oder Lineal (min. 2m)
- Testbehälter mit bekannter Tiefe

#### Schritt-für-Schritt:

1. **Sensor montieren:** JSN-SR04T in bekannter Höhe über Wasseroberfläche montieren (z.B. 150 cm)
2. **Referenzmessung:** Mit Maßband tatsächliche Distanz messen: ______ cm
3. **Sensor-Messung:** Serial Monitor öffnen, DIST-Wert ablesen: ______ cm
4. **Offset berechnen:** `Offset = Referenz - Sensor`
5. **Offset eintragen:** In `config.h` oder `sensors.cpp`:
   ```cpp
   #define ULTRASONIC_OFFSET 2.5  // cm (Beispiel)
   // In measureDistance():
   float distance = duration * 0.034 / 2 + ULTRASONIC_OFFSET;
   ```

#### Kalibrier-Tabelle:

| Referenz (cm) | Sensor (cm) | Abweichung (cm) |
|---------------|-------------|-----------------|
| 20            |             |                 |
| 50            |             |                 |
| 100           |             |                 |
| 150           |             |                 |
| 200           |             |                 |

**Durchschnittlicher Offset:** ______ cm

---

### Temperatur-Offset (DS18B20)

#### Ziel
Kompensation von Sensor-Drift oder Gehäuse-Einflüssen.

#### Benötigte Materialien
- Referenzthermometer (kalibriert, ±0,1°C Genauigkeit)
- Wasserbad (stabile Temperatur)

#### Schritt-für-Schritt:

1. **Wasserbad vorbereiten:** Stabile Temperatur (z.B. 20°C) mit Referenzthermometer messen
2. **Sensor eintauchen:** 2 Minuten warten (Temperatur stabilisiert sich)
3. **Referenz ablesen:** ______ °C
4. **Sensor ablesen:** ______ °C
5. **Offset berechnen:** `Offset = Referenz - Sensor`
6. **Offset eintragen:** In `sensors.cpp`:
   ```cpp
   #define TEMP_OFFSET -0.3  // °C (Beispiel)
   // In readTemperature():
   float temp = sensors.getTempCByIndex(0) + TEMP_OFFSET;
   ```

#### Kalibrier-Tabelle:

| Referenz (°C) | Sensor (°C) | Abweichung (°C) |
|---------------|-------------|-----------------|
| 10            |             |                 |
| 20            |             |                 |
| 30            |             |                 |

**Durchschnittlicher Offset:** ______ °C

---

### TDS Kalibrierungsfaktor

#### Ziel
TDS-Sensor auf bekannte Lösungen kalibrieren (ppm).

#### Benötigte Materialien
- Destilliertes Wasser (TDS ≈ 0 ppm)
- Kalibrierungslösung 1413 µS/cm (≈ 707 ppm bei 25°C)
- Referenz-TDS-Meter (falls vorhanden)

#### Schritt-für-Schritt:

1. **Null-Punkt Kalibrierung:**
   - Sensor in destilliertes Wasser tauchen
   - ADC-Wert ablesen (Serial Monitor): ______ (Raw)
   - Sollwert: 0 ppm
   
2. **Hochpunkt Kalibrierung:**
   - Sensor in Kalibrierlösung (707 ppm) tauchen
   - ADC-Wert ablesen: ______ (Raw)
   - Sollwert: 707 ppm

3. **Faktor berechnen:**
   ```
   TDS (ppm) = (ADC_Raw - ADC_Null) * Faktor
   Faktor = 707 / (ADC_Hochpunkt - ADC_Null)
   ```

4. **Faktor eintragen:** In `sensors.cpp`:
   ```cpp
   #define TDS_FACTOR 0.5  // Beispiel
   int tds = (analogRead(TDS_PIN) - TDS_OFFSET) * TDS_FACTOR;
   ```

#### Kalibrier-Tabelle:

| Lösung | Soll (ppm) | ADC (Raw) | Sensor (ppm) | Abweichung |
|--------|------------|-----------|--------------|------------|
| Dest. Wasser | 0 | | | |
| Kalibrier-Lsg. | 707 | | | |
| Leitungswasser | (messen) | | | |

**Berechneter Faktor:** ______

---

## ✅ Zusammenfassung

Nach Abschluss aller Tests sollten folgende Punkte erfüllt sein:

✅ **Hardware:**
- Alle Spannungen stabil bei 5,00V ±0,05V
- Verkabelung korrekt und isoliert
- CAT7-Kabel durchgemessen

✅ **Sensor-Node:**
- Alle Sensoren liefern plausible Werte
- RS-485 sendet regelmäßig Daten
- Serial Debug zeigt korrekte Nachrichten

✅ **Display-Node:**
- RS-485 empfängt alle Daten
- Nextion Display zeigt alle Werte
- Display aktualisiert sich zuverlässig

✅ **System:**
- End-to-End Kommunikation funktioniert
- Langzeittest 24h bestanden
- System ist störfest

---

## 📚 Weitere Ressourcen

- [SensorNode README](../SensorNode/README.md)
- [DisplayNode README](../DisplayNode/README.md)
- [Komponenten-Liste](../Komponenten-Liste.md)
- [Spannungsversorgung](../Spannungsversorgung.md)

---

**Viel Erfolg beim Testen! 🎉**

Bei Fragen oder Problemen: [GitHub Issues](https://github.com/LukeArrow/BioSync/issues)
