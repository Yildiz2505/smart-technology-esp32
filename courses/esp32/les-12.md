# Les 12 – Ventilator aansturen met een motordriver

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), L9110S motordriver, 3–5 V DC-ventilator

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen waarom je een motor niet rechtstreeks op een GPIO-pin aansluit;
- een DC-motor via een **L9110S motordriver** aansluiten;
- de motor starten, stoppen en van richting laten draaien;
- de koppeling GPIO → driver → motor uitleggen;
- veilig werken met bewegende onderdelen.

## 📖 Introductie

Een ventilator, pomp of speelgoedmotor vraagt veel meer stroom dan een ESP32-pin kan leveren (maximaal ongeveer 20–40 mA per pin). Een kleine DC-motor wil al gauw 100–300 mA. Sluit je die toch rechtstreeks aan, dan kan de ESP32 beschadigd raken.

Daarom gebruik je een **motordriver**: een klein printplaatje met powertransistors dat de stroom naar de motor schakelt. De ESP32 stuurt de driver met een klein signaal; de driver schakelt de grote stroom.

**Benodigd:** ESP-32D-board, USB-C-datakabel, laptop, L9110S-driver, 3–5 V DC-ventilator, breadboard en jumper wires.

> [!NOTE]
> In deze les laten we de ventilator alleen aan/uit en van richting wisselen. De snelheid regelen doe je in **les 13** met PWM.

## 💡 Theorie

### Waarom een motordriver?

| Onderdeel | Taak |
|---|---|
| ESP32 GPIO | Stuurt een klein besturingssignaal (3,3 V, enkele mA) |
| L9110S | Versterkt het signaal en schakelt de motorstroom |
| Motor | Zet elektrische energie om in beweging |

De L9110S heeft per motor twee besturingspinnen: **IA** en **IB**.

| IA | IB | Motor |
|---|---|---|
| LAAG | LAAG | stilstaand |
| HOOG | LAAG | vooruit draaien |
| LAAG | HOOG | achteruit draaien |
| HOOG | HOOG | remmen (niet gebruiken) |

De motor werkt op 3–5 V, die je van de 5V-lijn (VIN) van de ESP32 neemt. De logica van de driver werkt prima op 3,3 V-signalen van de ESP32.

> [!WARNING]
> Houd vingers, haar en kabels uit de buurt van draaiende blades. Zet de blades er pas op als de schakeling is nagekeken.

## 🛠️ Opdracht

### Stap 1 – Aansluiten

Sluit de USB-kabel af en maak deze verbindingen:

| L9110S | Naar ESP32 |
|---|---|
| VCC | 5V (VIN) |
| GND | GND |
| IA | GPIO26 |
| IB | GPIO25 |
| OUTA / OUTB | motoraansluitingen |

> [!TIP]
> Draait de motor de verkeerde kant op? Wissel de twee motoraansluitingen om, of verwissel IA en IB in je code.

### Stap 2 – Testprogramma

```cpp
const int PIN_IA = 26;
const int PIN_IB = 25;

void stopMotor() {
  digitalWrite(PIN_IA, LOW);
  digitalWrite(PIN_IB, LOW);
}

void setup() {
  pinMode(PIN_IA, OUTPUT);
  pinMode(PIN_IB, OUTPUT);
  Serial.begin(115200);
  stopMotor();
}

void loop() {
  Serial.println("Vooruit");
  digitalWrite(PIN_IA, HIGH);
  digitalWrite(PIN_IB, LOW);
  delay(2000);
  Serial.println("Stop");
  stopMotor();
  delay(1000);
  Serial.println("Achteruit");
  digitalWrite(PIN_IA, LOW);
  digitalWrite(PIN_IB, HIGH);
  delay(2000);
  Serial.println("Stop");
  stopMotor();
  delay(1000);
}
```

Upload het programma en controleer of de motor 2 seconden vooruit draait, stopt, 2 seconden achteruit draait en weer stopt.

### Stap 3 – Zelfstandig aanpassen

**Opdracht A (verplicht):** maak functies `startVentilator()` en `stopVentilator()`. Laat de ventilator 5 seconden draaien en 5 seconden stil zijn.

**Opdracht B (verplicht):** voeg een **drukknop** toe (zoals in les 3): elke druk wisselt de ventilator tussen aan en uit. Gebruik een `bool ventilatorAan` en detecteer de druk zonder `delay()` te gebruiken.

## ⭐ Bonusopdracht

Laat de motor korte tikjes draaien (0,5 s aan, 0,5 s uit, 5 keer) en laat in de Serial Monitor zien hoeveel tikjes er al geweest zijn.

## 💻 GitHub

Bewaar je opgeschoonde sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-12`.

## 🤖 AI-opdracht

Vraag een AI-assistent wat er gebeurt als je IA en IB tegelijk HOOG maakt. Controleer het antwoord in de theorie en bespreek met een medestudent waarom deze stand "remmen" heet.

## 🧠 Reflectie

1. Waarom mag een DC-motor niet rechtstreeks op een GPIO-pin?
2. Wat doet de motordriver precies?
3. Hoe wissel je de draairichting van de motor?

## 📤 Inleveren

**Lever één bestand in:** `les12_ventilator.ino`. Laat de werkende ventilator met drukknop aan de docent zien.

## 🧠 Kennischeck

1. Welke twee pinnen van de L9110S stuur je aan?
2. Wat doen IA en IB bij een stilstaande motor?
3. Welke spanning gebruikt de motor en welke gebruikt de besturing?

## 📚 Samenvatting

Je kunt nu een DC-motor via een motordriver aan, uit en van richting wisselen. In **les 13** regel je de **snelheid** met PWM en een potmeter.