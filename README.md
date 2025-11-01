# BioSync - Technische Dokumentation
## Vollständige Dokumentations-Übersicht

**Version:** 2.0  
**Datum:** November 2025  
**Status:** Production-Ready

---

## 📚 Dokumentations-Index

Diese umfassende technische Dokumentation deckt alle Aspekte des BioSync Klärgruben-Monitoring-Systems ab. Die Dokumentation ist so detailliert, dass ein Techniker ohne Vorkenntnisse das System vollständig nachbauen, in Betrieb nehmen und warten kann.

### Hauptdokumentation

| Dokument | Beschreibung | Zielgruppe | Seiten |
|----------|--------------|------------|--------|
| **[TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md)** | Hauptdokument mit System-Übersicht, Architektur-Diagrammen, technischen Spezifikationen | Alle Rollen | ~30 |
| **[HARDWARE_DETAILED.md](HARDWARE_DETAILED.md)** | Vollständige Hardware-Spezifikation, Schaltpläne, Stromversorgung, EMV | Hardware-Ingenieure, Techniker | ~25 |
| **[SOFTWARE_ARCHITECTURE.md](SOFTWARE_ARCHITECTURE.md)** | Software-Architektur, Code-Dokumentation auf Funktionsebene, State Machines | Software-Entwickler | ~20 |

### Spezialdokumentation

| Dokument | Beschreibung | Zielgruppe | Seiten |
|----------|--------------|------------|--------|
| **[COMMUNICATION_PROTOCOL.md](COMMUNICATION_PROTOCOL.md)** | RS-485 Protokoll-Spezifikation, Nachrichtenformat, Timing | Entwickler, Techniker | ~15 |
| **[SENSOR_INTEGRATION.md](SENSOR_INTEGRATION.md)** | Sensor-spezifische Dokumentation, Integration, Kalibrierung | Techniker, Installateure | ~20 |
| **[NEXTION_HMI_GUIDE.md](NEXTION_HMI_GUIDE.md)** | Nextion Display Programmierung, HMI Design, Event-Handling | UI-Entwickler, Techniker | ~15 |
| **[CALIBRATION_PROCEDURES.md](CALIBRATION_PROCEDURES.md)** | Schritt-für-Schritt Kalibrierungsverfahren für alle Sensoren | Techniker, Wartungspersonal | ~12 |
| **[TROUBLESHOOTING_GUIDE.md](TROUBLESHOOTING_GUIDE.md)** | Systematische Fehlerdiagnose, Entscheidungsbäume, Lösungen | Techniker, Support | ~18 |
| **[INSTALLATION_DETAILED.md](INSTALLATION_DETAILED.md)** | Detaillierte Installationsanleitung, Inbetriebnahme, Testing | Installateure, Techniker | ~15 |

### Bestehende Dokumentation (Ergänzend)

| Dokument | Beschreibung |
|----------|--------------|
| **[WIRING_GUIDE.md](WIRING_GUIDE.md)** | Verdrahtungsanleitung mit Pinbelegungen |
| **[TESTING_GUIDE.md](TESTING_GUIDE.md)** | Test-Szenarien und Validierung |
| **[BioSync_Handbuch.md](BioSync_Handbuch.md)** | Benutzerhandbuch |
| **[COMPLETE_DOCUMENTATION.md](COMPLETE_DOCUMENTATION.md)** | Vorherige Dokumentationsversion |

---

## 🎯 Dokumentation nach Rolle

### Für Hardware-Ingenieure / Elektronik-Techniker

**Start hier:**
1. [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md) - System-Übersicht
2. [HARDWARE_DETAILED.md](HARDWARE_DETAILED.md) - Schaltpläne, Komponenten
3. [WIRING_GUIDE.md](WIRING_GUIDE.md) - Praktische Verdrahtung
4. [INSTALLATION_DETAILED.md](INSTALLATION_DETAILED.md) - Installation
5. [TROUBLESHOOTING_GUIDE.md](TROUBLESHOOTING_GUIDE.md) - Fehlerdiagnose

**Relevante Abschnitte:**
- Stromversorgungsarchitektur (Berechnung, LM2596)
- RS-485 Bus-Konfiguration (Terminierung, Impedanz)
- CAT7-Kabel Adernbelegung (Parallel-Schaltung)
- PCB-Layout Empfehlungen
- EMI/EMC Überlegungen
- IP-Schutzklassen und Gehäuse

### Für Software-Entwickler

**Start hier:**
1. [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md) - System-Architektur
2. [SOFTWARE_ARCHITECTURE.md](SOFTWARE_ARCHITECTURE.md) - Code-Struktur
3. [COMMUNICATION_PROTOCOL.md](COMMUNICATION_PROTOCOL.md) - Protokoll
4. [SENSOR_INTEGRATION.md](SENSOR_INTEGRATION.md) - Sensor-APIs

