# Arduino Car Projekt - Dokumentation

## Inhaltsverzeichnis
1. [Arduino Car - Dokumentation 14.10.2024](#arduino-car---dokumentation-14102024)
   - [Bauteile](#bauteile)
   - [Arduino Uno Pinbelegung](#arduino-uno-pinbelegung)
   - [L298N Motor Driver](#l298n-motor-driver)
   - [Motorenanschluss](#motorenanschluss)
   - [Verbindung der Stromversorgung](#verbindung-der-stromversorgung)
   - [Arduino-Code](#arduino-code)
2. [Infrarot-Sensoren - Dokumentation 04.11.2024](#infrarot-sensoren---dokumentation-04112024)
   - [Projektübersicht](#projektübersicht)
   - [Hintergrundinformationen zu den Sensoren](#hintergrundinformationen-zu-den-sensoren)
   - [Hardware-Setup](#hardware-setup)
   - [Software-Setup](#software-setup)
   - [Tabellen](#tabellen)
   - [Diagramme](#diagramme)
3. [Erweiterungen - Dokumentation 09.12.2024](#erweiterungen---dokumentation-09122024)
   - [Start- und Stopp-Taster](#start--und-stopp-taster)
   - [Mittlenregelung](#mittenregelung)

---

## Arduino Car - Dokumentation 14.10.2024

### Bauteile
- **Arduino Uno**
- **L298N Motorsteuerungsmodul**
- **4 Gleichstrommotoren**
- **2 Batterien**
- **Jumper-Kabel**
- **Motorenhalterung**
- **Chassis für das Auto**

### Arduino Uno Pinbelegung
- Die Pins **5, 6, 9 und 10** am Arduino Uno sind mit dem L298N-Modul verbunden:
  - Pin 8 -> IN1 (L298N)
  - Pin 9 -> IN2 (L298N)
  - Pin 10 -> IN3 (L298N)
  - Pin 11 -> IN4 (L298N)
- **Ground (GND)** vom Arduino ist mit dem GND des L298N-Moduls verbunden.

### L298N Motor Driver
- Steuert die beiden linken und rechten Motoren des Autos.
- Je zwei Motoren sind parallel an den Ausgängen des L298N verbunden.
- Die Batterie versorgt sowohl die Motoren als auch das L298N-Modul.

### Motorenanschluss
- **Linke Motoren**: An OUT1 und OUT2 des L298N-Moduls.
- **Rechte Motoren**: An OUT3 und OUT4 des L298N-Moduls.

### Verbindung der Stromversorgung
- Verbinde die Plus- und Minuspole der Batterie mit VCC und GND des L298N-Moduls.
- Schließe den 5V-Ausgang des L298N mit dem 5V-Pin des Arduino an, um das Arduino mit Strom zu versorgen.

### Arduino-Code
Der Code steuert ein Fahrzeug, indem er die Motoren über definierte Pins für Vorwärtsfahrt und Stopps aktiviert oder deaktiviert:

```cpp
void forward() {
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  digitalWrite(in3, HIGH);
  digitalWrite(in4, LOW);
}

void stop() {
  digitalWrite(in1, LOW);
  digitalWrite(in2, LOW);
  digitalWrite(in3, LOW);
  digitalWrite(in4, LOW);
}

```

## Infrarot Sensoren - Dokumentation 04.11.2024 

### Projektübersicht

Dieses Projekt befasst sich mit der Messung von Abständen mithilfe von drei Infrarot-Sensoren (Sharp 2Y0A02). Ziel ist es, die Beziehung zwischen den gemessenen ADC-Werten und der tatsächlichen Entfernung zu analysieren. Die Daten wurden mithilfe eines Arduinos erfasst, verarbeitet und anschließend in Excel visualisiert, um wichtige Erkenntnisse über die Leistung der Sensoren zu gewinnen.

Die Ergebnisse sollen dabei helfen, die Sensoren für praktische Anwendungen besser zu kalibrieren.

---

### Hintergrundinformationen zu den Sensoren

### Funktionsweise von Infrarot-Sensoren
Infrarot-Sensoren wie der **Sharp 2Y0A02** basieren auf der Messung reflektierter Infrarotstrahlen. Ein integrierter Emitter sendet einen IR-Strahl aus, der von Objekten reflektiert wird. Der Sensor misst die Stärke des reflektierten Signals und gibt eine Spannung aus, die proportional zur Entfernung ist. 

### Eigenschaften des Sharp 2Y0A02
- **Reichweite:** 10 cm bis 150 cm
- **Typ:** Analoger Entfernungssensor
- **Besonderheiten:**
  - Liefert genaue Messungen im Nahbereich.
  - Zeigt eine nichtlineare Abhängigkeit zwischen Spannung und Entfernung.
  - Ideal für Anwendungen wie Hinderniserkennung, Abstandsmessungen und Robotik.

### Herausforderung
Da die Spannungsausgabe nichtlinear ist, erfordert die Berechnung der tatsächlichen Entfernung eine Kalibrierung und ggf. eine lineare Approximation.

---

### Hardware-Setup

### Komponenten
- **Sensoren:** Drei Sharp Infrarot-Sensoren (Front, Left, Right)
- **Microcontroller:** Arduino Uno
- **Shield:** Arduino Sensor Shield zur einfachen Verbindung
- **Verkabelung:**
  - Front-Sensor: A0
  - Left-Sensor: A0
  - Right-Sensor: A0
- **Software:** Arduino IDE für die Datenerfassung und Microsoft Excel für die Datenanalyse

### Aufbau
- Die Sensoren wurden auf einem festen Gestell montiert und so ausgerichtet, dass sie parallel zum Boden messen.
- Entfernungen von 10 cm bis 150 cm wurden schrittweise getestet.

---

### Software-Setup

### Erfassung und Verarbeitung der Daten
Der folgende Arduino-Code wurde verwendet zur Kalibrierung des Sensors.

```cpp

        ir_sensor_front_raw = analogRead(IR_SENSOR_FRONT_PIN);
        total_adc_value += ir_sensor_front_raw;

        ir_sensor_front_new = (uint16_t)(16256.4 / (ir_sensor_front_raw + 22.8)) - 8;

        if (ir_sensor_front_new > 150) {
            ir_sensor_front_new = 151;
        } else if (ir_sensor_front_new < 20) {
            ir_sensor_front_new = 19;
```
---

### Tabellen

#### Front Sensor

Die Tabelle dokumentiert die Ergebnisse für den Front-Sensor. Neben den ADC-Werten und dem Umkehrwert der Entfernung (1/l) werden auch die linearen Näherungen (m*ADC+d) und die verwendeten Kalibrierungsparameter angegeben.

| Länge [cm] | ADC Wert | Umkehrwert | 1 / (l+k)  | m*ADC+d   |
|------------|----------|------------|------------|-----------|
| 20         | 512      | 0.00195    | 0.03571    | 0.032289  |
| 30         | 408      | 0.00245    | 0.02632    | 0.026316  |
| 40         | 317      | 0.00315    | 0.02083    | 0.021089  |
| 50         | 247      | 0.00405    | 0.01724    | 0.017068  |
| 60         | 206      | 0.00485    | 0.01471    | 0.014713  |
| 70         | 172      | 0.00581    | 0.01282    | 0.012776  |
| 80         | 145      | 0.00690    | 0.01136    | 0.011121  |
| 90         | 112      | 0.00893    | 0.01020    | 0.009314  |
| 100        | 96       | 0.01042    | 0.00926    | 0.008395  |
| 110        | 87       | 0.01149    | 0.00877    | 0.007878  |
| 120        | 76       | 0.01316    | 0.00725    | 0.007246  |
| 130        | 64       | 0.01563    | 0.00676    | 0.006557  |
| 140        | 60       | 0.01667    | 0.00633    | 0.006327  |

#### Right/Left Sensor

Die Tabelle zeigt die gemessenen Werte für den rechten Sensor. Sie enthält die ADC-Werte, den Umkehrwert der Entfernung (1/l), und die berechneten linearen Näherungen basierend auf den Kalibrierungsparametern.

| Länge [cm] | ADC Wert | Umkehrwert | 1 / (l+k)  | m*ADC+d   |
|------------|----------|------------|------------|-----------|
| 10         | 497      | 0.00201    | 0.05556    | 0.06508854|
| 20         | 280      | 0.00357    | 0.03571    | 0.03740996|
| 30         | 190      | 0.00526    | 0.02632    | 0.02593037|
| 40         | 148      | 0.00676    | 0.02083    | 0.02057323|
| 50         | 124      | 0.00806    | 0.01724    | 0.017512  |
| 60         | 102      | 0.00980    | 0.01471    | 0.01470588|
| 70         | 87       | 0.01149    | 0.01282    | 0.01279262|
| 80         | 78       | 0.01282    | 0.01136    | 0.01164466|

---

### Diagramme

#### Front Sensor ADC-Wert

Dieses Diagramm zeigt den Verlauf der ADC-Werte des Front-Sensors in Abhängigkeit von der Entfernung. Es zeigt eine ähnliche nichtlineare Kennlinie wie beim linken und rechten Sensor.

<img src="./pics/adc_front.PNG" alt="verkabelung" width="500" height="280"/>

#### Front Sensor Linearitätsanalyse

Dieses Diagramm vergleicht den gemessenen Umkehrwert der Entfernung (1/l) mit der linearen Näherung (m*ADC+d) für den Front-Sensor. Auch hier stimmt die Näherung gut mit den Messwerten überein.

<img src="./pics/front.PNG" alt="verkabelung" width="500" height="280"/>

#### Right/Left Sensor ADC-Wert

Dieses Diagramm zeigt den Verlauf der ADC-Werte des linken Sensors in Abhängigkeit von der Entfernung. Die Werte nehmen mit zunehmender Entfernung ab und zeigen die typische nichtlineare Kennlinie eines IR-Sensors.

<img src="./pics/adc_right.PNG" alt="verkabelung" width="500" height="280"/>

#### Right/Left Sensor Linearitätsanalyse

Dieses Diagramm vergleicht den gemessenen Umkehrwert der Entfernung (1/l) mit der linearen Näherung (m*ADC+d) für den rechten Sensor. Es zeigt, dass die Näherung eine gute Übereinstimmung mit den Messwerten bietet.

<img src="./pics/right.PNG" alt="verkabelung" width="500" height="280"/>

## Erweiterungen - Dokumentation 09.12.2024

### Start- und Stopp-Taster

#### Beschreibung
Im heutigen Schritt wurde das Projekt um zwei Taster erweitert: einen **Start-Taster** und einen **Stopp-Taster**. Diese ermöglichen die manuelle Steuerung des Fahrzeugs.

- **Schwarzer Taster (Start):** Aktiviert den Betrieb des Autos. Solange der Taster gedrückt wird, führt das Auto die programmierten Bewegungen aus.
- **Roter Taster (Stopp):** Stoppt das Fahrzeug sofort. Dies hat Vorrang vor allen anderen Funktionen.

#### Hardware
- **Taster-Verbindungen:**
  - **Start-Taster:** Pin 2 (Arduino, INPUT_PULLUP)
  - **Stopp-Taster:** Pin 3 (Arduino, INPUT_PULLUP)

#### Funktionsweise
- Der Stopp-Taster wird geprüft, bevor eine Bewegung stattfindet. Wenn er gedrückt wird, werden die Motoren sofort gestoppt.
- Der Start-Taster aktiviert die normalen Bewegungsroutinen, solange keine Hindernisse erkannt werden.

Hier ist der Codeausschnitt für die Taster:
```cpp
#define startButtonPin 2
#define stopButtonPin 3

bool isCarRunning = false;

void setup() {
  pinMode(startButtonPin, INPUT);
  pinMode(stopButtonPin, INPUT);
}

void loop() {
  if (digitalRead(startButtonPin) == HIGH) {
    isCarRunning = true; // Auto starten
    delay(200); // Entprellung
  }
  if (digitalRead(stopButtonPin) == HIGH) {
    isCarRunning = false; // Auto stoppen
    delay(200); // Entprellung
  }

  if (isCarRunning) {
    // Hauptlogik des Autos
  } else {
    // Auto bleibt stehen
    stop();
  }
}
```

---

### Mittenregelung

#### Beschreibung
Eine neue Logik wurde implementiert, um Hindernissen dynamisch auszuweichen. Das Fahrzeug verwendet die drei IR-Sensoren (vorne, links, rechts), um freie Wege zu erkennen und entsprechend zu reagieren.

#### Funktionsweise
1. **Hindernisprüfung:** Wenn der Front-Sensor eine Entfernung von weniger als 30 cm erkennt, prüft das System die Werte der linken und rechten Sensoren.
2. **Abbiegelogik:** 
   - Wenn links mehr Platz ist als rechts, biegt das Fahrzeug nach links ab.
   - Wenn rechts mehr Platz ist als links, biegt das Fahrzeug nach rechts ab.
3. **Freie Fahrt:** Ist der Weg vor dem Fahrzeug frei, fährt es geradeaus.

#### Hardware
- **Front-Sensor:** Pin A0
- **Left-Sensor:** Pin A1
- **Right-Sensor:** Pin A2

Hier ist der erweiterte Code für die Mittenregelung:
```cpp

int frontThreshold = 400; // Schwelle für Hinderniserkennung vorne
int sideThreshold = 200;  // Schwelle für Hinderniserkennung seitlich

void loop() {
  if (isCarRunning) {
    int frontValue = analogRead(sensorFront);
    int rightValue = analogRead(sensorRight);
    int leftValue = analogRead(sensorLeft);

    if (frontValue > frontThreshold) {
      if (rightValue < leftValue) {
        turnLeft(); // Nach links abbiegen
      } else {
        turnRight(); // Nach rechts abbiegen
      }
    } else {
      forward(); // Weiter vorwärts fahren
    }
  } else {
    stop(); // Auto bleibt stehen
  }
}
```





