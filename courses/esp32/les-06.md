# Les 6 – Temperatuur en luchtvochtigheid meten met DHT11

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat **temperatuur** en **relatieve luchtvochtigheid** zijn;
- een DHT11-sensormodule veilig aansluiten op de ESP32;
- een Arduino-bibliotheek installeren en gebruiken;
- temperatuur en luchtvochtigheid uitlezen in de **Serial Monitor**;
- controleren of een sensormeting geldig is;
- met een `if/else`-statement een waarschuwing tonen bij een ingestelde temperatuur.

## 📖 Introductie

In de vorige lessen stuurde je een LED en een buzzer aan en mat je afstand. Vandaag gebruik je een sensor die **twee verschillende waarden** verzamelt: temperatuur en luchtvochtigheid.

Je kunt deze metingen gebruiken voor een slimme plantenmonitor, een weerstation of de bewaking van een ruimte.

**Benodigd:**

- ESP-32D WiFi+BT (HW-394) met USB-C-datakabel;
- je eigen laptop met Arduino IDE 2;
- breadboard en jumper wires;
- **DHT11-module met drie aansluitingen (VCC, DATA/OUT, GND)** die volgens de specificaties geschikt is voor **3,3 V**.

> [!WARNING]
> Niet alle DHT11-uitvoeringen hebben dezelfde pinvolgorde. Er bestaan modules met drie aansluitingen en losse sensoren met vier pootjes. Controleer **de opschriften en de specificaties** van jouw module. Sluit een als **5V-only** gemarkeerde module niet zomaar op een ESP32 aan. Gebruik voor deze les een bevestigde 3,3V-compatibele module.

## 💡 Theorie

### 1. Wat meet de DHT11?

De DHT11 is een digitale sensor met twee meetwaarden:

| Meetwaarde | Eenheid | Betekenis |
|---|---|---|
| **Temperatuur** | Graden Celsius (°C) | Hoe warm of koud de lucht is |
| **Relatieve luchtvochtigheid** | Procent (%) | Hoeveel waterdamp de lucht bevat ten opzichte van de maximale hoeveelheid bij die temperatuur |

Bij **60% relatieve luchtvochtigheid** is de lucht niet voor 60% uit water samengesteld. Het percentage beschrijft hoe dicht de lucht bij verzadiging met waterdamp zit.

De DHT11 is een **eenvoudige, niet erg nauwkeurige sensor**. Gebruik hem om te leren meten, niet voor metingen waarbij hoge nauwkeurigheid of veiligheid noodzakelijk is.

![Temperatuur en luchtvochtigheid](images/les-06/temperatuur-luchtvochtigheid.svg)

*Afbeelding 1. Een DHT11-sensor geeft temperatuur in °C en relatieve luchtvochtigheid in % door.*

### 2. Digitaal meten en bibliotheken

De DHT11 verstuurt zijn meetgegevens als een **digitaal signaal**. Je hoeft het communicatieprotocol niet zelf te programmeren: een **bibliotheek (library)** bevat code die dat al voor je doet.

Wij gebruiken de bibliotheken **DHT sensor library by Adafruit** en **Adafruit Unified Sensor**.

Een bibliotheek gebruik je met `#include`. Je roept daarna functies aan om de waarden te lezen.

### 3. Waarom niet continu meten?

Een DHT11 is niet bedoeld om honderden keren per seconde uitgelezen te worden. In deze les wachten we **minimaal twee seconden** tussen uitlezingen.

Een sensor kan soms geen geldige waarde teruggeven, bijvoorbeeld door een verkeerde aansluiting of een communicatiefout. Daarom controleren we de waarde voordat we die gebruiken.

## 🛠️ Opdracht

### Stap 1 – Sluit de DHT11 aan (15 minuten)

**Haal eerst de USB-kabel uit de ESP32.**

Voor deze les gebruiken we **GPIO27** voor het datasignaal.

![Aansluitschema ESP32 met DHT11](images/les-06/esp32-dht11-schema.svg)