**Relevante Abschnitte:**
- Software-Architektur und Modulstruktur
- State Machine Diagramme
- Interrupt-Handling, Timer-Konfiguration
- Memory Management (Flash, SRAM)
- Error-Handling Strategien
- Performance-Optimierung
- Protokoll-Parser Implementation

### Für Installations-Techniker

**Start hier:**
1. [INSTALLATION_DETAILED.md](INSTALLATION_DETAILED.md) - Hauptanleitung
2. [WIRING_GUIDE.md](WIRING_GUIDE.md) - Verdrahtung
3. [CALIBRATION_PROCEDURES.md](CALIBRATION_PROCEDURES.md) - Kalibrierung
4. [TROUBLESHOOTING_GUIDE.md](TROUBLESHOOTING_GUIDE.md) - Fehlersuche

**Checklisten:**
- [ ] Werkzeuge und Materialien bereit
- [ ] Stromversorgung geprüft (LM2596 auf 5.0V)
- [ ] Hardware aufgebaut und getestet
- [ ] Firmware hochgeladen
- [ ] Kommunikation funktioniert
- [ ] Sensoren kalibriert
- [ ] System-Test 24h durchgeführt

### Für Wartungspersonal

**Start hier:**
1. [CALIBRATION_PROCEDURES.md](CALIBRATION_PROCEDURES.md) - Regelmäßige Kalibrierung
2. [TROUBLESHOOTING_GUIDE.md](TROUBLESHOOTING_GUIDE.md) - Fehlerdiagnose
3. [SENSOR_INTEGRATION.md](SENSOR_INTEGRATION.md) - Sensor-Wartung

**Wartungsintervalle:**
- **Alle 3 Monate:** Sichtprüfung, Sensorreinigung
- **Jährlich:** Kalibrierung, Tiefergehende Inspektion
- **Bei Bedarf:** Komponenten-Austausch, Fehlerdiagnose

---

## 📖 Dokumentations-Features

### Umfang und Detailtiefe

- **~150+ Seiten** technische Dokumentation
- **50+ Code-Beispiele** mit Kommentaren
- **30+ Diagramme** (ASCII, Text-basiert)
- **100+ Tabellen** mit Spezifikationen
- **20+ Schritt-für-Schritt Prozeduren**

### Technische Präzision

- **Komponentenebene:** Pin-für-Pin Spezifikationen
- **Elektrische Werte:** Spannungen, Ströme, Widerstände berechnet
- **Timing-Analyse:** Millisekunden-genaue Diagramme
- **Fehlerszenarien:** Systematische Diagnose-Bäume
- **Code-Ebene:** Funktions-Dokumentation mit Parametern

### Praktische Anwendbarkeit

- **Reale Messungen:** Multimeter-Anweisungen
- **Fehlercodes:** Interpretation und Behebung
- **Kalibrierungs-Tabellen:** Zum Ausfüllen
- **Materiallisten:** Mit Mengenangaben und Kosten
- **Sicherheitshinweise:** An relevanten Stellen

---

## 🔧 Schnellzugriff - Häufige Aufgaben

### Ich möchte das System von Grund auf nachbauen

→ [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md) (System-Übersicht)  
→ [HARDWARE_DETAILED.md](HARDWARE_DETAILED.md) (Schaltpläne)  
→ [INSTALLATION_DETAILED.md](INSTALLATION_DETAILED.md) (Aufbauanleitung)

### Ich habe ein Problem mit der RS-485 Kommunikation

