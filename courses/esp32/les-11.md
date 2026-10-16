# Les 11 – Servomotor aansturen

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), SG90-servomotor

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat een servomotor doet en waarin die verschilt van een gewone motor;
- een SG90-servo aansluiten op de ESP32;
- met de bibliotheek **ESP32Servo** een servo draaien naar een vaste hoek;
- een sweep-programma schrijven dat de servo vloeiend laat bewegen;
- uitleggen waarom de servo aparte voeding of een volle USB-poort nodig heeft.

## 📖 Introductie

Tot nu toe stuurde je vooral één soort uitvoer aan: lampen en geluid. Maar veel slimme apparaten moeten iets kunnen **bewegen**: een raam dat opent, een klep die dichtgaat, een deur die opengaat.

Daarvoor gebruik je een **servomotor**. Een servo is een motor met ingebouwde besturing: je stuurt een hoek (bijvoorbeeld 90°) en de servo draait zelf naar die positie en houdt die vast.

**Benodigd:** ESP-32D-board, USB-C-datakabel, laptop met Arduino IDE 2, een SG90-servo en jumper wires.

> [!NOTE]
> De SG90 werkt op 5 V. Voor deze les mag de servo gevoed worden via de 5V-pin van de ESP32 zolang de laptop via USB voedt. In grotere projecten gebruik je een aparte voeding.

## 💡 Theorie

### Wat is een servomotor?

Een servo bevat een kleine motor, een tandwieloverbrenging en een feedback-sensor. De besturing in de servo vergelijkt de gewenste hoek met de echte positie en draait de motor net zolang tot die klopt.

| Begrip | Betekenis |
|---|---|
| Servo | Motor die naar een ingestelde hoek draait |
| PWM-signaal | Pulssignaal dat de gewenste hoek doorgeeft |
| SG90 | Kleine, goedkope servo van 180° |
| Bereik 0–180° | De SG90 kan 180 graden draaien |

Een servopulssignaal is heel precies: een puls van 500 microseconden betekent ongeveer 0°, een puls van 2400 microseconden ongeveer 180°. Gelukkig regelt een bibliotheek dat voor je.

### Waarom aparte voeding?

Een servo kan piekstromen trekken die groter zijn dan de ESP32-pin kan leveren. Sluit daarom **nooit** de motorstroom van een servo rechtstreeks op een GPIO-pin aan. De signaaldraad mag dat wel: die stuurt alleen het besturingssignaal.

## 🛠️ Opdracht

### Stap 1 – Bibliotheek installeren

1. Open in Arduino IDE **Sketch → Include Library → Manage Libraries**.
2. Zoek naar **ESP32Servo** en installeer deze bibliotheek.

### Stap 2 – De servo aansluiten

Sluit de USB-kabel eerst af. De SG90 heeft drie draden:

| Servodraad | Kleur (meestal) | Naar ESP32 |
|---|---|---|
| Signaal | oranje | GPIO13 |
| Voeding | rood | 5V (VIN) |
| GND | bruin | GND |

> [!WARNING]
> Controleer de kleuren op jouw eigen servo; fabrikanten verschillen. Sluit voeding nooit aan op 3V3: een servo heeft 5 V nodig.

### Stap 3 – Eerste programma

Maak een nieuwe sketch:

```cpp
#include <ESP32Servo.h>

const int SERVO_PIN = 13;

Servo mijnServo;

void setup() {
  Serial.begin(115200);
  ESP32PWM::allocateTimer(0);
  mijnServo.setPeriodHertz(50);
  mijnServo.attach(SERVO_PIN, 500, 2400);
}

void loop() {
  mijnServo.write(0);
  Serial.println("Hoek: 0");
  delay(1000);
  mijnServo.write(90);
  Serial.println("Hoek: 90");
  delay(1000);
  mijnServo.write(180);
  Serial.println("Hoek: 180");
  delay(1000);
}
```

**Wat gebeurt er?**

- `setPeriodHertz(50)` zet het PWM-signaal op 50 Hz, de standaard voor servo's.
- `attach(pin, 500, 2400)` geeft de minimale en maximale pulstijd door.
- `write(90)` draait de servo naar 90 graden.

Upload het programma en controleer of de servo elke seconde naar een nieuwe stand draait.

### Stap 4 – Zelfstandig aanpassen

**Opdracht A (verplicht):** laat de servo vloeiend van 0° naar 180° draaien met stappen van 5° en een pauze van 50 ms per stap, en daarna terug naar 0°. Gebruik een `for`-lus.

**Opdracht B (verplicht):** maak een functie `draaiNaar(int hoek)` die:

1. de hoek begrenst tussen 0 en 180 met `constrain()`;
2. de servo naar die hoek draait;
3. in de Serial Monitor print: `Servo staat op: <hoek> graden`.

Laat in `loop()` de servo om de 2 seconden wisselen tussen 45° en 135° via deze functie.

## ⭐ Bonusopdracht

Laat de servo bedienen met getallen 0–180 via **Serial Input** (`Serial.available()` en `Serial.parseInt()`).

## 💻 GitHub

Bewaar je sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-11`.

## 🤖 AI-opdracht

Vraag een AI-assistent waarom een servo zijn positie vasthoudt terwijl een gewone motor blijft doorlopen. Controleer: duw je zachtjes tegen de servo-arm terwijl het programma draait? Bespreek met een medestudent.

## 🧠 Reflectie

1. Wat is het verschil tussen een servo en een gewone (DC-)motor?
2. Waarom mag de servo-voeding niet op een GPIO-pin?
3. Wat doet `constrain()` in jouw functie?

## 📤 Inleveren

**Lever één bestand in:** `les11_servo.ino` via de Canvas-opdracht. Laat je werkende servo aan de docent zien.

## 🧠 Kennischeck

1. Over hoeveel graden kan de SG90 draaien?
2. Welke bibliotheek gebruik je voor servo's op de ESP32?
3. Wat gebeurt er bij `mijnServo.write(180)`?
4. Welke drie draden heeft een servo en wat doen ze?

## 📚 Samenvatting

Je kunt nu een servomotor aansluiten en met een vaste hoek of een vloeiende beweging aansturen. In **les 12** ga je een ventilator met een motordriver aansturen.