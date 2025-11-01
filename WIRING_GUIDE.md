# 🔌 BioSync – Detaillierte Verdrahtungsanleitung

## 📋 Inhaltsverzeichnis
1. [Einleitung](#einleitung)
2. [Sensor-Node Verdrahtung](#sensor-node-verdrahtung-im-schacht)
3. [Display-Node Verdrahtung](#display-node-verdrahtung-im-haus)
4. [CAT7-Kabel Belegung](#cat7-kabel-belegung)
5. [Schritt-für-Schritt Anleitung](#schritt-für-schritt-anleitung)
6. [Sicherheitshinweise](#sicherheitshinweise)
7. [Troubleshooting](#troubleshooting)
8. [Verdrahtungsdiagramme](#verdrahtungsdiagramme)
9. [Materialliste](#materialliste-mit-mengenangaben)
10. [Wartung und Langzeitbetrieb](#wartung-und-langzeitbetrieb)

---

## Einleitung

### Systemübersicht

Das BioSync-System besteht aus zwei Hauptkomponenten:

**🌊 Sensor-Node (im Schacht):**
- Arduino Nano Every als Steuereinheit
- Vier Sensoren zur Messung von Füllstand, Temperatur, Trübung und TDS
- RS-485 Modul für störsichere Datenübertragung
- LM2596 Step-Down zur Spannungsversorgung
- Wasserdichtes IP67-Gehäuse

**🏠 Display-Node (im Haus):**
- Arduino Mega als zentrale Steuereinheit
- Nextion NX4024T032 Touchdisplay zur Visualisierung
- RS-485 Modul zum Datenempfang
- LM2596 Step-Down zur Spannungsversorgung
- 12V DC Netzteil als zentrale Stromquelle

Die Kommunikation erfolgt über ein robustes CAT7-Kabel, das sowohl Strom (12V) als auch Daten (RS-485) überträgt.

### Benötigte Werkzeuge

- ✅ Multimeter (für Spannungsmessung und Durchgangsprüfung)
- ✅ Lötkolben und Lötzinn (optional, für dauerhaften Aufbau)
- ✅ Abisolierzange
- ✅ Seitenschneider
- ✅ Schraubendreher-Set
- ✅ Isolierband oder Schrumpfschlauch
- ✅ Breadboard oder Lochrasterplatine
- ✅ Jumper-Kabel (male-male, male-female)
- ✅ Kabelschneider für CAT7-Kabel

### Benötigte Materialien

Siehe [Materialliste mit Mengenangaben](#materialliste-mit-mengenangaben) weiter unten für eine vollständige Auflistung.

### ⚠️ Vorab-Sicherheitshinweise

**Bevor Sie beginnen:**
- ⚡ Netzteil VOR jeder Arbeit vom Strom trennen
- 🔍 Polarität IMMER prüfen (+ und - nicht verwechseln)
- 🔧 Lose Kabelenden vermeiden (Kurzschlussgefahr)
- 💧 Sensor-Node muss in wasserdichtem IP67-Gehäuse verbaut werden
- 📐 Arbeiten Sie in einer trockenen, sauberen Umgebung

---

## Sensor-Node Verdrahtung (im Schacht)

### Komponenten-Übersicht

Die Sensor-Node verarbeitet Messwerte und sendet diese via RS-485 an die Display-Node im Haus.

### Detaillierte Pinbelegung

#### Arduino Nano Every – Zentrale Steuerung

| Arduino Pin | Verbindung zu | Funktion |
|------------|---------------|----------|
| VIN (5V) | LM2596 OUT+ | Spannungsversorgung (5V) |
| GND | LM2596 OUT- | Gemeinsame Masse |

#### JSN-SR04T – Ultraschallsensor (Füllstandsmessung)

| Sensor Pin | Arduino Pin | Hinweis |
|-----------|-------------|---------|
| VCC | 5V | Spannungsversorgung |
| GND | GND | Gemeinsame Masse |
| TRIG | D2 | Trigger-Signal |
| ECHO | D3 | Echo-Empfang |

**Besonderheit:** Wasserdichter Sensor mit separater Sonde, große Reichweite bis 4m.

#### DS18B20 / MICREEN-DTSMK – Temperatursensor

| Sensor Pin | Arduino Pin | Hinweis |
|-----------|-------------|---------|
| VCC (Rot) | 5V | Spannungsversorgung |
| GND (Schwarz) | GND | Gemeinsame Masse |
| DATA (Gelb) | D4 | 1-Wire Datenleitung |

**⚠️ WICHTIG:** Zwischen DATA (D4) und 5V muss ein **4,7 kΩ Pull-up Widerstand** geschaltet werden!

```
    5V ----[ 4,7kΩ ]---- D4 (DATA)
```

**Ohne Pull-up:** Sensor wird nicht erkannt oder liefert falsche Werte (z.B. -127°C).

#### TSW-20M – Trübungssensor (optisch)

| Sensor Pin | Arduino Pin | Hinweis |
|-----------|-------------|---------|
| VCC | 5V | Spannungsversorgung |
| GND | GND | Gemeinsame Masse |
| OUT | A0 | Analoger Messwert (0-1023) |

**Funktion:** Misst optische Trübung des Wassers. Je höher der Wert, desto trüber das Wasser.

#### CQRSENTDS01 – TDS-Sensor (Wasserqualität)

| Sensor Pin | Arduino Pin | Hinweis |
|-----------|-------------|---------|
| VCC | 5V | Spannungsversorgung |
| GND | GND | Gemeinsame Masse |
| OUT | A1 | Analoger Messwert (ppm) |

**Funktion:** Misst Total Dissolved Solids (TDS) – Gesamtmenge gelöster Stoffe im Wasser.

#### RS-485 Modul – Datenübertragung

| Modul Pin | Arduino Pin / Verbindung | Hinweis |
|----------|-------------------------|---------|
| VCC | 5V | Spannungsversorgung |
| GND | GND | Gemeinsame Masse |
| RO (Receiver Output) | D7 (RX) | SoftwareSerial Empfang |
| DI (Driver Input) | D6 (TX) | SoftwareSerial Senden |
| DE/RE | D5 | Sendekontrolle (HIGH = Senden, LOW = Empfangen) |
| A | CAT7 Ader 5 (Blau-Weiß) | RS-485 Datenleitung A |
| B | CAT7 Ader 6 (Grün) | RS-485 Datenleitung B |

**Hinweis:** Bei langen Kabelstrecken (>50m) einen 120Ω Abschlusswiderstand zwischen A und B schalten.

#### LM2596 Step-Down Modul – Spannungsversorgung

| Modul Pin | Verbindung | Hinweis |
|----------|------------|---------|
| IN+ | CAT7 12V+ (Adern 1+2) | 12V vom Haus |
| IN- | CAT7 GND (Adern 3+4) | Gemeinsame Masse |
| OUT+ | Arduino Nano VIN | Geregelte 5V Ausgabe |
| OUT- | GND (alle Sensoren) | Gemeinsame Masse |

**⚠️ KRITISCH:** LM2596 muss VOR dem ersten Anschluss auf **exakt 5,0V** eingestellt werden!
- Modul OHNE Last an 12V anschließen
- Multimeter an OUT+/OUT- anschließen
- Poti mit kleinem Schraubendreher drehen, bis Multimeter **5,00V** anzeigt
- Markierung auf Poti setzen

### Wichtige Verdrahtungshinweise für Sensor-Node

1. **Gemeinsame Masse:** Alle GND-Verbindungen (Nano, Sensoren, LM2596, RS-485) MÜSSEN zusammengeführt werden!
2. **Pull-up Widerstand:** DS18B20 funktioniert NICHT ohne 4,7kΩ Pull-up zwischen DATA und 5V
3. **Polarität:** Sensoren können bei Verpolung zerstört werden – VCC und GND nicht verwechseln!
4. **LM2596 Einstellung:** Ausgangsspannung VOR dem Anschluss an Nano/Sensoren prüfen!
5. **IP67-Gehäuse:** Alle Komponenten müssen wasserdicht verbaut werden
6. **Kabeldurchführungen:** Gehäusedurchführungen mit Kabelverschraubungen abdichten

---

## Display-Node Verdrahtung (im Haus)

### Komponenten-Übersicht

Die Display-Node empfängt Daten vom Sensor-Node und zeigt diese auf dem Nextion-Display an.

### Detaillierte Pinbelegung

#### Arduino Mega – Zentrale Steuerung

| Arduino Pin | Verbindung zu | Funktion |
|------------|---------------|----------|
| 5V | LM2596 OUT+ | Spannungsversorgung |
| GND | LM2596 OUT- | Gemeinsame Masse |
| D19 (RX1) | RS-485 RO | Hardware Serial 1 Empfang |
| D18 (TX1) | RS-485 DI | Hardware Serial 1 Senden |
| D17 (RX2) | Nextion TX | Hardware Serial 2 Empfang |
| D16 (TX2) | Nextion RX | Hardware Serial 2 Senden |
| D10 | RS-485 DE/RE (optional) | Sendekontrolle |

**Vorteil Hardware Serial:** Mega hat mehrere Hardware-Serial-Ports – stabiler als SoftwareSerial!

#### RS-485 Modul – Datenempfang

| Modul Pin | Arduino Pin / Verbindung | Hinweis |
|----------|-------------------------|---------|
| VCC | 5V | Spannungsversorgung |
| GND | GND | Gemeinsame Masse |
| RO | D19 (RX1) | Empfang vom Sensor-Node |
| DI | D18 (TX1) | Senden (optional, meist nur Empfang) |
| DE/RE | D10 oder GND | Empfangsmodus: dauerhaft auf GND oder per Pin gesteuert |
| A | CAT7 Ader 5 (Blau-Weiß) | RS-485 Datenleitung A |
| B | CAT7 Ader 6 (Grün) | RS-485 Datenleitung B |

**Hinweis:** Für reinen Empfangsbetrieb kann DE/RE dauerhaft auf GND gelegt werden.

#### Nextion NX4024T032 Display – Visualisierung

| Display Pin | Arduino Pin | Hinweis |
|------------|-------------|---------|
| VCC (5V) | 5V | Spannungsversorgung (ca. 85mA) |
| GND | GND | Gemeinsame Masse |
| TX | D17 (RX2) | Display sendet an Mega |
| RX | D16 (TX2) | Display empfängt von Mega |

**⚠️ WICHTIG:** TX vom Display geht zu RX am Mega und umgekehrt!

**Strombedarf:**
- Nextion Display: ca. 85mA
- Arduino Mega: ca. 50mA
- **Gesamt: ~135mA** (LM2596 sollte mindestens 200mA liefern können)

#### LM2596 Step-Down Modul – Spannungsversorgung

| Modul Pin | Verbindung | Hinweis |
|----------|------------|---------|
| IN+ | 12V Netzteil + | 12V Einspeisung |
| IN- | 12V Netzteil - | Gemeinsame Masse |
| OUT+ | Mega 5V + Nextion 5V | Geregelte 5V Ausgabe |
| OUT- | GND (gemeinsam) | Gemeinsame Masse |

**⚠️ KRITISCH:** LM2596 VOR dem Anschluss auf **exakt 5,0V** einstellen (siehe Anleitung oben).

#### 12V DC Netzteil – Zentrale Stromversorgung

| Netzteil | Verbindung | Hinweis |
|---------|------------|---------|
| 12V+ | LM2596 IN+ UND CAT7 Ader 1+2 | Versorgt beide Nodes |
| 12V- | LM2596 IN- UND CAT7 Ader 3+4 | Gemeinsame Masse |

**Empfehlung:** Netzteil mit mindestens 2A Strombelastbarkeit (Reserve für Erweiterungen).

### Wichtige Verdrahtungshinweise für Display-Node

1. **Hardware Serial nutzen:** TX1/RX1 für RS-485, TX2/RX2 für Nextion – nicht TX/RX (USB-Port)!
2. **TX/RX nicht vertauschen:** TX vom Display geht zu RX am Mega und umgekehrt
3. **Stromversorgung:** LM2596 muss mindestens 200mA liefern können
4. **Gemeinsame Masse:** Alle GND-Anschlüsse zusammenführen
5. **12V Aufteilung:** Netzteil versorgt Display-Node UND Sensor-Node über CAT7

---

## CAT7-Kabel Belegung

### Ader-Zuordnung

Das CAT7-Kabel verbindet Display-Node (Haus) mit Sensor-Node (Schacht). Es überträgt sowohl Stromversorgung (12V) als auch Daten (RS-485).

| Ader | Farbe | Funktion | Hinweis |
|------|-------|----------|---------|
| 1 | Orange-Weiß | 12V+ | Stromversorgung (parallel mit Ader 2) |
| 2 | Orange | 12V+ | Stromversorgung (parallel mit Ader 1) |
| 3 | Grün-Weiß | GND | Gemeinsame Masse (parallel mit Ader 4) |
| 4 | Blau | GND | Gemeinsame Masse (parallel mit Ader 3) |
| 5 | Blau-Weiß | RS-485 A | Datenleitung (verdrillt mit Ader 6) |
| 6 | Grün | RS-485 B | Datenleitung (verdrillt mit Ader 5) |
| 7 | Braun-Weiß | Reserve | Für zukünftige Erweiterungen |
| 8 | Braun | Reserve | Für zukünftige Erweiterungen |

### Wichtige Hinweise zur Kabelverlegung

**Parallel geschaltete Adern:**
- Adern 1+2 für 12V+ → verdoppelt die Strombelastbarkeit
- Adern 3+4 für GND → reduziert Spannungsabfall bei langen Strecken

**Verdrillte Paare für RS-485:**
- Nutzen Sie ein verdrilltes Aderpaar (5+6) für RS-485 A und B
- Minimiert elektromagnetische Störungen (EMI)
- Wichtig bei längeren Kabelstrecken

**Abschlusswiderstand (bei Strecken >50m):**
- 120Ω Widerstand zwischen RS-485 A und B schalten
- An beiden Enden (Sensor-Node UND Display-Node)
- Reduziert Signalreflexionen

**Kabelenden vorbereiten:**
1. Kabel sauber abisolieren (ca. 5mm)
2. Adern verzinnen (verhindert Oxidation)
3. Enden beschriften (Haus/Schacht, Funktion)
4. In Klemmen/Lüsterklemmen fixieren

### Verbindung im Haus (Display-Node)

```
12V Netzteil:
  12V+ ──┬── LM2596 IN+
         └── CAT7 Ader 1+2
  
  12V- ──┬── LM2596 IN-
         └── CAT7 Ader 3+4

RS-485 Modul:
  A ── CAT7 Ader 5
  B ── CAT7 Ader 6
```

### Verbindung im Schacht (Sensor-Node)

```
CAT7 Ader 1+2 ── LM2596 IN+
CAT7 Ader 3+4 ── LM2596 IN-

CAT7 Ader 5 ── RS-485 A
CAT7 Ader 6 ── RS-485 B
```

---

## Schritt-für-Schritt Anleitung

### Schritt 1: LM2596 Module vorbereiten

**Beide LM2596 (Sensor-Node UND Display-Node) müssen kalibriert werden!**

1. ⚡ LM2596 Modul OHNE Last (ohne Arduino/Sensoren) an 12V Quelle anschließen
2. 🔌 IN+ an 12V+, IN- an 12V- (Netzteil oder Labornetzteil)
3. 📏 Multimeter auf DC Voltage stellen
4. 🔴 Rote Messspitze an OUT+, schwarze Messspitze an OUT-
5. 🔧 Kleinen Schraubendreher nehmen und Poti vorsichtig drehen
6. ✅ Drehen bis Multimeter **exakt 5,00V** anzeigt
7. ✏️ Markierung auf Poti setzen (z.B. mit Lackstift)
8. 🔌 Netzteil trennen und Modul beschriften ("Sensor" / "Display")

**⚠️ Warnung:** Nie LM2596 ohne Last auf >5V einstellen – könnte nachfolgende Komponenten zerstören!

### Schritt 2: Sensor-Node aufbauen

**Aufbau auf Breadboard oder Lochrasterplatine:**

1. 🔹 Arduino Nano Every auf Breadboard/Platine stecken
2. 🔹 LM2596 mit Nano verbinden:
   - LM2596 OUT+ → Nano VIN
   - LM2596 OUT- → Nano GND
3. 🔹 **Test:** LM2596 IN an 12V anschließen → Nano sollte Power-LED zeigen
4. 🔹 Sensoren nacheinander anschließen und testen:
   - **JSN-SR04T:** VCC→5V, GND→GND, TRIG→D2, ECHO→D3
   - **DS18B20:** VCC→5V, GND→GND, DATA→D4 + **4,7kΩ Pull-up zu 5V**
   - **TSW-20M:** VCC→5V, GND→GND, OUT→A0
   - **CQRSENTDS01:** VCC→5V, GND→GND, OUT→A1
5. 🔹 Sketch hochladen und jeden Sensor einzeln testen (Serieller Monitor)
6. 🔹 RS-485 Modul zuletzt anschließen:
   - VCC→5V, GND→GND
   - RO→D7, DI→D6, DE/RE→D5
7. 🔹 Alle Komponenten in IP67-Gehäuse einbauen
8. 🔹 Kabelverschraubungen für CAT7-Kabel verwenden (wasserdicht!)

**Tipp:** Vor dem Einbau ins Gehäuse alle Verbindungen durchmessen und Software testen!

### Schritt 3: Display-Node aufbauen

**Aufbau auf Breadboard oder in Gehäuse:**

1. 🔸 Arduino Mega auf Breadboard/Gehäuse montieren
2. 🔸 LM2596 an 12V Netzteil anschließen (noch NICHT einstecken!)
3. 🔸 LM2596 mit Mega und Nextion verbinden:
   - LM2596 OUT+ → Mega 5V UND Nextion VCC
   - LM2596 OUT- → Mega GND UND Nextion GND
4. 🔸 **Test:** Netzteil einstecken → Mega und Nextion sollten starten
5. 🔸 RS-485 Modul anschließen:
   - VCC→5V, GND→GND
   - RO→D19 (RX1), DI→D18 (TX1), DE/RE→GND (Empfangsmodus)
6. 🔸 Nextion Display verbinden:
   - Display TX → Mega D17 (RX2)
   - Display RX → Mega D16 (TX2)
7. 🔸 Sketch hochladen und Nextion-Kommunikation testen
8. 🔸 Display-Node in Gehäuse montieren (optional)

**Tipp:** Hardware Serial nutzen (TX1/RX1, TX2/RX2) – stabiler als SoftwareSerial!

### Schritt 4: CAT7-Kabel verlegen

**Verbindung zwischen Haus und Schacht:**

1. 🔶 CAT7-Kabel vom Haus zum Schacht verlegen
   - Kabel schützen (Rohr, Kabelkanal, unterirdisch verlegen)
   - Kabelwege planen (Zugentlastung, keine scharfen Knicke)
2. 🔶 Beide Enden ca. 10cm abisolieren
3. 🔶 Einzelne Adern abisolieren (ca. 5mm), verzinnen und beschriften
4. 🔶 **Im Haus (Display-Node):**
   - Adern 1+2 (12V+) an Netzteil + UND LM2596 IN+ (parallel)
   - Adern 3+4 (GND) an Netzteil - UND LM2596 IN- (parallel)
   - Ader 5 (Blau-Weiß) an RS-485 A
   - Ader 6 (Grün) an RS-485 B
5. 🔶 **Im Schacht (Sensor-Node):**
   - Adern 1+2 (12V+) an LM2596 IN+
   - Adern 3+4 (GND) an LM2596 IN-
   - Ader 5 (Blau-Weiß) an RS-485 A
   - Ader 6 (Grün) an RS-485 B
6. 🔶 Verbindung mit Multimeter durchmessen (Durchgang, keine Kurzschlüsse)

**Sicherheitscheck:**
- ✅ Polarität korrekt? (12V+ und GND nicht vertauscht)
- ✅ Keine Kurzschlüsse zwischen Adern?
- ✅ RS-485 A mit A und B mit B verbunden?
- ✅ Kabeldurchführungen wasserdicht?

### Schritt 5: Systemtest durchführen

1. 🧪 Netzteil im Haus einstecken
2. 🧪 Sensor-Node sollte starten (evtl. LED auf Nano prüfen)
3. 🧪 Display-Node sollte starten und Display aktivieren
4. 🧪 Nach kurzer Initialisierung sollten Sensordaten auf Display erscheinen
5. 🧪 Alle Werte plausibel? (Temperatur ~15-25°C, Füllstand realistisch)
6. 🧪 Bei Problemen: Siehe [Troubleshooting](#troubleshooting)

**Detaillierte Tests:**
- Serieller Monitor am Sensor-Node: Werden Daten gesendet?
- Serieller Monitor am Display-Node: Werden Daten empfangen?
- Nextion-Komponenten aktualisieren sich?
- Sensoren liefern plausible Werte?

**Verweis:** Ausführliche Test-Szenarien in separater Testing-Dokumentation (falls vorhanden).

---

## Sicherheitshinweise

### ⚠️ Allgemeine Sicherheit

**VOR jeder Arbeit beachten:**

- ⚡ **Netzteil IMMER vom Strom trennen** bevor Änderungen vorgenommen werden
- 🔍 **Polarität IMMER prüfen** – Verpolung kann Komponenten sofort zerstören
- 🔧 **Keine losen Kabelenden** – Kurzschlussgefahr!
- 💧 **IP67-Gehäuse im Schacht MUSS wasserdicht sein**
- 📏 **LM2596 nie ohne Last auf >5V einstellen**
- 🔌 **Netzteil mit Überlastschutz verwenden**

### ⚡ Überspannungsschutz

**Empfohlene Schutzmaßnahmen:**

1. **TVS-Dioden (Transient Voltage Suppressor):**
   - Z.B. P6KE18A zwischen 12V+ und GND
   - Schützt vor Spannungsspitzen (Blitzeinschlag, Schaltspitzen)
   - Am Netzteilausgang und am Sensor-Node Eingang

2. **Sicherungen:**
   - 1A Feinsicherung im 12V+ Pfad (zwischen Netzteil und System)
   - Schützt vor Überstrom und Kurzschluss

3. **Erdung:**
   - Netzteil-Gehäuse erden
   - Metallgehäuse der Nodes erden (falls vorhanden)

### 🔒 Spezielle Hinweise

**LM2596 Einstellung:**
- ⚠️ Ausgangsspannung VOR dem Anschluss an Komponenten einstellen
- ⚠️ Nie >5,3V einstellen – Arduino/Sensoren können beschädigt werden
- ⚠️ Multimeter nutzen – nicht auf Anzeige-LED des Moduls verlassen

**Sensor-Node im Schacht:**
- 💧 IP67-Gehäuse muss dicht sein – regelmäßig prüfen
- 💧 Kabeldurchführungen mit Kabelverschraubungen abdichten
- 💧 Silica-Gel Beutel im Gehäuse (absorbiert Restfeuchtigkeit)
- 🌡️ Temperaturbereich beachten (-10°C bis +50°C für die meisten Komponenten)

**RS-485 Verkabelung:**
- 🔌 A mit A und B mit B verbinden – NICHT vertauschen!
- 🔌 Bei langen Strecken (>50m): 120Ω Abschlusswiderstand
- 🔌 Verdrillte Kabelpaare für RS-485 nutzen (reduziert Störungen)

**Nextion Display:**
- 📺 Display nicht mit >5,3V versorgen
- 📺 TX/RX nicht vertauschen (zerstört Display meist nicht, funktioniert aber nicht)
- 📺 Bei falscher Darstellung: Baudrate prüfen (Standard: 9600)

### 🚨 Notfallmaßnahmen

**Bei Rauchentwicklung oder Brandgeruch:**
1. Sofort Netzteil vom Strom trennen
2. Betroffene Komponente identifizieren
3. Kurzschluss oder Verpolung prüfen
4. Beschädigte Komponente ersetzen

**Bei Fehlfunktion:**
1. Netzteil trennen
2. Alle Verbindungen visuell prüfen
3. Mit Multimeter Spannungen messen
4. Siehe [Troubleshooting](#troubleshooting)

---

## Troubleshooting

### Häufige Probleme und Lösungen

| Problem | Mögliche Ursache | Lösung |
|---------|------------------|--------|
| **Nano startet nicht** | Keine 5V Versorgung | LM2596 Ausgangsspannung mit Multimeter prüfen. Verkabelung zwischen LM2596 und Nano checken. |
| **Sensor liefert keine Werte** | Falsche Verkabelung oder defekter Sensor | Pin-Belegung prüfen (VCC, GND, Signal-Pins). GND-Verbindung checken (gemeinsame Masse!). Sensor einzeln testen. |
| **Display bleibt schwarz** | Zu wenig Strom oder falsche Verkabelung | LM2596 Strombegrenzung prüfen (min. 200mA). VCC/GND Verbindung checken. Separate Versorgung testen. |
| **Display zeigt nur weißen Bildschirm** | Display nicht initialisiert oder TX/RX vertauscht | TX/RX Verbindung prüfen (Display TX → Mega RX2 und umgekehrt). Baudrate prüfen. |
| **RS-485 keine Daten** | A/B vertauscht oder DE/RE falsch | A mit A und B mit B verbinden. DE/RE Pin Steuerung prüfen (HIGH=Senden, LOW=Empfangen). Gemeinsame Masse prüfen. |
| **RS-485 Datenmüll** | Baudrate falsch oder Störungen | Baudrate auf beiden Seiten identisch? (9600). Abschlusswiderstand 120Ω bei langen Strecken. Kabel auf Beschädigung prüfen. |
| **Ultraschall misst 0 oder Timeout** | Echo-Pin defekt oder falsche Verkabelung | TRIG/ECHO Pins prüfen (nicht vertauscht?). Timeout-Wert im Code erhöhen. Sensor auf Verschmutzung prüfen. Sensor tauschen. |
| **DS18B20 liefert -127°C oder DISCONNECTED** | Pull-up Widerstand fehlt oder falsche Verkabelung | 4,7kΩ Pull-up zwischen DATA (D4) und 5V einbauen. VCC/GND/DATA Verkabelung prüfen. Sensor auf Kurzschluss prüfen. |
| **TDS/Trübung liefert konstant 0 oder 1023** | Sensor nicht im Wasser oder defekt | Sensor in Wasser tauchen. Analoger Pin korrekt verbunden? (A0/A1). Sensor tauschen. |
| **Mega startet ständig neu (Bootloop)** | Stromversorgung zu schwach | LM2596 auf Überlast prüfen. Stromverbrauch messen. Separate Versorgung für Display/Mega testen. |
| **Nextion reagiert nicht auf Befehle** | Falsche Baudrate oder TX/RX vertauscht | Baudrate im Code und Nextion HMI identisch? (Standard: 9600). TX/RX Verbindung prüfen. Nextion neu starten. |
| **Werte auf Display werden nicht aktualisiert** | Parser erkennt Format nicht | Nachrichtenformat vom Sensor-Node prüfen. Seriellen Monitor nutzen um eingehende Daten zu sehen. Komponenten-Namen im HMI prüfen (tDist, tTemp, etc.). |
| **System funktioniert nur manchmal** | Wackelkontakt oder Störungen | Alle Verbindungen auf festen Sitz prüfen. Lötverbindungen verwenden statt Steckverbindungen. CAT7-Kabel auf Beschädigung prüfen. |
| **Sensor-Node offline nach einiger Zeit** | Feuchtigkeit im Gehäuse | IP67-Gehäuse auf Dichtigkeit prüfen. Silica-Gel Beutel einlegen. Kondensation vermeiden. |

### Systematische Fehlersuche

**Schritt 1: Stromversorgung prüfen**
1. Multimeter verwenden
2. 12V am Netzteil-Ausgang messen
3. 12V am CAT7-Kabel (Haus) messen
4. 12V am CAT7-Kabel (Schacht) messen → Spannungsabfall?
5. 5V am LM2596-Ausgang (beide Nodes) messen
6. 5V am Arduino VIN/5V-Pin messen

**Schritt 2: Kommunikation prüfen**
1. Seriellen Monitor am Sensor-Node öffnen (USB)
2. Werden Sensordaten im Terminal ausgegeben?
3. Seriellen Monitor am Display-Node öffnen (USB)
4. Werden RS-485 Nachrichten empfangen?
5. Format korrekt? `<SENSOR;DIST=...;TMP=...;...>`

**Schritt 3: Sensoren einzeln testen**
1. Einen Sensor nach dem anderen anschließen
2. Testprogramm für einzelnen Sensor hochladen
3. Werte im Seriellen Monitor beobachten
4. Defekten Sensor identifizieren und tauschen

**Schritt 4: RS-485 Bus isolieren**
1. Sensor-Node vom CAT7-Kabel trennen
2. RS-485 Module direkt mit Jumperkabel verbinden (kurze Strecke)
3. Funktioniert Kommunikation? → CAT7-Kabel prüfen
4. Funktioniert nicht? → RS-485 Module oder Firmware prüfen

### Debug-Tipps

**Serieller Monitor nutzen:**
- Sensor-Node: Debug-Ausgaben der Messwerte
- Display-Node: Empfangene RS-485 Nachrichten ausgeben
- Baudrate: 115200 für Debug, 9600 für RS-485/Nextion

**LED-Indikatoren:**
- Power-LED auf Arduino → Stromversorgung OK
- TX/RX LEDs auf RS-485 Modul → Datenfluss sichtbar
- Blinkende LED im Code als Lebenszeichen

**Multimeter ist dein Freund:**
- Spannungen an allen kritischen Punkten messen
- Durchgang prüfen (Kurzschlüsse ausschließen)
- Widerstandswerte prüfen (Pull-up, Abschlusswiderstand)

---

## Verdrahtungsdiagramme

### Sensor-Node Blockdiagramm

```
    ╔══════════════════════════════════════════════════════════╗
    ║                  SENSOR-NODE (im Schacht)                ║
    ╚══════════════════════════════════════════════════════════╝
    
         12V vom Haus (CAT7 Ader 1+2)
                      │
                 ┌────▼────┐
                 │ LM2596  │ Step-Down (12V → 5V)
                 │ 5V OUT  │
                 └────┬────┘
                      │ 5V
        ┬─────────────┼─────────────┬─────────────┬─────────────┬
        │             │             │             │             │
   ┌────▼─────┐  ┌───▼────┐  ┌─────▼─────┐  ┌───▼────┐  ┌─────▼─────┐
   │  Arduino │  │ JSN-   │  │ DS18B20   │  │ TSW-   │  │ CQRSENT-  │
   │  Nano    │  │ SR04T  │  │ (Temp)    │  │ 20M    │  │ DS01      │
   │  Every   │  │ (Ultra)│  │ + 4,7kΩ   │  │ (Trüb) │  │ (TDS)     │
   │          │  │        │  │  Pull-up  │  │        │  │           │
   │ D2←TRIG  │  │        │  │           │  │        │  │           │
   │ D3←ECHO  │  │        │  │           │  │        │  │           │
   │ D4←DATA  │  │        │  │           │  │        │  │           │
   │ A0←OUT   │  │        │  │           │  │        │  │           │
   │ A1←OUT   │  │        │  │           │  │        │  │           │
   └────┬─────┘  └────────┘  └───────────┘  └────────┘  └───────────┘
        │
        │ D6(TX), D7(RX), D5(DE)
        ▼
   ┌─────────────┐
   │  RS-485     │
   │  Modul      │
   │             │
   │  A ◄────────┼─── CAT7 Ader 5 (Blau-Weiß)
   │  B ◄────────┼─── CAT7 Ader 6 (Grün)
   └─────────────┘
        │
       GND (gemeinsam, CAT7 Ader 3+4)
```

### Display-Node Blockdiagramm

```
    ╔══════════════════════════════════════════════════════════╗
    ║                 DISPLAY-NODE (im Haus)                   ║
    ╚══════════════════════════════════════════════════════════╝
    
         12V Netzteil
         │
         ├─────────────────┐
         │                 │
         │                 │
         ▼                 ▼
    ┌─────────┐       ┌────────────┐
    │ LM2596  │       │ CAT7 Kabel │ ──► Sensor-Node
    │ 5V OUT  │       │ 12V (1+2)  │
    └────┬────┘       │ GND (3+4)  │
         │ 5V         │ RS485 (5+6)│
         │            └──────┬─────┘
         │                   │
    ┬────┴────┬──────────────┘
    │         │         
    │         │         
    ▼         ▼         
┌────────┐  ┌──────────────┐
│Arduino │  │  RS-485      │
│ Mega   │  │  Modul       │
│        │  │              │
│RX1(D19)◄──┤ RO           │
│TX1(D18)├─►│ DI           │
│        │  │ DE/RE → GND  │
└───┬────┘  └──────────────┘
    │
    │ TX2(D16), RX2(D17)
    ▼
┌────────────────┐
│  Nextion       │
│  NX4024T032    │
│  3.2" Display  │
│                │
│  RX ◄── TX2    │
│  TX ──► RX2    │
└────────────────┘
    │
   GND (gemeinsam)
```

### Vollständiges System-Diagramm

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                         BioSync GESAMT-SYSTEM                                 ║
╚═══════════════════════════════════════════════════════════════════════════════╝

    🏠 HAUS                         📡 CAT7 KABEL                   🌊 SCHACHT
                                    (bis 100m)
                                                            
┌─────────────────┐                                        ┌─────────────────┐
│  12V Netzteil   │                                        │ Sensor-Node     │
│  (2A)           │                                        │                 │
│                 │                                        │ Arduino Nano    │
│  ├─ 12V+ ───────┼────[Ader 1+2]─────────────────────────┤ ← LM2596 ← 12V  │
│  └─ GND  ───────┼────[Ader 3+4]─────────────────────────┤ ← LM2596 GND    │
└─────────────────┘    [Ader 5] RS-485 A                  │                 │
                       [Ader 6] RS-485 B                   │ Sensoren:       │
                       [Ader 7+8] Reserve                  │  • JSN-SR04T    │
                                                            │  • DS18B20      │
┌─────────────────┐                                        │  • TSW-20M      │
│ Display-Node    │                                        │  • CQRSENTDS01  │
│                 │                                        │                 │
│ Arduino Mega    │◄───[Ader 5+6]───RS-485────────────────┤ RS-485 Modul    │
│ + LM2596        │                                        │                 │
│                 │                                        │ IP67 Gehäuse    │
│ Nextion Display │                                        └─────────────────┘
│ (3.2" Touch)    │
└─────────────────┘
```

---

## Materialliste mit Mengenangaben

### Elektronik-Komponenten

| Komponente | Menge | Einsatzort | Bemerkung |
|------------|-------|------------|-----------|
| Arduino Nano Every | 1x | Sensor-Node | Kompakte Steuereinheit |
| Arduino Mega | 1x | Display-Node | Mehrere Hardware-Serial-Ports |
| JSN-SR04T | 1x | Sensor-Node | Wasserdichter Ultraschallsensor |
| DS18B20 / MICREEN-DTSMK | 1x | Sensor-Node | Edelstahl-Temperatursonde |
| TSW-20M | 1x | Sensor-Node | Optischer Trübungssensor |
| CQRSENTDS01 | 1x | Sensor-Node | TDS-Sensor für Wasserqualität |
| RS-485 Modul (z.B. MAX485) | 2x | Beide Nodes | Für störsichere Übertragung |
| LM2596 Step-Down Modul | 2x | Beide Nodes | Spannungsregler 12V→5V |
| Nextion NX4024T032 | 1x | Display-Node | 3.2" Touchdisplay |
| 12V/2A DC Netzteil | 1x | Display-Node | Zentrale Stromversorgung |

### Passive Bauteile

| Komponente | Menge | Einsatzort | Bemerkung |
|------------|-------|------------|-----------|
| 4,7 kΩ Widerstand (1/4W) | 1x | Sensor-Node | Pull-up für DS18B20 |
| 120Ω Widerstand (1/4W) | 2x | Beide Nodes (optional) | RS-485 Abschluss (bei >50m) |
| TVS-Diode P6KE18A | 2x | Beide Nodes (optional) | Überspannungsschutz |
| 1A Feinsicherung + Halter | 1x | Display-Node (optional) | Überstromschutz |

### Verkabelung & Montage

| Komponente | Menge | Einsatzort | Bemerkung |
|------------|-------|------------|-----------|
| CAT7 Kabel | Nach Bedarf | Verbindung Haus-Schacht | Mindestens benötigte Länge + 2m Reserve |
| Jumper-Kabel male-male | 20x | Breadboard-Aufbau | Für Prototyping |
| Jumper-Kabel male-female | 10x | Sensoren → Arduino | Wenn Sensoren Buchsen haben |
| Breadboard (mittel) | 2x | Beide Nodes (optional) | Für lötfreien Aufbau |
| Lochrasterplatine | 2x | Beide Nodes (optional) | Für permanenten Aufbau |
| Lüsterklemmen / Wago-Klemmen | 10x | CAT7-Anschluss | Für sichere Kabelverbindungen |

### Gehäuse & Schutz

| Komponente | Menge | Einsatzort | Bemerkung |
|------------|-------|------------|-----------|
| IP67 Gehäuse (ca. 15x10x5cm) | 1x | Sensor-Node | Wasserdicht, mit Kabelverschraubungen |
| Kabelverschraubung M12 | 2x | Sensor-Node Gehäuse | Für CAT7 + Sensorkabel |
| Gehäuse für Display-Node | 1x | Display-Node (optional) | Z.B. Kunststoff-Instrumentengehäuse |
| Silica-Gel Beutel | 2x | Sensor-Node Gehäuse | Feuchtigkeitsabsorber |
| Schrumpfschlauch (Sortiment) | 1 Set | Alle Verbindungen | Für Isolation |

### Werkzeuge (falls nicht vorhanden)

| Werkzeug | Menge | Bemerkung |
|----------|-------|-----------|
| Multimeter | 1x | Unverzichtbar für Spannungsmessung |
| Lötkolben + Lötzinn | 1 Set | Für permanente Verbindungen |
| Abisolierzange | 1x | Zum Abisolieren von Kabeln |
| Seitenschneider | 1x | Zum Kürzen von Kabeln |
| Schraubendreher-Set | 1 Set | Kreuz, Schlitz, kleine Größen |
| Kabelschneider | 1x | Für dicke Kabel (CAT7) |

### Geschätzte Gesamtkosten

| Kategorie | Geschätzte Kosten (EUR) |
|-----------|------------------------|
| Arduino-Boards | 30-40 |
| Sensoren | 40-60 |
| Display | 15-25 |
| RS-485 Module | 5-10 |
| LM2596 Module | 5-8 |
| Netzteil | 10-15 |
| CAT7 Kabel (50m) | 30-50 |
| Gehäuse & Kabel | 20-30 |
| Kleinteile & Werkzeug | 20-30 |
| **GESAMT** | **175-268 EUR** |

**Hinweis:** Preise können je nach Anbieter und Region variieren. Aliexpress/China-Importe sind günstiger, haben aber längere Lieferzeiten.

---

## Wartung und Langzeitbetrieb

### Regelmäßige Wartung

**Alle 3 Monate:**

1. ✅ **Sichtprüfung Sensor-Node:**
   - IP67-Gehäuse auf Risse und Beschädigungen prüfen
   - Kabeldurchführungen auf Dichtigkeit prüfen
   - Innen auf Kondenswasser/Feuchtigkeit prüfen
   - Silica-Gel Beutel austauschen (wenn farblich verändert)

2. ✅ **Sensoren reinigen:**
   - JSN-SR04T: Ultraschall-Membran vorsichtig mit destilliertem Wasser reinigen
   - DS18B20: Edelstahlsonde von Ablagerungen befreien
   - TSW-20M: Optische Fenster mit weichem Tuch reinigen
   - CQRSENTDS01: Elektroden reinigen oder kalibrieren

3. ✅ **Elektrische Kontakte prüfen:**
   - Steckverbindungen auf festen Sitz prüfen
   - Korrosion an Kontakten überprüfen (ggf. Kontaktspray)
   - Lötverbindungen auf Risse prüfen
   - CAT7-Kabel-Anschlüsse nachziehen

4. ✅ **Funktionstest:**
   - Display zeigt aktuelle Werte an?
   - Alle Sensorwerte plausibel?
   - RS-485 Kommunikation stabil?
   - Keine Fehlermeldungen im System?

**Jährlich:**

1. 🔧 **Tiefergehende Inspektion:**
   - Alle Schraubverbindungen nachziehen
   - LM2596 Ausgangsspannung nachmessen (sollte 5,00V sein)
   - CAT7-Kabel auf Beschädigungen prüfen (visuell)
   - Netzteil auf Überlastungszeichen prüfen (Hitze, Geruch)

2. 🔧 **Software-Update:**
   - Firmware auf Updates prüfen
   - Sensorkalibrierung durchführen
   - Log-Dateien auswerten (falls implementiert)

3. 🔧 **Austausch Verschleißteile:**
   - Silica-Gel Beutel erneuern
   - Stark korrodierte Kabel/Stecker ersetzen
   - Dichtungen bei Bedarf erneuern

### Fehlerprotokoll führen

**Dokumentation bei jedem Ausfall:**
- Datum und Uhrzeit
- Welcher Fehler ist aufgetreten?
- Welche Werte waren betroffen?
- Welche Maßnahme wurde ergriffen?
- Wurde ein Bauteil getauscht?

**Vorteil:** Muster erkennen (z.B. immer gleicher Sensor fällt aus → Konstruktionsfehler).

### Ersatzteile bevorraten

**Empfohlene Ersatzteile:**

| Komponente | Anzahl | Begründung |
|------------|--------|------------|
| RS-485 Modul | 2x | Häufigster Ausfallgrund (Überspannung) |
| LM2596 Modul | 1x | Kann bei Verpolung durchbrennen |
| JSN-SR04T | 1x | Mechanischer Verschleiß möglich |
| DS18B20 | 1x | Kann durch Kurzschluss ausfallen |
| Jumper-Kabel | 10x | Brechen bei häufiger Wartung |
| Sicherungen 1A | 5x | Verbrauchsmaterial |
| Silica-Gel Beutel | 5x | Regelmäßiger Austausch nötig |

**Kostenaufwand Ersatzteile:** ~30-50 EUR (einmalig)

### Langzeitbetrieb-Tipps

**Stabilität erhöhen:**

1. **Lötverbindungen statt Steckverbindungen:**
   - Breadboard nur für Prototyping
   - Für Dauerbetrieb: Lochrasterplatine mit gelöteten Verbindungen
   - Reduziert Wackelkontakte und Ausfälle

2. **Kabelverdrehung vermeiden:**
   - CAT7-Kabel mit Zugentlastung fixieren
   - Keine scharfen Knicke im Kabel
   - Kabelkanal oder Schutzrohr verwenden

3. **Temperaturmanagement:**
   - LM2596 produziert Wärme → ausreichend Belüftung
   - Sensor-Node nicht direkter Sonneneinstrahlung aussetzen
   - Betriebstemperatur: -10°C bis +50°C

4. **Stromversorgung sichern:**
   - Netzteil mit Überlastschutz und Kurzschlussschutz
   - USV (unterbrechungsfreie Stromversorgung) erwägen
   - Alternative: Solarpanel + Akku (für autarken Betrieb)

5. **Software-Robustheit:**
   - Watchdog-Timer implementieren (Auto-Restart bei Freeze)
   - CRC-Prüfsummen für RS-485 Datenübertragung
   - Retry-Logik bei fehlgeschlagenen Messungen

### Erweiterungen & Optimierungen

**Mögliche Zukunfts-Upgrades:**

- 📊 **SD-Karten-Logging:** Langzeit-Aufzeichnung der Messwerte
- 📱 **WiFi/LoRa-Modul:** Remote-Zugriff auf Daten
- 🔔 **Alarm-System:** SMS/E-Mail bei kritischen Werten
- 🔋 **Batterie-Backup:** Betrieb bei Stromausfall
- 🌐 **Web-Interface:** Dashboard im Browser
- 📈 **Grafische Auswertung:** Trends und Statistiken

**Modularer Aufbau:** BioSync wurde so konzipiert, dass Erweiterungen leicht integrierbar sind!

---

## Zusätzliche Ressourcen

### Weitere Dokumentation

- **[Hauptdokumentation](../README.md)** – Projektübersicht und Systembeschreibung
- **[SensorNode README](../SensorNode/README.md)** – Firmware und Code-Struktur Sensor-Node
- **[DisplayNode README](../DisplayNode/README.md)** – Firmware und Code-Struktur Display-Node
- **[Nextion Anleitung](../Nextion/Anleitung.md)** – HMI Design und Display-Programmierung
- **[Komponenten-Liste](../Komponenten-Liste.md)** – Detaillierte Bauteile-Übersicht
- **[Spannungsversorgung](../Spannungsversorgung.md)** – Power-Management Details

### Hilfreiche Links

- **Arduino Nano Every:** [Offizielle Dokumentation](https://docs.arduino.cc/hardware/nano-every)
- **Arduino Mega:** [Pinout und Specs](https://docs.arduino.cc/hardware/mega-2560)
- **JSN-SR04T Datasheet:** [PDF](https://www.google.com/search?q=JSN-SR04T+datasheet)
- **DS18B20 Datasheet:** [PDF](https://datasheets.maximintegrated.com/en/ds/DS18B20.pdf)
- **LM2596 Datasheet:** [PDF](https://www.ti.com/lit/ds/symlink/lm2596.pdf)
- **MAX485 Datasheet:** [PDF](https://datasheets.maximintegrated.com/en/ds/MAX1487-MAX491.pdf)
- **Nextion Editor:** [Download](https://nextion.tech/nextion-editor/)

### Community & Support

- **GitHub Issues:** [Bug-Reports und Feature-Requests](https://github.com/LukeArrow/BioSync/issues)
- **Diskussionen:** [GitHub Discussions](https://github.com/LukeArrow/BioSync/discussions)
- **Arduino Forum:** [Arduino Community](https://forum.arduino.cc/)
- **RS-485 Tutorial:** [How RS-485 Works](https://www.ti.com/lit/an/slyt441/slyt441.pdf)

---

## Lizenz & Haftungsausschluss

**Lizenz:** Dieses Projekt steht unter der MIT-Lizenz (siehe LICENSE-Datei im Repository).

**⚠️ Haftungsausschluss:**
- Dieses Projekt ist ein DIY-Projekt und wird ohne Garantie bereitgestellt.
- Der Aufbau und Betrieb erfolgt auf eigene Verantwortung.
- Bei unsachgemäßem Aufbau können Komponenten beschädigt werden.
- Elektrische Arbeiten sollten nur von Personen mit entsprechender Fachkenntnis durchgeführt werden.
- Für Schäden, die durch den Nachbau oder Betrieb entstehen, wird keine Haftung übernommen.

---

**✅ Viel Erfolg beim Aufbau Ihres BioSync-Systems!**

Bei Fragen, Problemen oder Verbesserungsvorschlägen: [GitHub Issues](https://github.com/LukeArrow/BioSync/issues) öffnen oder [Diskussion starten](https://github.com/LukeArrow/BioSync/discussions).

*Erstellt: Oktober 2025 | Projekt: BioSync | Version: 1.0*