→ [TROUBLESHOOTING_GUIDE.md](TROUBLESHOOTING_GUIDE.md#5-kommunikations-probleme)  
→ [COMMUNICATION_PROTOCOL.md](COMMUNICATION_PROTOCOL.md) (Protokoll-Details)

### Sensor zeigt falsche Werte

→ [SENSOR_INTEGRATION.md](SENSOR_INTEGRATION.md) (Sensor-spezifisch)  
→ [CALIBRATION_PROCEDURES.md](CALIBRATION_PROCEDURES.md) (Kalibrierung)

### Nextion Display reagiert nicht

→ [NEXTION_HMI_GUIDE.md](NEXTION_HMI_GUIDE.md#8-troubleshooting)  
→ [TROUBLESHOOTING_GUIDE.md](TROUBLESHOOTING_GUIDE.md#7-display-probleme)

### LM2596 einstellen

→ [HARDWARE_DETAILED.md](HARDWARE_DETAILED.md#31-lm2596-buck-converter)  
→ [INSTALLATION_DETAILED.md](INSTALLATION_DETAILED.md#31-lm2596-vorbereitung-und-einstellung)

### Code verstehen / anpassen

→ [SOFTWARE_ARCHITECTURE.md](SOFTWARE_ARCHITECTURE.md)  
→ Quellcode mit inline Kommentaren in `/SensorNode/` und `/DisplayNode/`

---

## 📋 Dokumentations-Checkliste

### Für neue Installationen

- [ ] TECHNICAL_DOCUMENTATION gelesen (System verstehen)
- [ ] HARDWARE_DETAILED konsultiert (Komponenten, Schaltplan)
- [ ] INSTALLATION_DETAILED befolgt (Schritt-für-Schritt)
- [ ] WIRING_GUIDE verwendet (Verdrahtung)
- [ ] SOFTWARE_ARCHITECTURE verstanden (Firmware)
- [ ] CALIBRATION_PROCEDURES durchgeführt (Sensoren)
- [ ] TESTING_GUIDE befolgt (Systemtest)
- [ ] TROUBLESHOOTING_GUIDE griffbereit (Support)

### Für Entwickler (Code-Änderungen)

- [ ] SOFTWARE_ARCHITECTURE gelesen (Code-Struktur)
- [ ] COMMUNICATION_PROTOCOL verstanden (Protokoll)
- [ ] Änderungen dokumentiert (Code-Kommentare)
- [ ] Tests durchgeführt (Unit, Integration)
- [ ] TROUBLESHOOTING_GUIDE aktualisiert (neue Fehler)

---

## 🌐 Zusätzliche Ressourcen

### Externe Referenzen

- **Arduino Nano Every:** [docs.arduino.cc/hardware/nano-every](https://docs.arduino.cc/hardware/nano-every)
- **Arduino Mega 2560:** [docs.arduino.cc/hardware/mega-2560](https://docs.arduino.cc/hardware/mega-2560)
- **Nextion Editor:** [nextion.tech/nextion-editor](https://nextion.tech/nextion-editor/)
- **RS-485 Standard:** TIA/EIA-485-A Specification
- **CAT7 Kabel:** ISO/IEC 11801 Class F

### Datasheets

- JSN-SR04T Ultrasonic Sensor
- DS18B20 Temperature Sensor (Maxim Integrated)
- TSW-20M Turbidity Sensor
- CQRSENTDS01 TDS Sensor
- LM2596 Buck Converter (Texas Instruments)
- MAX485 RS-485 Transceiver (Maxim Integrated)

### Community

- **GitHub Repository:** [github.com/LukeArrow/BioSync](https://github.com/LukeArrow/BioSync)
- **Issues:** [github.com/LukeArrow/BioSync/issues](https://github.com/LukeArrow/BioSync/issues)
- **Discussions:** [github.com/LukeArrow/BioSync/discussions](https://github.com/LukeArrow/BioSync/discussions)

---

## 📝 Glossar (Auszug)

| Begriff | Erklärung |
|---------|-----------|
| **ADC** | Analog-to-Digital Converter (Analogwandler) |
| **Buck Converter** | Step-Down Spannungswandler (z.B. 12V → 5V) |
| **CAT7** | Kategorie 7 Ethernet-Kabel (S/FTP, bis 600 MHz) |
| **CRC** | Cyclic Redundancy Check (Prüfsumme) |
| **DE/RE** | Driver Enable / Receiver Enable (RS-485) |
| **EMI** | Electromagnetic Interference (Störungen) |
| **HMI** | Human-Machine Interface (Benutzeroberfläche) |
| **IP67** | Schutzart gegen Staub und zeitweiliges Untertauchen |
| **1-Wire** | Dallas Semiconductor One-Wire Protokoll |
| **ppm** | Parts Per Million (Einheit für TDS) |
| **RS-485** | Robuster serieller Kommunikations-Standard |
| **TDS** | Total Dissolved Solids (gelöste Feststoffe) |
| **UART** | Universal Asynchronous Receiver/Transmitter |

---

## ⚠️ Wichtige Hinweise

### Sicherheit

- ⚡ **Netzteil immer vor Arbeiten vom Strom trennen**
- 🔍 **Polarität immer prüfen** (+ und -)
- 💧 **IP67-Gehäuse im Schacht zwingend erforderlich**
- 🔧 **Werkzeug ordnungsgemäß verwenden**

### Haftungsausschluss

Dieses Projekt ist Open Source und wird ohne Garantie bereitgestellt. Der Aufbau und Betrieb erfolgt auf eigene Verantwortung. Für Schäden wird keine Haftung übernommen. Elektrische Arbeiten sollten nur von Fachpersonal durchgeführt werden.

### Lizenz

Dieses Projekt und die Dokumentation stehen unter der MIT-Lizenz.

---

## 📧 Support

Bei Fragen oder Problemen:

1. **Dokumentation durchsuchen:** Nutze die Suchfunktion (Ctrl+F)
2. **GitHub Issues:** [Neue Issue erstellen](https://github.com/LukeArrow/BioSync/issues/new)
3. **Discussions:** [Community fragen](https://github.com/LukeArrow/BioSync/discussions)

---

**Letzte Aktualisierung:** November 2025  
**Dokumentationsversion:** 2.0  
**Projekt:** BioSync Klärgruben-Monitoring-System  
**Ersteller:** LukeArrow & Contributors

---

*Die Dokumentation wird kontinuierlich verbessert. Feedback und Verbesserungsvorschläge sind willkommen!*