*Afbeelding 2. Schematische verbindingen voor een **3,3V-compatibele DHT11-module met drie pinnen**. Controleer de werkelijke volgorde op je eigen onderdeel.*

| DHT11-module | ESP-32D-board |
|---|---|
| **VCC / +** | **3V3** |
| **DATA / OUT / S** | **GPIO27** |
| **GND / −** | **GND** |

1. Zoek de drie labels op **jouw eigen module**.
2. Verbind **VCC** met **3V3** op de ESP32.
3. Verbind **DATA/OUT** met **GPIO27**.
4. Verbind **GND** met **GND**.
5. Laat je bedrading controleren en sluit de USB-kabel daarna weer aan.

> [!NOTE]
> De meeste geschikte **3-pins DHT11-modules** hebben al de benodigde pull-upweerstand op het printplaatje. Een **kale DHT11 met vier pinnen** heeft doorgaans een externe pull-up van ongeveer 10 kΩ tussen DATA en 3V3 nodig. Dit lesvoorbeeld is specifiek voor de 3-pins module.

### Stap 2 – Installeer de bibliotheken (10 minuten)

1. Open **Arduino IDE 2**.
2. Open de **Library Manager** via het bibliotheekpictogram links.
3. Zoek **DHT sensor library** en installeer **DHT sensor library by Adafruit**.
4. Installeer ook **Adafruit Unified Sensor** als deze niet automatisch als afhankelijkheid wordt geïnstalleerd.

> [!TIP]
> Krijg je een foutmelding dat `DHT.h` niet gevonden wordt? Controleer of je de juiste bibliotheek hebt geïnstalleerd. Herstart Arduino IDE indien nodig.

### Stap 3 – Meet temperatuur en luchtvochtigheid (20 minuten)

Maak een nieuwe sketch en gebruik deze code:

```cpp
#include <DHT.h>

const int DHT_PIN = 27;
#define DHT_TYPE DHT11

DHT dht(DHT_PIN, DHT_TYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
}

void loop() {
  // Wacht minstens twee seconden tussen metingen
  delay(2000);

  float temperatuur = dht.readTemperature();
  float luchtvochtigheid = dht.readHumidity();

  if (isnan(temperatuur) || isnan(luchtvochtigheid)) {
    Serial.println("Meting mislukt. Controleer de sensor.");
    return;
  }

  Serial.print("Temperatuur: ");
  Serial.print(temperatuur, 1);
  Serial.println(" °C");

  Serial.print("Luchtvochtigheid: ");
  Serial.print(luchtvochtigheid, 1);
  Serial.println(" %");

  Serial.println("-----------------------");
}
```

**Wat doet deze code?**

- `#include <DHT.h>` laadt de bibliotheek.
- `DHT dht(DHT_PIN, DHT_TYPE);` maakt een sensorobject.
- `dht.begin()` initialiseert de sensor.
- `dht.readTemperature()` leest de temperatuur in °C.
- `dht.readHumidity()` leest de relatieve luchtvochtigheid in %.
- `isnan(...)` controleert of een waarde **geen geldig getal** is.
- `return;` stopt deze uitvoering van `loop()`, waarna de volgende meting weer kan plaatsvinden.

Upload het programma. Open de **Serial Monitor op 115200 baud**.

![Voorbeeld van Serial Monitor met DHT11](images/les-06/dht11-serial-monitor.svg)

*Afbeelding 3. Voorbeeldwaarden; jouw temperatuur en luchtvochtigheid zullen anders zijn.*

### Stap 4 – Probeer verschillende metingen (10 minuten)

Bekijk de sensorwaarden gedurende ongeveer één minuut. Houd de sensor op afstand van je hand en kijk of de temperatuur stabiel blijft. Houd daarna je hand in de buurt (zonder elektrische contacten aan te raken) en vergelijk de waarden.

De waarden kunnen langzaam veranderen. Een DHT11 is geen snelle of precieze meetsensor.

