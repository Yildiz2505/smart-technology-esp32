# Les 2 – LED en weerstand

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat een LED en een weerstand doen;
- de begrippen **volt (V)**, **ampère (A)** en **ohm (Ω)** herkennen;
- de plus- en minkant van een gewone LED onderscheiden;
- een eenvoudige stroomkring op een breadboard aansluiten;
- met `pinMode()` en `digitalWrite()` een LED laten knipperen;
- de knippersnelheid zelfstandig veranderen.

## 📖 Introductie

In les 1 liet je de ESP32 berichten naar je laptop sturen. Nu gaan we voor het eerst iets **fysieks aansturen**: een LED-lampje.

Je sluit een LED en een weerstand aan op het breadboard. Daarna programmeer je de ESP32 zodat de LED aan en uit gaat.

**Benodigd:**

- ESP-32D WiFi+BT met USB-C-datakabel;
- eigen laptop met Arduino IDE 2;
- breadboard;
- rode LED (gewone 3 mm of 5 mm LED);
- weerstand van **220 Ω of 330 Ω**;
- enkele jumper wires.

> [!NOTE]
> Je hoeft nog geen ervaring met elektronica te hebben. We bouwen de schakeling stap voor stap. Gebruik voor deze les één gewone rode LED, geen LED-module met ingebouwde weerstand.

## 💡 Theorie

### 1. Wat is een LED?

LED staat voor **Light Emitting Diode**. Een LED is een elektronisch onderdeel dat licht geeft als er stroom in de juiste richting doorheen loopt.

Een gewone LED heeft twee aansluitingen:

- **Anode (+):** meestal het **lange pootje**.
- **Kathode (−):** meestal het **korte pootje**; aan deze zijde zit vaak ook een **platte rand** aan de behuizing.

![LED met lange en korte poot](images/les-02/led-polariteit.svg)

*Afbeelding 1. Herken de anode (+) en kathode (−) voordat je de LED aansluit.*

### 2. Waarom gebruiken we een weerstand?

Een LED kan beschadigd raken als er te veel stroom doorheen loopt. Daarom plaatsen we een **weerstand in serie** met de LED.

Drie begrippen zijn belangrijk:

| Begrip | Symbool | Eenvoudige uitleg |
|---|---|---|
| Spanning | Volt (V) | Het verschil in elektrische potentiaal dat stroom kan laten lopen |
| Stroomsterkte | Ampère (A) | Hoeveel elektrische lading per seconde door een onderdeel loopt |
| Weerstand | Ohm (Ω) | Hoe sterk een onderdeel de stroom tegenwerkt |

De ESP32 werkt met **3,3 V-logica**. In deze les gebruiken we **220 Ω of 330 Ω** om de stroom door een gewone rode LED te beperken.

De wet van Ohm beschrijft de relatie tussen spanning, stroom en weerstand:

`U = I × R`

Hierbij is `U` de spanning in volt, `I` de stroom in ampère en `R` de weerstand in ohm.

> [!WARNING]
> Sluit een losse LED **nooit zonder serieweerstand** rechtstreeks aan op een GPIO-pin. Verander bedrading alleen als de USB-kabel los is.

### 3. Wat is een breadboard?

Een breadboard is een oefenbord waarop je onderdelen kunt verbinden zonder te solderen. Op een standaard breadboard zijn **de vijf gaten A–E van dezelfde genummerde rij onderling verbonden**. De vijf gaten F–J aan de andere kant vormen een afzonderlijke groep. De lange voedingsrails kunnen onderbroken zijn; controleer daarom altijd hoe jouw breadboard is opgebouwd.

## 🛠️ Opdracht

### Stap 1 – Bouw de schakeling (15 minuten)

**Haal eerst de USB-kabel uit de ESP32.**

Voor deze les gebruiken we **GPIO18** en **GND**. Controleer de labels die op jouw ESP32-board staan.

De stroomkring wordt:

**GPIO18 → weerstand (220/330 Ω) → anode (+) LED → kathode (−) LED → GND**

![Aansluitschema ESP32 LED en weerstand](images/les-02/esp32-led-schema.svg)

*Afbeelding 2. Aansluitschema voor jullie ESP-32D-board. De weerstand mag ook tussen de kathode en GND staan, zolang hij maar in serie met de LED is aangesloten.*

**Voorbeeld met een standaard breadboard (linkerkant A–E):**

1. Verbind **GPIO18** van de ESP32 met **E12** op het breadboard.
2. Plaats één pootje van de **weerstand in A12** en het andere in **A16**.
3. Plaats het **lange LED-pootje (+) in B16**.
4. Plaats het **korte LED-pootje (−) in B20**.
5. Verbind **E20** met een **GND-pin** van de ESP32.
6. Controleer alles en sluit **daarna pas** USB weer aan.

