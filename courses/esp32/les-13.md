# Les 13 – Snelheid regelen met PWM en een potmeter

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), L9110S, ventilator, potmeter 10 kΩ

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat PWM (pulsbreedtemodulatie) is;
- een potmeter als analoge ingang uitlezen op de ESP32;
- de gemeten waarde omrekenen naar een snelheid (0–255);
- de ventilator vloeiend regelen met de potmeter;
- een ADC-waarde naar een bruikbare waarde mappen met `map()`.

## 📖 Introductie

Aan en uit is handig, maar een goede ventilator, lampdimmer of motoraansturing is **regelbaar**. Daarvoor gebruiken we **PWM**: de pin wisselt heel snel aan en uit, en de verhouding bepaalt hoeveel energie de motor krijgt.

Als besturing gebruiken we een **potmeter**: een draaiknop die een spanningswaarde tussen 0 en 3,3 V afgeeft. De ESP32 zet die met een ADC (analoog-naar-digitaalomzetter) om naar een getal.

**Benodigd:** de opstelling van les 12, plus een potmeter van 10 kΩ.

## 💡 Theorie

### Wat is PWM?

| Begrip | Betekenis |
|---|---|
| PWM | Snel aan/uit-schakelen van een pin |
| Duty cycle | Percentage van de tijd dat de pin HOOG is |
| 0 % | Pin altijd uit → motor staat stil |
| 100 % | Pin altijd aan → motor op volle kracht |
| Resolutie | In hoeveel stappen we regelen, vaak 0–255 |

Op de ESP32 gebruik je `analogWrite(pin, waarde)` met een waarde van 0 tot 255. Achter de schermen maakt de ESP32 daar een PWM-signaal van.

### De potmeter en de ADC

- Sluit de uiterste pinnen van de potmeter aan op **3V3** en **GND**.
- De middelste pin (slider) gaat naar een ADC-ingang: **GPIO34**.
- `analogRead(34)` geeft een waarde tussen ongeveer 0 en 4095 (12 bits).
- Met `map(waarde, 0, 4095, 0, 255)` reken je dat om naar 0–255.

> [!NOTE]
> GPIO34–39 hebben geen pull-up of pull-down en zijn alleen ingang. Daardoor zijn ze ideaal voor analoge sensoren.

### Waarom niet lager dan ~30?

Veel kleine motorjes draaien niet bij hele lage PWM-waarden; ze hebben een minimale puls nodig om te starten. Daarom begrenzen we de laagste waarde.

## 🛠️ Opdracht

### Stap 1 – Potmeter aansluiten

| Potmeter | Naar ESP32 |
|---|---|
| Pin 1 | 3V3 |
| Pin 2 (middelste) | GPIO34 |
| Pin 3 | GND |

### Stap 2 – Testprogramma

```cpp
const int PIN_IA = 26;
const int PIN_IB = 25;
const int PIN_POT = 34;

void setup() {
  pinMode(PIN_IA, OUTPUT);
  pinMode(PIN_IB, OUTPUT);
  Serial.begin(115200);
  digitalWrite(PIN_IB, LOW);
}

void loop() {
  int ruw = analogRead(PIN_POT);          // 0 - 4095
  int snelheid = map(ruw, 0, 4095, 0, 255);
  snelheid = constrain(snelheid, 0, 255);

  if (snelheid < 30) {
    digitalWrite(PIN_IA, LOW);            // volledig uit
    Serial.println("Ventilator uit");
  } else {
    analogWrite(PIN_IA, snelheid);
    Serial.print("Snelheid: ");
    Serial.println(snelheid);
  }
  delay(200);
}
```

Upload het programma en draai langzaam aan de potmeter. De ventilator moet van stil naar vol gas gaan.

### Stap 3 – Zelfstandig aanpassen

**Opdracht A (verplicht):** print naast de snelheid ook het **percentage**: `Snelheid: 51 % (130/255)`.

**Opdracht B (verplicht):** maak een functie `zetVentilator(int procent)` die:

1. het percentage begrenst tussen 0 en 100 met `constrain()`;
2. het percentage omrekent naar 0–255;
3. de ventilator instelt en print: `Ventilator staat op 40%`.

Roep die functie aan vanuit `loop()` met de potmeterwaarde.

## ⭐ Bonusopdracht

Laat de ventilator automatisch 10 seconden oplopen van 0 naar 100 % en 10 seconden zakken, zonder potmeter. Gebruik een `for`-lus.

## 💻 GitHub

Bewaar je opgeschoonde sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-13`.

## 🤖 AI-opdracht

Vraag een AI-assistent uit te leggen wat de duty cycle van een PWM-signaal is. Controleer: zet de potmeter op de helft en meet of de snelheid ongeveer 127 van 255 is. Klopt dat met de uitleg?

## 🧠 Reflectie

1. Wat verandert er aan het PWM-signaal als je de potmeter omdraait?
2. Waarom reken je 0–4095 om naar 0–255?
3. Waarom stopt de motor onder een bepaalde waarde?

## 📤 Inleveren

**Lever één bestand in:** `les13_pwm.ino`. Laat de regelbare ventilator aan de docent zien.

## 🧠 Kennischeck

1. Wat is de duty cycle bij `analogWrite(pin, 64)` (van 255)?
2. Welke waarde geeft `analogRead(34)` maximaal ongeveer terug?
3. Wat doet `map()`?
4. Waarom is GPIO34 geschikt als analoge ingang?

## 📚 Samenvatting

Je kunt nu de snelheid van een motor vloeiend regelen met PWM en een potmeter. In **les 14** leer je met een **relais** een apparaat schakelen dat meer stroom vraagt, zoals een waterpomp.