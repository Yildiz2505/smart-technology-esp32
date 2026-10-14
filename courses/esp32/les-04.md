# Les 4 – Afstand meten met de HC-SR04

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen hoe een ultrasone afstandssensor werkt;
- de aansluitingen **VCC, TRIG, ECHO en GND** herkennen;
- een 5V-signaal veilig geschikt maken voor een ESP32-ingang met een **spanningsdeler**;
- een afstand in centimeters berekenen en tonen in de Serial Monitor;
- met een `if/else`-statement reageren wanneer een voorwerp dichterbij komt;
- herkennen wanneer een meting mislukt en een foutmelding tonen.

## 📖 Introductie

In les 3 reageerde de ESP32 op een drukknop. In deze les hoeft niemand op een knop te drukken: de ESP32 meet zelf hoe ver een voorwerp weg staat.

We gebruiken hiervoor de **HC-SR04**, een ultrasone afstandssensor met twee ronde onderdelen. Zo'n sensor kun je bijvoorbeeld gebruiken voor een parkeerhulp, een afstandswaarschuwing of een eenvoudig alarmsysteem.

**Benodigd:**

- ESP-32D-board en USB-C-datakabel;
- eigen laptop met Arduino IDE 2;
- breadboard en jumper wires;
- **HC-SR04** ultrasone afstandssensor (standaard 5V-uitvoering);
- één weerstand van **2,2 kΩ** en één weerstand van **3,3 kΩ** (voor de spanningsdeler);
- meetlint of liniaal en een vlak voorwerp zoals een boek.

> [!WARNING]
> De gebruikelijke HC-SR04 heeft **5V voeding en een 5V ECHO-signaal**. Een ESP32-GPIO is **niet 5V-tolerant**. Sluit ECHO daarom **nooit rechtstreeks** aan op GPIO19. Gebruik de spanningsdeler hieronder of een geschikte level shifter.

## 💡 Theorie

### 1. Hoe meet een ultrasone sensor afstand?

De HC-SR04 verstuurt een korte serie geluidspulsen die mensen niet kunnen horen. Het geluid weerkaatst op een voorwerp en komt terug.

De sensor meet hoe lang het geluid onderweg is. De ESP32 kan daarmee berekenen hoe ver het voorwerp weg staat.

![Uitleg ultrasone afstandsmeting](images/les-04/ultrasoon-werking.svg)

*Afbeelding 1. Het geluid gaat naar het voorwerp én weer terug. Daarom delen we de totale afgelegde afstand door twee.*

### 2. Welke pinnen heeft de sensor?

| Pin op HC-SR04 | Betekenis | Naar de ESP32 |
|---|---|---|
| VCC | Voeding voor de sensor | **5V** van het board, alleen als de pin duidelijk als 5V/VIN is aangeduid en daarvoor geschikt is |
| TRIG | Start de meting | **GPIO5** |
| ECHO | Geeft de terugreistijd als puls | Via spanningsdeler naar **GPIO19** |
| GND | Gemeenschappelijke massa | **GND** |

> [!NOTE]
> Gebruik **niet** de 3V3-pin om een standaard 5V-HC-SR04 te voeden. Controleer met de docent welk contact op jullie HW-394-board de juiste **5V-voedingspin** is. Is dat niet zeker, sluit de sensor dan nog niet aan.

### 3. Waarom een spanningsdeler?

De sensor geeft op ECHO ongeveer **5 volt**. De ESP32 mag dat niet rechtstreeks op een GPIO-ingang ontvangen.

Met twee weerstanden maak je van dat signaal ongeveer **3 volt**:

- van **ECHO naar het meetpunt**: **2,2 kΩ**;
- van **meetpunt naar GND**: **3,3 kΩ**;
- van **meetpunt naar GPIO19**: een jumper wire.

Rekenvoorbeeld:

`5 V × 3,3 kΩ / (2,2 kΩ + 3,3 kΩ) = 3,0 V`

![Veilige spanningsdeler voor ECHO](images/les-04/echo-spanningsdeler.svg)