De letters zijn voorbeeldposities; bij een ander breadboard kun je dezelfde elektrische verbindingen op andere rijen maken. Het belangrijkste is dat de stroomkring exact klopt.

> [!TIP]
> Werkt de LED straks niet? Kijk eerst of de LED omgekeerd zit en of de onderdelen in de juiste *verbonden rijen* zitten.

### Stap 2 – Schrijf je eerste LED-programma (15 minuten)

Maak in Arduino IDE een nieuwe sketch:

```cpp
const int LED_PIN = 18;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);

  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
```

**Uitleg:**

- `const int LED_PIN = 18;` bewaart het gebruikte GPIO-nummer.
- `pinMode(LED_PIN, OUTPUT);` maakt GPIO18 een uitgang.
- `digitalWrite(LED_PIN, HIGH);` zet de uitgang hoog: de LED gaat aan.
- `digitalWrite(LED_PIN, LOW);` zet de uitgang laag: de LED gaat uit.
- `delay(1000);` wacht één seconde.

Upload de sketch naar de ESP32. De LED moet nu **één seconde branden** en daarna **één seconde uit** zijn. Dit herhaalt zich vanzelf.

### Stap 3 – Zelfstandig programmeren (20 minuten)

**Opdracht A (verplicht):** pas de code aan zodat de LED **0,5 seconde aan** en **0,5 seconde uit** is.

**Opdracht B (verplicht):** laat de LED een patroon uitvoeren dat zich blijft herhalen:

1. Kort aan: **200 ms**
2. Kort uit: **200 ms**
3. Kort aan: **200 ms**
4. Lang uit: **1000 ms**

Gebruik daarvoor `digitalWrite()` en `delay()`.

**Test:** laat de docent of een medestudent zien dat de LED steeds twee keer kort knippert, gevolgd door een langere pauze.

## ⭐ Bonusopdracht

Maak een functie `knipper(int aanTijd, int uitTijd)` die een LED eenmaal laat knipperen. Gebruik deze functie vervolgens twee keer in `loop()` om het patroon van opdracht B uit te voeren.

> [!TIP]
> Je kent functies al uit het programmeren. Bedenk welke regels in je code je vaak herhaalt en hoe je die in één functie kunt plaatsen.

## 💻 GitHub

**Optioneel:** je mag je werk bewaren in een eigen GitHub-repository, bijvoorbeeld in de map `smart-technology/les-02`. Een GitHub-upload is voor deze les **niet verplicht**.

## 🤖 AI-opdracht

Vraag een AI-assistent waarom een LED een serieweerstand nodig heeft. Vergelijk de uitleg met de theorie en kijk of je zelf kunt uitleggen waarom direct aansluiten op GPIO18 onverstandig is. Je hoeft hier niets over in te leveren.

## 🧠 Reflectie

Bespreek kort met een medestudent:

1. Wat gebeurde er toen je de wachttijd veranderde?
2. Wat gebeurt er als je de LED verkeerd om aansluit?
3. Waarom gebruiken we een weerstand?

Je hoeft deze antwoorden niet schriftelijk in te leveren.

## 📤 Inleveren

**Lever één bestand in:** `les2_led.ino` via de Canvas-opdracht van les 2.

Je programma moet:

1. GPIO18 gebruiken;
2. een LED met `digitalWrite()` aan en uit zetten;
3. het **twee-keer-kort-knipperen-patroon** van opdracht B laten zien.

**Laat tijdens de les je werkende schakeling aan de docent zien.**

> [!NOTE]
> Je hoeft geen foto's, screenshots of reflectieverslag in te leveren. De bonusopdracht is vrijwillig.

## 🧠 Kennischeck

1. Wat is een LED?
2. Waarom gebruiken we een weerstand bij een LED?
3. Wat is het verschil tussen volt, ampère en ohm?
4. Waarom moeten de lange en korte poot van de LED op verschillende elektrische verbindingen zitten?
5. Wat doet `pinMode(LED_PIN, OUTPUT)`?
6. Wat doet `digitalWrite(LED_PIN, HIGH)`?
7. Hoelang is `delay(200)` in seconden?

## 📚 Samenvatting

Je hebt voor het eerst een fysieke schakeling gebouwd en aangestuurd met een ESP32. Je weet waarom een LED een weerstand nodig heeft en je kunt met `digitalWrite()` zelf een knipperpatroon programmeren.

In **les 3** voeg je een drukknop toe, zodat je de LED met een fysieke invoer kunt bedienen.
