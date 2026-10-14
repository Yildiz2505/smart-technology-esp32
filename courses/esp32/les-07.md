# Les 7 – Bodemvocht meten met een capacitieve sensor

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat een **capacitieve bodemvochtsensor** meet;
- het verschil tussen **analoge** en **digitale** invoer beschrijven;
- een analoge sensor veilig aansluiten op **GPIO34 (ADC1)**;
- met `analogRead()` een ruwe meetwaarde uitlezen;
- met metingen in droge en vochtige grond een **eigen grenswaarde** bepalen;
- een eenvoudige melding voor droge of vochtige potgrond programmeren.

## 📖 Introductie

In les 6 las je temperatuur en luchtvochtigheid uit met een digitale DHT11. Vandaag meet je iets anders: **hoe vochtig de grond van een plant is**.

Een capacitieve bodemvochtsensor geeft een analoog signaal. De ESP32 zet dat om in een getal dat jij in je programma kunt gebruiken.

**Benodigd:**

- ESP-32D WiFi+BT (HW-394) en USB-C-datakabel;
- je eigen laptop met Arduino IDE 2;
- capacitieve **analoge** bodemvochtsensor die op **3,3 V** werkt en een analoge uitgang heeft (AOUT, AO of SIG);
- breadboard en jumper wires;
- een potje met **droge potgrond** en een potje met **licht vochtige potgrond**, of twee bakjes met verschillend vochtige grond;
- papier of een opvangbakje om de elektronica droog te houden.

> [!WARNING]
> Gebruik een **3,3V-compatibele sensor** waarvan de analoge uitgang bij 3,3V-voeding binnen het bereik **0–3,3 V** blijft. Sluit nooit een 5V-analoog signaal rechtstreeks aan op een ESP32-GPIO. Controleer altijd de opschriften op jouw sensor; de pinvolgorde verschilt per uitvoering.

## 💡 Theorie

### 1. Wat meet een capacitieve sensor?

Een capacitieve bodemvochtsensor reageert op veranderingen in de elektrische eigenschappen van de omgeving rond het meetgedeelte. Natte en droge potgrond leveren daardoor meestal verschillende meetwaarden op.

De sensor meet **niet rechtstreeks een exact vochtpercentage van de grond**. Om betekenis aan de getallen te geven, moet je de sensor vergelijken met bekende situaties.

![Capacitieve bodemvochtsensor in droge en vochtige grond](images/les-07/bodemvocht-uitleg.svg)

*Afbeelding 1. De sensor geeft bij droge en vochtige potgrond meestal verschillende ruwe waarden. Welke waarde hoger is, kan per sensor verschillen.*

### 2. Analoog tegenover digitaal

| Digitale invoer | Analoge invoer |
|---|---|
| Werkt met toestanden zoals HIGH en LOW | Heeft een bereik aan waarden |
| Voorbeeld: een drukknop | Voorbeeld: een bodemvochtsensor |
| Je gebruikt vaak `digitalRead()` | Je gebruikt `analogRead()` |

De ESP32 gebruikt een **ADC** (Analog-to-Digital Converter) om de analoge spanning om te zetten naar een getal.

Bij de standaard **12-bits** resolutie levert `analogRead()` doorgaans een ruwe waarde tussen **0 en 4095**. De werkelijke waarden hangen onder andere af van sensor, voedingsspanning en ADC-instellingen. Het getal is **niet automatisch een percentage**.

### 3. Waarom gebruiken we GPIO34?

Op de ESP32-WROOM-32D is **GPIO34** een analoge ingang van **ADC1**. Deze pin kan signalen lezen, maar **niet als uitgang** werken.

We gebruiken daarom:

- **3V3** voor de sensorvoeding;
- **GND** als gemeenschappelijk referentiepunt;
- **GPIO34** voor het analoge signaal.

### 4. Wat is kalibreren?

Kalibreren betekent dat je eigen **referentiewaarden** vastlegt. We vergelijken:

1. Een meting in **droge potgrond**.
2. Een meting in **licht vochtige potgrond**.