### Stap 5 – Zelfstandig programmeren (20 minuten)

**Opdracht A (verplicht):** voeg een `if/else`-statement toe:

- bij een **geldige temperatuur boven 25 °C**: toon `LET OP: warme ruimte!`;
- bij een **geldige temperatuur van 25 °C of lager**: toon `Temperatuur is normaal.`.

**Opdracht B (verplicht):** voeg ook een waarschuwing voor relatieve luchtvochtigheid toe:

- bij een **geldige luchtvochtigheid lager dan 40%**: toon `Lucht is droog.`;
- anders: toon `Luchtvochtigheid is voldoende.`.

**Belangrijk:** controleer **eerst** of beide meetwaarden geldig zijn. Vergelijk geen ongeldige meting met een grenswaarde.

**Test:** pas tijdelijk de temperatuurgrens in je code aan zodat je beide berichten kunt controleren. Zet hem daarna terug op **25 °C**.

## ⭐ Bonusopdracht

Maak een functie `toonMeetwaarden(float temperatuur, float luchtvochtigheid)` die de waarden overzichtelijk afdrukt. Roep de functie vanuit `loop()` aan.

Extra uitdaging: bereken het gemiddelde van **drie geldige temperatuurmetingen** en toon het resultaat. Zorg dat mislukte metingen niet meetellen.

## 💻 GitHub

**Optioneel:** bewaar je sketch in je eigen GitHub-repository in de map `smart-technology/les-06`. Een GitHub-upload is voor deze les niet verplicht.

## 🤖 AI-opdracht

Vraag een AI-assistent: **Wat betekent relatieve luchtvochtigheid van 60%?** Controleer of de uitleg klopt met wat je in de theorie hebt geleerd. Je hoeft niets in te leveren.

## 🧠 Reflectie

Bespreek kort met een medestudent:

1. Wat is het verschil tussen temperatuur en relatieve luchtvochtigheid?
2. Waarom gebruiken we voor de DHT11 een bibliotheek?
3. Wat gebeurt er als een sensor een ongeldige waarde teruggeeft?

Je hoeft de antwoorden niet schriftelijk in te leveren.

## 📤 Inleveren

**Lever één bestand in:** `les6_dht11.ino` via de Canvas-opdracht van les 6.

Je programma moet:

1. de DHT11 op **GPIO27** uitlezen;
2. temperatuur en luchtvochtigheid in de **Serial Monitor** tonen;
3. ongeldige metingen herkennen;
4. een temperatuurwaarschuwing tonen bij **meer dan 25 °C**;
5. een waarschuwing tonen bij **minder dan 40% luchtvochtigheid**.

**Laat de werkende metingen tijdens de les aan de docent zien.**

> [!NOTE]
> Geen foto's, screenshots of schriftelijke reflectie nodig. De bonusopdracht is vrijwillig.

## 🧠 Kennischeck

1. Wat meet een DHT11?
2. In welke eenheden tonen we temperatuur en luchtvochtigheid?
3. Wat betekent relatieve luchtvochtigheid van 60%?
4. Welke drie aansluitingen heeft onze DHT11-module?
5. Waarom gebruiken we een Arduino-bibliotheek?
6. Waarom wachten we minstens twee seconden tussen metingen?
7. Wat controleert `isnan()`?
8. Wanneer toont je programma `LET OP: warme ruimte!`?

## 📚 Samenvatting

Je hebt geleerd om met een **digitale sensor** temperatuur en luchtvochtigheid te meten. Je hebt een Arduino-bibliotheek geïnstalleerd, ongeldige meetwaarden gecontroleerd en waarschuwingen geprogrammeerd met `if/else`.

In **les 7** ga je de vochtigheid van potgrond meten met een **analoge bodemvochtsensor**.

**Naslag:** [Adafruit DHT-sensorbibliotheek](https://github.com/adafruit/DHT-sensor-library) en [Adafruit Unified Sensor](https://github.com/adafruit/Adafruit_Sensor).
