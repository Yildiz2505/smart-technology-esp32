# Les 3 – Drukknop en digitale invoer

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat het verschil is tussen **invoer** (INPUT) en **uitvoer** (OUTPUT);
- een drukknop op een breadboard aansluiten;
- uitleggen waarom we `INPUT_PULLUP` gebruiken;
- de stand van een knop uitlezen met `digitalRead()`;
- met een `if/else`-statement een LED bedienen;
- een programma aanpassen op basis van een fysieke invoer.

## 📖 Introductie

In les 2 liet je een LED knipperen met een programma. Maar hoe laat je een apparaat reageren op een gebruiker? Met een **drukknop**.

Vandaag blijft de LED uit zolang je niets doet. Druk je op de knop, dan gaat de LED aan. De ESP32 leest dus een **invoersignaal** en bepaalt vervolgens wat er gebeurt met een **uitgangssignaal**.

**Benodigd:**

- ESP-32D WiFi+BT en USB-C-datakabel;
- eigen laptop met Arduino IDE 2;
- breadboard en jumper wires;
- drukknop met vier pootjes (tactile switch);
- rode LED en weerstand van **220 Ω of 330 Ω** (uit les 2).

> [!NOTE]
> Je bouwt verder met dezelfde LED en weerstand als in les 2, maar deze les is ook zelfstandig te maken. Controleer voordat je begint de aansluiting van GPIO18 en GND.

## 💡 Theorie

### 1. Invoer en uitvoer

Een **invoer (INPUT)** vertelt de ESP32 wat er buiten het programma gebeurt. Bijvoorbeeld: is een knop ingedrukt?

Een **uitvoer (OUTPUT)** laat de ESP32 iets aansturen. Bijvoorbeeld: de LED aanzetten.

| Onderdeel | Rol | GPIO |
|---|---|---|
| Drukknop | Invoer (INPUT) | GPIO23 |
| Rode LED met weerstand | Uitvoer (OUTPUT) | GPIO18 |
| GND | Gemeenschappelijk referentiepunt | GND |

### 2. HIGH, LOW en INPUT_PULLUP

Een digitale ingang leest een toestand als **HIGH** of **LOW**. Een loshangende ingang kan onbetrouwbaar wisselen. Daarom gebruiken we `INPUT_PULLUP`: een interne weerstand houdt GPIO23 normaal op HIGH.

De knop verbindt GPIO23 met GND wanneer je hem indrukt.

| Knop | `digitalRead(23)` |
|---|---|
| Niet ingedrukt | `HIGH` |
| Ingedrukt | `LOW` |

**Belangrijk:** bij `INPUT_PULLUP` betekent **LOW dat de knop is ingedrukt**. Dat lijkt misschien omgekeerd, maar komt door de manier waarop de knop is aangesloten.

### 3. Hoe werkt een drukknop met vier pootjes?

Veel kleine drukknoppen hebben vier pootjes. **Twee pootjes aan dezelfde zijde zijn meestal al met elkaar verbonden**. Als je de knop indrukt, worden beide zijden met elkaar verbonden.

![Uitleg vierpoots drukknop](images/les-03/drukknop-uitleg.svg)

*Afbeelding 1. Schematische uitleg van een gebruikelijke vierpoots drukknop. Controleer bij jouw exemplaar welke pootjes bij elkaar horen.*

> [!WARNING]
> Steek een vierpoots drukknop bij voorkeur **over de middensleuf van het breadboard**, zodat de twee schakelzijden elektrisch gescheiden zijn. Controleer welke pootjes aan dezelfde kant zitten. Verander de bedrading alleen wanneer de USB-kabel is losgekoppeld.

## 🛠️ Opdracht

### Stap 1 – Sluit LED en drukknop aan (15 minuten)

**Haal eerst de USB-kabel uit je ESP32.**

De verbindingen zijn:

**LED-schakeling (zoals in les 2):**

1. **GPIO18 → weerstand van 220/330 Ω → lange LED-poot (+)**.
2. **Korte LED-poot (−) → GND**.

**Drukknop:**

3. Verbind **GPIO23** met één schakelzijde van de drukknop.
4. Verbind de **tegenoverliggende schakelzijde** met **GND**.
5. Controleer of de knop bij loslaten de verbinding verbreekt. We gebruiken **geen externe weerstand** voor de knop, omdat `INPUT_PULLUP` die functie intern verzorgt.
6. Laat indien nodig de docent je bedrading controleren en sluit pas daarna de USB-kabel weer aan.

![Aansluitschema ESP32 drukknop en LED](images/les-03/esp32-knop-led-schema.svg)

*Afbeelding 2. Het elektrische aansluitschema: GPIO23 → drukknop → GND en GPIO18 → serieweerstand → LED → GND. De tekening toont de verbindingen; de werkelijke plaats op je breadboard kan anders zijn.*

### Stap 2 – Test eerst alleen de drukknop (10 minuten)

Maak een nieuwe Arduino-sketch en upload:

```cpp
const int KNOP_PIN = 23;

void setup() {
  Serial.begin(115200);
  pinMode(KNOP_PIN, INPUT_PULLUP);
}

void loop() {
  int knopWaarde = digitalRead(KNOP_PIN);

  if (knopWaarde == LOW) {
    Serial.println("Knop ingedrukt");
  } else {
    Serial.println("Knop losgelaten");
  }

  delay(200);
}
```

