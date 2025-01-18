# Mechanischen und hardware technischen Aufbaus

## Bauteile

Folgende elektronische Bauteile sind initial in Verwendung:
- Arduino Board
- Sensor-Shield
- L298N Motor-Driver
- Sharp IR Range Sensor
  - 1x GP2Y0A02YK0F
  - 2x GP2Y0A21YK0F
- 4x Räder mit Motoren (und Decoderscheiben)
- 1x doppelt Batteriehalterung
- ein kleines Steckbrett

Chassisplaten und 3D Druck Halterungen und solche teile sind auch in Verwendung (es haltet sich leider nicht mit Magie zusammen).

Die datenblätter sind zu finden in [datasheets](../datasheets)
<!-- Kann möglicherweise entfernt werden und durch links zur Quelle ersetzt -->

## Grundsätzliche stromversorgung

![Schaltbild Skizze welches die Anschlüsse vom Akku zum Motortreiber und Arduino darstellt.](img/Stromversorgung_schaltplan.jpg)
Die Strom Versorgung unserer Battery (~7,4V) geht zum Motortreiber eingang, dieser gibt über den 5V Output die Versorgung an den Arduino Uno weiter. Die Drei Teile sind an einer gemeinsamen GND angeschlossen.

## physischer Aufbau

Zurzeit ist das fahrzeug in zwei ebenen Unterteilt.
In der Untersten sind alle vier Räder (offensichtlich) und der Motor Treiber.
Darüber auf der zweiten Ebene sind Batterie Halterung, Arduino, Steckbrett und Sharp IR Sensoren (mit 3D-Druck mount)

## Arduino Motor verknüpf

![alt text](img/Arduino-Motor-pincontrolls.jpg)

# IR Sensor

## Sensorerfassung

### werte Erfassung zur Kalibrierung

Standardmäßig werden unsere IR Sensoren uns spannungswerte liefern jedoch nicht sehr gebräuchlich.
Einerseits das es kein klares Verständnis zwischen Spannung und Entfernung herrscht und anderer seits weil sie immer eindeutig sind.
Dies liegt aufgrund dessen das die Spannungswerte nicht streng genommen linear mit der Entfernung verlaufen, siehe dazu die Graphik. 

![Spannungsverlauf zu Entfernung des Front Sensors vom Datenblatt](img/Datasheet-Front-IRSensor.png)

Weiters kann jeweiliges Stück leicht abweichen vom datenblatt.

Um dieses problem zu lösen werden wir die werte an gegebenen Entfernungen erfassen und mit der gegebenen Formel Linearisieren.
<!-- füg Formel ein -->

#### Messerfassung
![Aufbau der wert Erfassung](img/sharp_erfassung.jpg)
<!-- füg graphic ein -->
Die benötigten werte zur Erfassung haben wir ganz einfach ermittelt indem wir die gegebenen werte für die jeweiligen entfernungsschritte messen.
Der  Code dafür ermisst einfach alle werte in einem kurzen zeitfenster und nimmt den Durchschnitt davon.

Siehe [sensor_test/car11/src/main.cpp](../sensor_test/car11/src/main.cpp)
```cpp
#include <Arduino.h>

unsigned long startMillis;  
unsigned long currentMillis;
const unsigned long period = 250;

uint8_t IR_SENSOR = A0;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  pinMode(IR_SENSOR,INPUT);
  startMillis = millis();  //initial start time
}

long sum=0;
int count=0;
void loop() {

  int ir_sensor_raw = analogRead(IR_SENSOR);
  sum+=ir_sensor_raw;
  count++;

  currentMillis = millis();   
  if (currentMillis - startMillis >= period)  
  {
  // read IR sensor data
  int value = sum/count;
  Serial.println(String(value));
  sum=0;
  count=0;
  startMillis = currentMillis;  
  }
}
```
#### Ergebnisse