*Afbeelding 2. Alleen het **meetpunt** tussen de weerstanden mag met GPIO19 worden verbonden.*

### 4. Hoe rekenen we de tijd om naar centimeters?

De geluidssnelheid bij kamertemperatuur is ongeveer **0,0343 centimeter per microseconde**.

`afstand in cm = (tijd in microseconden × 0,0343) / 2`

We delen door **2** omdat het geluid heen én terug gaat. De meting is een schatting en kan veranderen door de vorm van het voorwerp en de omgeving.

## 🛠️ Opdracht

### Stap 1 – Bouw de schakeling (20 minuten)

**Haal de USB-kabel uit de ESP32 voordat je begint.**

![Aansluitschema ESP32 en HC-SR04](images/les-04/esp32-hcsr04-schema.svg)

*Afbeelding 3. Het volledige **elektrische schema** met de vier sensoraansluitingen en de spanningsdeler. Controleer de labels op je eigen sensor en ESP32; een schema toont niet de fysieke positie van de gaten in het breadboard.*

**Maak deze verbindingen:**

1. **HC-SR04 VCC → 5V/VIN-voedingspin** van de ESP32, uitsluitend als deze op jullie board als geschikte 5V-pin is bevestigd.
2. **HC-SR04 GND → ESP32 GND**.
3. **HC-SR04 TRIG → ESP32 GPIO5**.
4. **HC-SR04 ECHO → één kant van de 2,2 kΩ-weerstand**.
5. **Andere kant 2,2 kΩ → meetpunt** op het breadboard.
6. **Meetpunt → één kant van de 3,3 kΩ-weerstand**.
7. **Andere kant 3,3 kΩ → GND**.
8. **Meetpunt → ESP32 GPIO19**.

Een meetpunt maak je door meerdere draadjes in **dezelfde elektrisch verbonden breadboardrij** te plaatsen. De sensor-ECHO en GPIO19 zitten dus **niet** zonder weerstand aan elkaar.

> [!WARNING]
> Laat de docent de schakeling controleren **voordat** je de USB-kabel aansluit. Let extra op dat de **2,2 kΩ bovenaan** (tussen ECHO en meetpunt) en de **3,3 kΩ onderaan** (tussen meetpunt en GND) zit. Controleer de kleurcodes of meet de weerstanden indien mogelijk met een multimeter.

### Stap 2 – Meet afstand met Arduino-code (15 minuten)

Maak een nieuwe sketch en gebruik deze code:

```cpp
const int TRIG_PIN = 5;
const int ECHO_PIN = 19;

void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  digitalWrite(TRIG_PIN, LOW);
}

void loop() {
  // Stuur een TRIG-puls van 10 microseconden
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Wacht maximaal 30.000 microseconden op ECHO
  unsigned long duur = pulseIn(ECHO_PIN, HIGH, 30000UL);

  if (duur == 0) {
    Serial.println("Geen geldige meting");
  } else {
    float afstandCm = duur * 0.0343 / 2.0;

    Serial.print("Afstand: ");
    Serial.print(afstandCm, 1);
    Serial.println(" cm");
  }

  delay(250);
}
```

**Wat doet de code?**

- `digitalWrite()` start de meting via TRIG.
- `delayMicroseconds(10)` houdt de TRIG-puls 10 microseconden hoog.
- `pulseIn()` meet de lengte van de ECHO-puls. De timeout voorkomt dat het programma te lang blijft wachten.
- `float afstandCm` bewaart een afstand met decimalen.
- `Serial.print()` toont de uitkomst in de Serial Monitor.

Open de **Serial Monitor op 115200 baud** en houd een boek voor de sensor.

### Stap 3 – Test de meting (10 minuten)

Houd een vlak voorwerp op verschillende afstanden vóór de sensor. Noteer of de getoonde afstand ongeveer klopt.

| Afstand met liniaal | Wat toont de Serial Monitor? |
|---|---|
| Ongeveer 10 cm | ... cm |
| Ongeveer 30 cm | ... cm |
| Ongeveer 60 cm | ... cm |

