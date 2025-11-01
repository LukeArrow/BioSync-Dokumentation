# BioSync - Kalibrierungsverfahren
## Schritt-für-Schritt Kalibrierungsanleitungen

**Version:** 2.0  
**Datum:** November 2025

---

## Inhaltsverzeichnis

1. [Kalibrierungs-Übersicht](#1-kalibrierungs-übersicht)
2. [JSN-SR04T Distanzkalibrierung](#2-jsr-sr04t-distanzkalibrierung)
3. [DS18B20 Temperaturkalibrierung](#3-ds18b20-temperaturkalibrierung)
4. [TSW-20M Trübungskalibrierung](#4-tsw-20m-trübungskalibrierung)
5. [CQRSENTDS01 TDS-Kalibrierung](#5-cqrsentds01-tds-kalibrierung)
6. [Kalibrierungsprotokoll](#6-kalibrierungsprotokoll)

---

## 1. Kalibrierungs-Übersicht

### 1.1 Wann kalibrieren?

| Ereignis | JSN-SR04T | DS18B20 | TSW-20M | TDS |
|----------|-----------|---------|---------|-----|
| Erste Installation | ✓ | ✓ | ✓ | ✓ |
| Nach Sensor-Austausch | ✓ | ✓ | ✓ | ✓ |
| Jährliche Wartung | ✓ | ✓ | ✓ | ✓ |
| Bei Genauigkeitsverlust | ✓ | ✓ | ✓ | ✓ |
| Nach Firmware-Update | - | - | ✓ | ✓ |

### 1.2 Benötigte Werkzeuge

**Für alle Sensoren:**
- Laptop mit Arduino IDE und Serial Monitor
- USB-Kabel zum Arduino
- Notizblock für Messwerte

**Sensor-spezifisch:**
- **JSN-SR04T:** Maßband (2m), flache Referenzfläche
- **DS18B20:** Geeichtes Thermometer, Eiswasser, Wasserbad
- **TSW-20M:** Destilliertes Wasser, Trübungs-Standardlösungen (optional)
- **TDS:** TDS-Eichlösungen (342 ppm, 1413 ppm), destilliertes Wasser

---

## 2. JSN-SR04T Distanzkalibrierung

### 2.1 Offset-Kalibrierung

**Ziel:** Systematische Abweichung korrigieren

**Benötigt:**
- Maßband oder Zollstock (mind. 2m)
- Flache, harte Referenzfläche (z.B. Brett, Tisch)
- Stabiler Messaufbau

**Durchführung:**

```
Schritt 1: Messaufbau
  ┌─────────────┐
  │ JSN-SR04T   │ ← Sensor fest montieren
  │ Transducer  │   (senkrecht nach unten)
  └──────┬──────┘
         │
         │ h = ?
         │
         ▼
  ═══════════════ ← Flache Referenzfläche

Schritt 2: Referenzmessungen
  Position 1: h = 50 cm
  Position 2: h = 100 cm
  Position 3: h = 200 cm
  
  Je Position: 10 Messungen durchführen

Schritt 3: Serial Monitor verwenden
  > Tools → Serial Monitor
  > Baudrate: 115200
  > Messwerte notieren
```

**Beispiel-Messdaten:**

| Soll-Distanz (cm) | Messung 1 | Messung 2 | Messung 3 | ... | Mittelwert | Fehler |
|-------------------|-----------|-----------|-----------|-----|------------|--------|
| 50.0 | 49.2 | 49.3 | 49.1 | ... | 49.2 | -0.8 |
| 100.0 | 99.4 | 99.6 | 99.5 | ... | 99.5 | -0.5 |
| 200.0 | 199.2 | 199.4 | 199.3 | ... | 199.3 | -0.7 |

```
Offset berechnen:
  Offset = Durchschnitt(Fehler)
         = (-0.8 + -0.5 + -0.7) / 3
         = -0.67 cm
```

**Code-Anpassung:**

```cpp
// In SensorNode/sensors.cpp:

const float DISTANCE_OFFSET = -0.67;  // Kalibrierter Wert

float measureDistance() {
  // ... bestehender Code ...
  float distance = duration / 58.0;
  
  // Offset anwenden
  distance += DISTANCE_OFFSET;
  
  return distance;
}
```

**Validierung:**

1. Firmware neu kompilieren und hochladen
2. Erneut bei 50, 100, 200 cm messen
3. Abweichung sollte nun <±0.5 cm sein

---

## 3. DS18B20 Temperaturkalibrierung

### 3.1 Zwei-Punkt-Kalibrierung

**Ziel:** Offset und Linearität korrigieren

**Benötigt:**
- Geeichtes Referenzthermometer (±0.1°C)
- Eiswasser (0°C Referenz)
- Wasserbad mit einstellbarer Temperatur ODER
- Raumtemperatur (stabil, gemessen mit Referenzthermometer)

**Durchführung:**

```
Punkt 1: Eiswasser (0°C)
  1. Glas mit Eiswürfeln füllen, destilliertes Wasser zugeben
  2. Warten bis thermisches Gleichgewicht (5 min)
  3. DS18B20 Sonde eintauchen (vollständig unter Wasser)
  4. Referenzthermometer ebenfalls eintauchen
  5. 10 Messungen durchführen (je 1 Minute Abstand)
  6. Mittelwerte bilden

Punkt 2: Raumtemperatur (~20°C)
  1. Sensor und Referenzthermometer in stabiler Umgebung (keine Zugluft)
  2. 30 Minuten akklimatisieren lassen
  3. 10 Messungen durchführen (je 1 Minute Abstand)
  4. Mittelwerte bilden
```

**Beispiel-Messdaten:**

| Referenz (°C) | DS18B20 (°C) | Differenz (°C) |
|---------------|--------------|----------------|
| 0.0 | 0.3 | +0.3 |
| 20.0 | 20.2 | +0.2 |

```
Offset berechnen:
  Offset = Durchschnitt(Differenz)
         = (0.3 + 0.2) / 2
         = 0.25°C
  
  Korrektur: Gemessene Temperatur - 0.25°C
```

**Code-Anpassung:**

```cpp
// In SensorNode/sensors.cpp:

const float TEMP_OFFSET = -0.25;  // Kalibrierter Wert

float readTemperature() {
  dsSensor.requestTemperatures();
  float temp = dsSensor.getTempCByIndex(0);
  
  if (temp == DEVICE_DISCONNECTED_C) {
    return -127.0;
  }
  
  // Offset anwenden
  temp += TEMP_OFFSET;
  
  return temp;
}
```

---

## 4. TSW-20M Trübungskalibrierung

### 4.1 Null-Punkt-Kalibrierung (Destilliertes Wasser)

**Ziel:** Sensor-Offset bei klarem Wasser bestimmen

**Durchführung:**

1. **Vorbereitung:**
   - Glas mit destilliertem Wasser füllen
   - TSW-20M Sonde reinigen (Linsen säubern)
   - Sensor vollständig eintauchen

2. **Messung:**
   ```
   Serial Monitor öffnen (115200 baud)
   20 Messungen durchführen (je 5 Sekunden)
   Messwerte notieren
   ```

3. **Beispiel-Daten:**
   ```
   Destilliertes Wasser (sollte ~0 sein):
     Messungen: 12, 15, 13, 14, 12, 11, 13, ...
     Mittelwert: 13
     → Offset = -13
   ```

4. **Code-Anpassung:**
   ```cpp
   const int TURBIDITY_OFFSET = -13;
   
   int readTurbidity() {
     int raw = analogRead(TURB_PIN);
     raw += TURBIDITY_OFFSET;
     if (raw < 0) raw = 0;  // Clamp to 0
     return raw;
   }
   ```

### 4.2 Skalierungsfaktor (Optional mit Standards)

Falls Trübungs-Standards verfügbar:

```
Standard 1: 100 NTU → gemessen: 250 (nach Offset)
Standard 2: 400 NTU → gemessen: 980 (nach Offset)

Skalierungsfaktor: NTU = (ADC_value × 0.4)
```

---

## 5. CQRSENTDS01 TDS-Kalibrierung

### 5.1 Zwei-Punkt-Kalibrierung mit Eichlösungen

**Ziel:** ADC-Werte in ppm umrechnen

**Benötigt:**
- TDS-Eichlösung 342 ppm (low standard)
- TDS-Eichlösung 1413 ppm (high standard)
- Destilliertes Wasser (zum Spülen)
- Bechergläser

**Durchführung:**

```
Punkt 1: 342 ppm Standard
  1. Becherglas mit Eichlösung 342 ppm füllen
  2. Sonde eintauchen, schwenken (Luftblasen entfernen)
  3. 1 Minute warten (Temperaturausgleich)
  4. 10 Messungen notieren
  5. Mittelwert: z.B. ADC = 412

Punkt 2: 1413 ppm Standard
  1. Sonde mit dest. Wasser spülen, abtrocknen
  2. Becherglas mit Eichlösung 1413 ppm füllen
  3. Sonde eintauchen, schwenken
  4. 1 Minute warten
  5. 10 Messungen notieren
  6. Mittelwert: z.B. ADC = 870
```

**Kalibrierungskurve berechnen:**

```
Lineare Regression: TDS(ppm) = m × ADC + b

Gegeben:
  Punkt 1: ADC=412, TDS=342
  Punkt 2: ADC=870, TDS=1413

Steigung m:
  m = (TDS2 - TDS1) / (ADC2 - ADC1)
    = (1413 - 342) / (870 - 412)
    = 1071 / 458
    = 2.34

Y-Achsenabschnitt b:
  b = TDS1 - m × ADC1
    = 342 - 2.34 × 412
    = 342 - 964
    = -622

Formel: TDS = 2.34 × ADC - 622
```

**Code-Anpassung:**

```cpp
const float TDS_SLOPE = 2.34;
const float TDS_INTERCEPT = -622.0;

float readTDS_ppm() {
  int adc = analogRead(TDS_PIN);
  float tds = TDS_SLOPE * adc + TDS_INTERCEPT;
  if (tds < 0) tds = 0;  // Clamp to 0
  return tds;
}

// Wenn nur ADC-Wert gesendet wird (aktuell):
int readTDS() {
  return analogRead(TDS_PIN);
}

// Im Display Node kann dann umgerechnet werden:
// TDS_ppm = 2.34 × ADC_value - 622
```

---

## 6. Kalibrierungsprotokoll

### 6.1 Vorlage zum Ausfüllen

```
═══════════════════════════════════════════════════════════════
         BIOSYNC SENSOR KALIBRIERUNGSPROTOKOLL
═══════════════════════════════════════════════════════════════

Datum: __________________  Techniker: _______________________

System-ID: ______________  Firmware-Version: _______________

┌─────────────────────────────────────────────────────────────┐
│ JSN-SR04T (Ultraschall-Distanzsensor)                       │
├─────────────────────────────────────────────────────────────┤
│ Soll (cm) │ Messung (cm) │ Fehler (cm) │                   │
├───────────┼──────────────┼─────────────┼───────────────────┤
│   50.0    │              │             │                   │
│  100.0    │              │             │                   │
│  200.0    │              │             │                   │
├───────────┴──────────────┴─────────────┴───────────────────┤
│ Berechneter Offset: ____________ cm                         │
│ Im Code eingetragen: □ Ja  □ Nein                           │
│ Validiert (Abweichung <0.5cm): □ Ja  □ Nein                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ DS18B20 (Temperatursensor)                                  │
├─────────────────────────────────────────────────────────────┤
│ Referenz (°C) │ Messung (°C) │ Differenz (°C) │            │
├───────────────┼──────────────┼────────────────┼────────────┤
│      0.0      │              │                │            │
│     20.0      │              │                │            │
├───────────────┴──────────────┴────────────────┴────────────┤
│ Berechneter Offset: ____________ °C                         │
│ Im Code eingetragen: □ Ja  □ Nein                           │
│ Validiert (Abweichung <0.5°C): □ Ja  □ Nein                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ TSW-20M (Trübungssensor)                                    │
├─────────────────────────────────────────────────────────────┤
│ Destilliertes Wasser ADC: ____________                      │
│ Berechneter Offset: ____________                            │
│ Im Code eingetragen: □ Ja  □ Nein                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ CQRSENTDS01 (TDS-Sensor)                                    │
├─────────────────────────────────────────────────────────────┤
│ Standard (ppm) │ ADC-Wert │                                 │
├────────────────┼──────────┼─────────────────────────────────┤
│      342       │          │                                 │
│     1413       │          │                                 │
├────────────────┴──────────┴─────────────────────────────────┤
│ Steigung (m): ____________                                  │
│ Achsenabschnitt (b): ____________                           │
│ Formel: TDS = ______ × ADC + ______                         │
│ Im Code eingetragen: □ Ja  □ Nein                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Bemerkungen:                                                │
│                                                             │
│                                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Unterschrift Techniker: _____________________________________

═══════════════════════════════════════════════════════════════
```