Front (F 9Z)
| Länge | Werte | Umkehrwert | 1 / (l+k)   | m\*ADC+d    |
| ----- | ----- | ---------- | ----------- | ----------- |
| 20    | 504   | 0,00198413 | 0,035714286 | 0,031484135 |
| 30    | 417   | 0,00239808 | 0,026315789 | 0,026315789 |
| 40    | 314   | 0,00318471 | 0,020833333 | 0,020196944 |
| 50    | 258   | 0,00387597 | 0,017241379 | 0,016870193 |
| 60    | 212   | 0,00471698 | 0,014705882 | 0,014137504 |
| 70    | 179   | 0,00558659 | 0,012820513 | 0,012177097 |
| 80    | 162   | 0,00617284 | 0,011363636 | 0,011167191 |
| 90    | 138   | 0,00724638 | 0,010204082 | 0,00974144  |
| 100   | 122   | 0,00819672 | 0,009259259 | 0,00879094  |
| 110   | 114   | 0,00877193 | 0,008474576 | 0,00831569  |
| 120   | 100   | 0,01       | 0,0078125   | 0,007484002 |
| 130   | 96    | 0,01041667 | 0,007246377 | 0,007246377 |
| 140   | 87    | 0,01149425 | 0,006756757 | 0,00671172  |

Left(F 06)
| Länge | Werte | Umkehrwert | 1 / (l+k)   | m\*ADC+d    |
| ----- | ----- | ---------- | ----------- | ----------- |
| 10    | 503   | 0,00198807 | 0,071428571 | 0,078846704 |
| 20    | 259   | 0,003861   | 0,041666667 | 0,042500834 |
| 30    | 166   | 0,0060241  | 0,029411765 | 0,028647695 |
| 40    | 122   | 0,00819672 | 0,022727273 | 0,022093522 |
| 50    | 98    | 0,01020408 | 0,018518519 | 0,018518519 |
| 60    | 80    | 0,0125     | 0,015625    | 0,015837266 |
| 70    | 70    | 0,01428571 | 0,013513514 | 0,014347681 |
| 80    | 59    | 0,01694915 | 0,011904762 | 0,012709138 |

Right (F 21)
| Länge | Werte | Umkehrwert | 1 / (l+k)   | m\*ADC+d    |
| ----- | ----- | ---------- | ----------- | ----------- |
| 10    | 485   | 0,00206186 | 0,071428571 | 0,072883228 |
| 20    | 265   | 0,00377358 | 0,041666667 | 0,042372422 |
| 30    | 167   | 0,00598802 | 0,029411765 | 0,028781244 |
| 40    | 126   | 0,00793651 | 0,022727273 | 0,023095139 |
| 50    | 93    | 0,01075269 | 0,018518519 | 0,018518519 |
| 60    | 79    | 0,01265823 | 0,015625    | 0,016576922 |
| 70    | 62    | 0,01612903 | 0,013513514 | 0,014219269 |
| 80    | 52    | 0,01923077 | 0,011904762 | 0,012832414 |

Siehe auch Messungen [Sharp_IR-Sensoren_Linearisierung.xlsx](messungen/Sharp_IR-Sensoren_Linearisierung.xlsx)

### Generische Umsetzung der Sensoren

<!-- Formel -->
\[
\frac{1}{l + k} = m \cdot ADC + d
\]

\[
l = \frac{1}{m \cdot ADC + d} - k \quad \rightarrow \quad l = \frac{m'}{ADC + d'} - k
\]

<!-- welche Bereiche für Formel verwendet wurden -->

|      | Front      | Left       | Right      |
| ---- | ---------- | ---------- | ---------- |
| k =  | 8          | 4          | 4          |
| m =  | 5,9406E-05 | 0,00014896 | 0,00014147 |
| d =  | 0,00154337 | 0,00392059 | 0,00465415 |
| m' = | 16833,24   | 6713,28    | 7068,48    |
| d' = | 25,98      | 26,32      | 32,8977778 |

Beispiel Umsetzung im code für den Front Sensor:
```cpp
uint16_t Sensor_front(int IR_SENSOR_FRONT){
    uint16_t ir_sensor_front_raw = analogRead(IR_SENSOR_FRONT);
    uint16_t ir_sensor_front_new = (uint16_t) (16833.24 / (ir_sensor_front_raw + 25.98)) - 8;

    if(ir_sensor_front_new > 150)
        ir_sensor_front_new = 151;
    else if(ir_sensor_front_new < 20)
        ir_sensor_front_new = 19;

    return ir_sensor_front_new;
}
```