Open de **Serial Monitor** op **115200 baud**. Druk een paar keer op de knop en controleer of de berichten veranderen.

> [!TIP]
> Blijft de knop altijd 'ingedrukt'? Controleer of je niet twee pootjes gebruikt die al permanent met elkaar verbonden zijn.

### Stap 3 – Bedien de LED met de drukknop (15 minuten)

Vervang de code door:

```cpp
const int KNOP_PIN = 23;
const int LED_PIN = 18;

void setup() {
  pinMode(KNOP_PIN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  int knopWaarde = digitalRead(KNOP_PIN);

  if (knopWaarde == LOW) {
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }
}
```

**Wat moet je zien?** De LED brandt **alleen zolang je de knop ingedrukt houdt**.

**Uitleg:**

- `pinMode(KNOP_PIN, INPUT_PULLUP)`: zet de knop als ingang met interne pull-up.
- `digitalRead(KNOP_PIN)`: leest of de knop ingedrukt is.
- `if (knopWaarde == LOW)`: voert code uit als de knop is ingedrukt.
- `digitalWrite(LED_PIN, HIGH)`: schakelt de LED aan.
- `else`: bepaalt wat er gebeurt als de knop niet ingedrukt is.

### Stap 4 – Zelfstandig aanpassen (20 minuten)

**Opdracht A (verplicht):** laat de LED juist **branden wanneer de knop niet is ingedrukt**. Druk je de knop in, dan gaat de LED uit.

**Opdracht B (verplicht):** laat de LED **knipperen zolang je de knop ingedrukt houdt**:

- 200 ms aan;
- 200 ms uit;
- bij loslaten blijft de LED uit.

Gebruik hierbij `if/else`, `digitalRead()`, `digitalWrite()` en `delay()`.

**Test:** laat de docent of een medestudent zien dat je LED reageert op de knop.

## ⭐ Bonusopdracht

Maak van de knop een **aan/uit-schakelaar**: één keer drukken zet de LED aan; nog een keer drukken zet hem weer uit. De LED moet dus aan blijven nadat je de knop loslaat.

**Aanwijzingen:**

- Gebruik een `bool ledAan` om de stand van de LED te onthouden.
- Reageer alleen op het moment dat de knop verandert van `HIGH` naar `LOW`.
- Een mechanische drukknop kan bij één druk meerdere snelle signalen geven: dit heet **contactdender (debouncing)**. Voeg eventueel een korte vertraging of een tijdcontrole met `millis()` toe.

> [!NOTE]
> Deze bonus is uitdagender dan de basisopdracht. Een simpele `if (knopWaarde == LOW)` is niet genoeg om betrouwbaar één keer per druk van stand te wisselen.

## 💻 GitHub

**Optioneel:** bewaar je code in je eigen GitHub-repository, bijvoorbeeld onder `smart-technology/les-03`. Voor deze les hoef je niets naar GitHub te pushen.

## 🤖 AI-opdracht

Vraag een AI-assistent: **Waarom is een knop met INPUT_PULLUP ingedrukt bij LOW?** Controleer het antwoord met de theorie en jouw eigen metingen in de Serial Monitor. Je hoeft niets in te leveren.

## 🧠 Reflectie

Bespreek kort met een medestudent:

1. Wat is het verschil tussen `digitalRead()` en `digitalWrite()`?
2. Waarom gebruiken we `INPUT_PULLUP`?
3. Wat gaat er mis als je de knop verkeerd op het breadboard plaatst?

Je hoeft deze antwoorden niet schriftelijk in te leveren.

## 📤 Inleveren

**Lever één bestand in:** `les3_drukknop.ino` via de Canvas-opdracht van les 3.

Je programma moet:

1. de knop uitlezen op **GPIO23** met `INPUT_PULLUP`;
2. de LED op **GPIO18** aansturen;
3. de LED laten **knipperen zolang de knop ingedrukt is**, en uitschakelen wanneer je de knop loslaat (opdracht B).

**Laat tijdens de les je werkende schakeling aan de docent zien.**

> [!NOTE]
> Geen screenshots, foto's of schriftelijke reflectie nodig. De bonusopdracht is vrijwillig.

## 🧠 Kennischeck

1. Wat is het verschil tussen een INPUT en OUTPUT?
2. Welke functie gebruik je om een knop uit te lezen?
3. Wat betekent `LOW` bij onze knop met `INPUT_PULLUP`?
4. Wat doet een `if/else`-statement?
5. Waarom heeft de LED wél een serieweerstand nodig, maar de knop met `INPUT_PULLUP` geen externe pull-upweerstand?
6. Wat is contactdender?

## 📚 Samenvatting

Je hebt een drukknop aangesloten en geleerd hoe een ESP32 op een digitale invoer reageert. Met `digitalRead()` en `if/else` kun je gedrag programmeren dat afhankelijk is van een gebruiker.

In **les 4** gebruiken we een afstandssensor om automatisch te reageren wanneer een voorwerp dichtbij komt.
