# Les 8 – Lichtsterkte meten met de BH1750

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat **lichtsterkte in lux** betekent;
- het verschil tussen een analoge sensor en een **digitale I²C-sensor** herkennen;
- de aansluitingen **VCC, GND, SDA en SCL** correct verbinden;
- de bibliotheek **BH1750** installeren en gebruiken;
- lichtwaarden in de Serial Monitor uitlezen;
- een eigen lichtgrens instellen en een waarschuwing programmeren.

## 📖 Introductie

Een smartphone kan de helderheid van het scherm aanpassen aan de omgeving. Een slimme lamp kan aangaan wanneer het donker wordt. Daarvoor moet het systeem eerst weten **hoeveel licht** er is.

In deze les gebruik je de **BH1750**. De sensor stuurt een digitale lichtmeting naar de ESP32. De waarde wordt uitgedrukt in **lux**.

**Benodigd:**

- ESP-32D WiFi+BT met USB-C-datakabel en Arduino IDE 2;
- een **BH1750-lichtsensormodule die geschikt is voor 3,3 V-logica**;
- breadboard en vier jumper wires;
- zaklamp (eventueel telefoonlamp) en een boek of stuk karton.

> [!WARNING]
> Er bestaan meerdere soorten BH1750-modules. Controleer de opschriften en de technische specificaties. Voed in deze les uitsluitend een voor **3,3 V geschikte** module met 3V3. Sommige modules hebben een extra **ADDR**-aansluiting; die laten we ongemoeid. Verbind nooit zonder controle 5V-signalen met ESP32-GPIO's.

## 💡 Theorie

### 1. Wat betekent lux?

**Lux (lx)** is een eenheid voor de hoeveelheid licht die op een oppervlak valt (verlichtingssterkte).

| Situatie | Wat verwacht je ongeveer? |
|---|---|
| Donkere ruimte | Weinig lux |
| Normale binnenverlichting | Meer lux |
| Lamp vlak bij de sensor | Veel lux |

De werkelijke meetwaarden hangen af van de afstand, richting en verlichting. We gebruiken **geen vaste 'goede' luxwaarde** voor iedere situatie.

![Lichtsterkte meten in lux](images/les-08/lux-uitleg.svg)

*Afbeelding 1. De BH1750 meet verlichtingssterkte in lux: minder licht geeft doorgaans een lagere waarde.*

### 2. Wat is I²C?

In les 7 gebruikte je `analogRead()` voor een analoog signaal. De BH1750 gebruikt **I²C**, een digitale communicatiemethode. De sensor stuurt meetgegevens via twee signaallijnen:

- **SDA** = gegevens (data);
- **SCL** = kloksignaal (clock).

Daarnaast heeft de sensor een voeding en GND nodig.

Bij deze ESP32 gebruiken we **GPIO21 voor SDA** en **GPIO22 voor SCL**.

> [!NOTE]
> Andere ESP32-modellen kunnen andere standaardpinnen gebruiken. Daarom geven we in de code expliciet `Wire.begin(21, 22)` op.

### 3. Hoe gebruik je een bibliotheek?

Een Arduino-bibliotheek bevat code voor de communicatie met de sensor. Voor deze les installeren we **BH1750 by Christopher Laws**. De standaard `Wire`-bibliotheek regelt het I²C-verkeer.

## 🛠️ Opdracht

### Stap 1 – Sluit de sensor aan (15 minuten)

**Haal eerst de USB-kabel uit de ESP32.**

![Aansluitschema ESP32 BH1750](images/les-08/esp32-bh1750-schema.svg)

*Afbeelding 2. Elektrisch schema voor een 3,3V-compatibele BH1750-module; kijk altijd naar de pinlabels op jouw eigen onderdeel.*

| BH1750 | ESP-32D |
|---|---|
| **VCC** | **3V3** |
| **GND** | **GND** |
| **SDA** | **GPIO21** |
| **SCL** | **GPIO22** |
| **ADDR** (indien aanwezig) | Niet aansluiten; standaardadres gebruiken |

Controleer de vier verbindingen. Sluit daarna pas de USB-kabel aan.

### Stap 2 – Bibliotheek installeren (10 minuten)

1. Open **Arduino IDE 2 → Library Manager**.
2. Zoek op **BH1750**.
3. Installeer **BH1750 by Christopher Laws**.
4. De `Wire`-bibliotheek is inbegrepen bij de ESP32-boardondersteuning.

### Stap 3 – Licht meten (20 minuten)

Maak een nieuwe sketch:

```cpp
#include <Wire.h>
#include <BH1750.h>

BH1750 lichtsensor;

void setup() {
  Serial.begin(115200);

  Wire.begin(21, 22);  // SDA, SCL

  if (!lichtsensor.begin(BH1750::CONTINUOUS_HIGH_RES_MODE)) {
    Serial.println("BH1750 niet gevonden. Controleer de bedrading.");
    while (true) {
      delay(1000);
    }
  }
}

void loop() {
  if (lichtsensor.measurementReady()) {
    float lux = lichtsensor.readLightLevel();

    if (lux < 0) {
      Serial.println("Lichtmeting mislukt.");
    } else {
      Serial.print("Lichtsterkte: ");
      Serial.print(lux, 1);
      Serial.println(" lux");
    }
  }

  delay(500);
}
```

Open de **Serial Monitor op 115200 baud**. Houd je hand even boven de sensor en schijn daarna met een lampje in de buurt.

**Uitleg:**

- `Wire.begin(21, 22)` start I²C op de juiste pinnen.
- `lichtsensor.begin(...)` start de BH1750.
- `measurementReady()` controleert of een meting gereed is.
- `readLightLevel()` leest de waarde in lux.
- Een negatieve waarde behandelen we als een foutmelding.

![Serial Monitor met luxwaarden](images/les-08/lux-serial-monitor.svg)

*Afbeelding 3. De waarden zijn voorbeelden; jouw metingen verschillen.*

### Stap 4 – Doe een klein lichtonderzoek (15 minuten)

Meet de lichtsterkte onder drie omstandigheden:

| Situatie | Jouw meting |
|---|---|
| Sensor in normaal lokaallicht | ... lux |
| Sensor afgedekt met karton | ... lux |
| Lampje in de buurt van sensor | ... lux |

**Bespreek:** wanneer is de waarde het laagst? Waarom? Houd de sensor tijdens elke meting ongeveer in dezelfde richting.

### Stap 5 – Zelfstandig programmeren (20 minuten)

**Opdracht A (verplicht):** laat je programma `Te donker!` tonen wanneer de lichtwaarde onder **100 lux** komt en anders `Voldoende licht.`.

**Opdracht B (verplicht):** maak een `const float LICHT_GRENS` en pas de grenswaarde aan op basis van jouw metingen. Laat aan de docent zien dat de waarschuwing verandert wanneer je de sensor afdekt.

Controleer eerst of de meting geldig is voordat je een vergelijking uitvoert.

## ⭐ Bonusopdracht

Sluit een **rode LED met een serieweerstand van 220 of 330 Ω** aan zoals in les 2 op GPIO18. Laat deze LED branden wanneer er te weinig licht is en uitgaan wanneer het licht genoeg is.

> [!TIP]
> Combineer `digitalWrite()` met jouw lichtgrens. Verander de bedrading uitsluitend als de USB-kabel los is.

## 💻 GitHub

**Optioneel:** bewaar je code in je eigen repository in `smart-technology/les-08`. Een GitHub-upload is niet verplicht.

## 🤖 AI-opdracht

Vraag een AI-assistent wat het verschil is tussen **lux** en de **ruwe ADC-waarde** van les 7. Controleer de uitleg met de theorie.

## 🧠 Reflectie

Bespreek kort: waarom hebben SDA en SCL allebei een aparte functie? En waarom zou een lamp met een lichtsensor nuttig kunnen zijn?

## 📤 Inleveren

**Lever één bestand in:** `les8_licht.ino` via Canvas.

Je programma moet:

1. de BH1750 uitlezen via I²C op **GPIO21 en GPIO22**;
2. geldige lichtwaarden in **lux** tonen;
3. ongeldige metingen herkenbaar afhandelen;
4. op basis van een instelbare grens `Te donker!` of `Voldoende licht.` tonen.

**Laat de sensor tijdens de les werken bij donker en licht.** Geen screenshot of verslag vereist.

## 🧠 Kennischeck

1. Wat meet de BH1750?
2. Wat betekent lux?
3. Wat doet SDA? En wat doet SCL?
4. Op welke ESP32-pinnen sluiten we SDA en SCL aan?
5. Waarom installeren we de BH1750-bibliotheek?
6. Waarom mogen we meetwaarden niet zonder controle gebruiken?
7. Wat gebeurt er met de luxwaarde wanneer je de sensor afdekt?

## 📚 Samenvatting

Je hebt leren meten met een digitale **I²C-sensor**. Je kunt lichtsterkte in lux uitlezen, analyseren en op basis van een grenswaarde een waarschuwing maken.

In **les 9** verbind je de ESP32 met wifi en maak je een eenvoudige **HTTP-webpagina** die je via je laptop kunt bekijken.

**Naslag:** [BH1750-bibliotheek (Christopher Laws)](https://github.com/claws/BH1750).