Daarna kies je een grenswaarde waarmee je programma kan bepalen of de grond voor deze oefening droog of vochtig is.

> [!NOTE]
> Een ruwe waarde van bijvoorbeeld 2400 betekent niet dat de grond '24% vochtig' is. En de grenswaarde van jouw sensor hoeft niet hetzelfde te zijn als die van een medestudent.

## 🛠️ Opdracht

### Stap 1 – Sluit de sensor aan (15 minuten)

**Maak eerst de USB-kabel los van de ESP32.** Houd het breadboard, de ESP32 en de USB-aansluiting weg van water en vochtige aarde.

![Aansluitschema ESP32 en bodemvochtsensor](images/les-07/esp32-bodemvocht-schema.svg)

*Afbeelding 2. Elektrisch aansluitschema voor een geschikte 3,3V-sensor met drie aansluitingen. Controleer de volgorde op je eigen sensor voordat je de draden verbindt.*

| Bodemvochtsensor | ESP-32D-board |
|---|---|
| **VCC / +** | **3V3** |
| **GND / −** | **GND** |
| **AOUT / AO / SIG** | **GPIO34** |

1. Zoek de labels **VCC, GND en AOUT** op jouw sensor.
2. Verbind **VCC met 3V3**.
3. Verbind **GND met GND**.
4. Verbind **AOUT met GPIO34**.
5. Laat indien nodig je aansluiting controleren.
6. Sluit de USB-kabel pas weer aan als alles goed verbonden is.

> [!WARNING]
> Steek alleen het **daarvoor bedoelde meetgedeelte** van de sensor in de aarde. De elektronica, connectoren en bedrading aan de bovenkant moeten droog blijven. Voeg geen water toe terwijl de ESP32 of het breadboard vlak naast de pot ligt.

### Stap 2 – Lees de analoge waarde uit (15 minuten)

Maak in Arduino IDE een nieuwe sketch:

```cpp
const int VOCHT_PIN = 34;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
}

void loop() {
  int meetwaarde = analogRead(VOCHT_PIN);

  Serial.print("Bodemvocht (ruwe waarde): ");
  Serial.println(meetwaarde);

  delay(1000);
}
```

**Wat doet de code?**

- `const int VOCHT_PIN = 34;` kiest de analoge ingang.
- `analogReadResolution(12);` stelt de leesresolutie in op 12 bits.
- `analogRead(VOCHT_PIN)` leest de sensor uit.
- `Serial.println(meetwaarde)` laat het getal zien in de Serial Monitor.
- `delay(1000)` wacht één seconde tussen de metingen.

Open de **Serial Monitor** op **115200 baud**.

![Voorbeeld analoge meetwaarden in de Serial Monitor](images/les-07/bodemvocht-serial-monitor.svg)

*Afbeelding 3. Voorbeeld van ruwe meetwaarden. De getallen zijn illustratief en kunnen op jouw sensor anders zijn.*

### Stap 3 – Vergelijk droge en vochtige grond (15 minuten)

Plaats het meetgedeelte eerst in droge potgrond en noteer een stabiele waarde. Plaats de sensor daarna in licht vochtige grond, op ongeveer dezelfde diepte, en noteer opnieuw een stabiele waarde.

| Situatie | Eigen ruwe meetwaarde |
|---|---|
| Droge potgrond | ... |
| Licht vochtige potgrond | ... |

**Vragen om te bespreken:**

- Is de waarde in vochtige grond hoger of lager?
- Zijn de waarden voldoende verschillend?
- Welke grenswaarde ligt ongeveer tussen jouw twee metingen?

> [!TIP]
> Beweeg de sensor zo min mogelijk tijdens het uitlezen. Gebruik steeds ongeveer dezelfde insteekdiepte. Wacht een paar seconden totdat de waarde stabiel is.

### Stap 4 – Programmeer een vochtmelding (20 minuten)

**Opdracht A (verplicht):** voeg een variabele `grenswaarde` toe met een getal dat je **zelf hebt bepaald uit jouw twee metingen**.