De waarden hoeven niet precies overeen te komen. Let erop dat je recht voor de sensor meet en houd het voorwerp stil.

> [!TIP]
> Zie je steeds 'Geen geldige meting'? Controleer eerst VCC/GND, GPIO5/GPIO19 en vooral de ECHO-spanningsdeler. Houd ook een voldoende groot en vlak voorwerp vóór de sensor.

### Stap 4 – Zelfstandig programmeren (20 minuten)

**Opdracht A (verplicht):** laat de ESP32 met `if/else` een tekst tonen:

- bij een **geldige afstand kleiner dan 50 cm**: `LET OP: te dichtbij!`
- bij een **geldige afstand van 50 cm of meer**: `Afstand is veilig.`
- bij een **ongeldige meting**: `Geen geldige meting`

**Opdracht B (verplicht):** verander de grenswaarde van **50 cm naar 30 cm**. Controleer met een liniaal of je programma op de nieuwe grenswaarde reageert.

**Test:** laat tijdens de les aan de docent zien dat je programma bij dichtbij en verder weg een verschillend bericht geeft.

## ⭐ Bonusopdracht

Maak een functie `meetAfstandCm()` die de sensor uitleest en een afstand retourneert. Geef bij een mislukte meting bijvoorbeeld `-1` terug. Laat `loop()` op basis van het resultaat een passende melding afdrukken.

> [!NOTE]
> Voor een betrouwbare meting kun je later meerdere metingen middelen of heel afwijkende metingen wegfilteren. Dat hoeft vandaag nog niet.

## 💻 GitHub

**Optioneel:** bewaar je sketch in je eigen repository in een map `smart-technology/les-04`. Een GitHub-upload is voor deze les niet verplicht.

## 🤖 AI-opdracht

Vraag een AI-assistent waarom we de ECHO-pin van een standaard HC-SR04 **niet rechtstreeks op een ESP32-GPIO** aansluiten. Controleer of de uitleg het verschil tussen **5V en 3,3V** duidelijk maakt. Je hoeft niets in te leveren.

## 🧠 Reflectie

Bespreek kort met een medestudent:

1. Waarom delen we de berekende afstand door twee?
2. Wat gebeurt er als een voorwerp niet goed reflecteert?
3. Waarom gebruiken we een timeout bij `pulseIn()`?

Je hoeft dit niet schriftelijk in te leveren.

## 📤 Inleveren

**Lever één bestand in:** `les4_afstand.ino` via de Canvas-opdracht van les 4.

Je programma moet:

1. de sensor via **GPIO5 (TRIG)** en **GPIO19 (ECHO via spanningsdeler)** uitlezen;
2. de afstand in centimeters in de Serial Monitor tonen;
3. bij de **grenswaarde van 30 cm** de juiste waarschuwing geven;
4. een mislukte meting herkenbaar melden, zonder die als nul centimeter te behandelen.

**Laat de werkende schakeling tijdens de les aan de docent zien.**

> [!NOTE]
> Geen foto's, screenshots of reflectieverslag nodig. De bonusopdracht is vrijwillig.

## 🧠 Kennischeck

1. Waarvoor gebruikt de HC-SR04 ultrasoon geluid?
2. Welke functie hebben de TRIG- en ECHO-pinnen?
3. Waarom mag ECHO niet direct op een ESP32-GPIO worden aangesloten?
4. Wat doen de twee weerstanden van de spanningsdeler?
5. Waarom delen we de gemeten geluidstijd door twee?
6. Wat betekent het als `pulseIn()` met een timeout de waarde 0 teruggeeft?
7. Hoe reageert je code als de gemeten afstand 25 cm is en de grens op 30 cm staat?

## 📚 Samenvatting

Je hebt een nieuwe sensor leren aansluiten en met je ESP32 de afstand tot een voorwerp gemeten. Je begrijpt waarom het **5V ECHO-signaal veilig moet worden aangepast naar ongeveer 3V** en hoe je met `if/else` een waarschuwing programmeert.

In **les 5** leer je een buzzer aansluiten en geluidssignalen programmeren.