Laat de ESP32 afhankelijk van de ruwe waarde één van deze berichten tonen:

- `LET OP: potgrond is droog.`
- `Potgrond is vochtig.`

**Let op:** controleer eerst of jouw sensor **hogere of lagere waarden** geeft in droge grond. Kies op basis daarvan de juiste vergelijking (`>` of `<`).

**Opdracht B (verplicht):** pas de Serial Monitor-uitvoer aan zodat zowel de **ruwe waarde** als de **droog/vochtig-melding** zichtbaar is.

**Test:** laat de docent zien dat de melding verandert wanneer je de sensor verplaatst tussen de twee soorten potgrond.

> [!NOTE]
> De droog/vochtig-melding is een **eenvoudige proefopstelling**. Voor een echte plant hangt de juiste grenswaarde ook af van de plantensoort, grondsoort en hoe je de sensor hebt gekalibreerd.

## ⭐ Bonusopdracht

Maak een functie `bepaalVochtstatus(int waarde)` die de tekst `"Droog"` of `"Vochtig"` teruggeeft.

**Extra uitdaging:** toon een indicatief percentage tussen 0% en 100% op basis van je eigen droge en vochtige referentiewaarden. Gebruik bijvoorbeeld `map()` en `constrain()`.

> [!WARNING]
> Noem zo'n percentage altijd **indicatief**: het is een schaal tussen jouw twee referentiemetingen, **geen nauwkeurig gemeten waterpercentage** van de aarde.

## 💻 GitHub

**Optioneel:** bewaar je sketch in je eigen GitHub-repository, bijvoorbeeld in `smart-technology/les-07`. Je hoeft voor deze les niets naar GitHub te pushen.

## 🤖 AI-opdracht

Vraag een AI-assistent waarom `analogRead()` een getal tussen 0 en 4095 kan teruggeven. Vergelijk de uitleg met de theorie over de ADC. Controleer of de AI ook uitlegt waarom dat getal **geen direct vochtpercentage** is.

Je hoeft hier niets over in te leveren.

## 🧠 Reflectie

Bespreek kort met een medestudent:

1. Wat is het verschil tussen digitaal en analoog meten?
2. Waarom moet je een bodemvochtsensor kalibreren?
3. Waarom kan je grenswaarde verschillen van die van een medestudent?

Je hoeft dit niet schriftelijk in te leveren.

## 📤 Inleveren

**Lever slechts één bestand in:** `les7_bodemvocht.ino` via de Canvas-opdracht van les 7.

Je programma moet:

1. de sensor uitlezen via **GPIO34** met `analogRead()`;
2. de ruwe meetwaarde tonen in de Serial Monitor;
3. een **eigen grenswaarde** gebruiken die past bij jouw droge en vochtige metingen;
4. de tekst **'Droog'** of **'Vochtig'** laten zien op basis van die grenswaarde.

**Laat tijdens de les de werkende schakeling en beide situaties aan de docent zien.**

> [!NOTE]
> Geen foto's, screenshots of schriftelijk verslag nodig. De bonusopdracht is vrijwillig.

## 🧠 Kennischeck

1. Wat meet een capacitieve bodemvochtsensor?
2. Wat is het verschil tussen een digitale en een analoge ingang?
3. Wat betekent ADC?
4. Welke functie gebruik je om de sensor uit te lezen?
5. Waarom gebruiken we GPIO34 in deze les?
6. Waarom betekent een ruwe meetwaarde van 2500 niet automatisch 25% bodemvocht?
7. Waarom moeten we zowel droge als vochtige potgrond meten?
8. Wanneer gebruik je `>` en wanneer `<` in je vochtcontrole?

## 📚 Samenvatting

Je hebt met de ESP32 een **analoge bodemvochtsensor** uitgelezen en geleerd hoe je een meetwaarde betekenis geeft door te vergelijken met droge en vochtige grond. Je gebruikt nu `analogRead()`, `if/else` en **kalibratie**.

In **les 8** gaan we licht meten met de **BH1750**. Daar maak je kennis met de communicatie via **I²C**.
