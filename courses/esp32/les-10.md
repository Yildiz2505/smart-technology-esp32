# Les 10 – Eindopdracht: bouw een slimme sensormonitor

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- twee eerder gebruikte sensoren **samen** op één ESP32 aansluiten;
- temperatuur, luchtvochtigheid en lichtsterkte uitlezen;
- meetgegevens via een **lokale webserver** presenteren;
- eenvoudige waarschuwingen programmeren bij ingestelde grenswaarden;
- een prototype **testen, demonstreren en verbeteren**;
- uitleggen welke vervolgstappen mogelijk zijn voor een echt Smart Technology-product.

## 📖 Introductie

In de vorige negen lessen leerde je verschillende onderdelen van de ESP32 kennen. Je programmeerde LED's, een knop en een buzzer, las sensoren uit en maakte een webserver.

Vandaag combineer je die kennis tot één werkend prototype: **een slimme sensormonitor**.

Je systeem meet:

1. **Temperatuur** met de DHT11;
2. **Luchtvochtigheid** met de DHT11;
3. **Lichtsterkte** met de BH1750.

Op de laptop bekijk je deze meetwaarden via een lokale website op de ESP32. Zo heb je een eerste versie van een slimme omgevingsmonitor.

**Benodigd:**

- ESP-32D WiFi+BT, USB-C-datakabel en laptop met Arduino IDE 2;
- een **3,3V-compatibele DHT11-module met drie aansluitingen** (zoals in les 6);
- een **3,3V-compatibele BH1750-module** (zoals in les 8);
- breadboard en jumper wires;
- de Arduino-bibliotheken **DHT sensor library by Adafruit**, **Adafruit Unified Sensor** en **BH1750 by Christopher Laws**;
- een geschikt **2,4GHz-testnetwerk** waarop laptop en ESP32 elkaar lokaal mogen bereiken.

> [!NOTE]
> Dit is een **basisprototype**, geen professioneel gekalibreerd meetsysteem. De DHT11 is beperkt nauwkeurig en de webserver werkt alleen op het lokale testnetwerk. We maken vandaag nog geen 3D-behuizing en gebruiken nog geen database.

## 💡 Theorie

### 1. Van losse onderdelen naar één systeem

![Overzicht van het eindproject](images/les-10/sensormonitor-architectuur.svg)

*Afbeelding 1. Twee sensoren sturen hun gegevens naar de ESP32. De ESP32 biedt de meetgegevens via wifi aan de browser aan.*

De onderdelen werken samen:

| Onderdeel | Functie | Aansluiting |
|---|---|---|
| DHT11 | Temperatuur en luchtvochtigheid | GPIO27 |
| BH1750 | Lichtsterkte in lux | SDA = GPIO21, SCL = GPIO22 |
| ESP32 | Metingen verzamelen en server draaien | Wifi |
| Browser op laptop | Meetwaarden tonen | HTTP |

### 2. Wat is een meetcyclus?

Een **meetcyclus** is een herhaalbare stap waarin je sensoren uitleest en de waarden bewaart. We meten niet continu, maar ongeveer elke **2,5 seconde**.

Waarom? De DHT11 mag niet te snel worden uitgelezen. Ook blijft ons programma overzichtelijk wanneer meten en webverzoeken apart plaatsvinden.

### 3. Wat tonen we bij een storing?

Een sensor kan tijdelijk geen geldige waarde teruggeven. Een goed prototype mag dan geen verzonnen waarde zoals `0 °C` laten zien.

We gebruiken daarom een eenvoudige foutmelding: **'Geen geldige meting'**. Hierdoor kan de gebruiker onderscheiden of de ruimte koud is of de sensor niet goed werkt.

## 🛠️ Opdracht

### Stap 1 – Maak een technisch ontwerp (10 minuten)

Bekijk het schema en bepaal waar je op het breadboard de sensoren plaatst.

![Aansluitschema ESP32 met DHT11 en BH1750](images/les-10/esp32-twee-sensoren.svg)

*Afbeelding 2. Elektrisch schema voor twee 3,3V-compatibele sensormodules. De werkelijke pinvolgorde kan verschillen; volg altijd de labels op je eigen modules.*

**Sluit de USB-kabel eerst af en maak de volgende verbindingen:**

| Van | Naar ESP32 |
|---|---|
| DHT11 VCC | 3V3 |
| DHT11 DATA / OUT | GPIO27 |
| DHT11 GND | GND |
| BH1750 VCC | 3V3 |
| BH1750 GND | GND |
| BH1750 SDA | GPIO21 |
| BH1750 SCL | GPIO22 |

De twee sensoren delen **3V3** en **GND**. Gebruik de breadboardrails waar die geschikt voor zijn; controleer of de voedingsrails nergens onderbroken zijn.

> [!WARNING]
> Gebruik alleen modules waarvan je hebt gecontroleerd dat ze op **3,3 V** werken en geen hoger signaal naar de ESP32 sturen. Houd alle elektronische onderdelen droog. Laat bij twijfel de docent eerst de schakeling controleren.

### Stap 2 – Test beide sensoren (15 minuten)

Gebruik de kennis uit les 6 en 8:

1. Controleer of je alle benodigde bibliotheken hebt.
2. Upload de gecombineerde voorbeeldcode hieronder.
3. Bekijk eerst de waarden in de **Serial Monitor**.
4. Controleer of DHT11 én BH1750 geldige waarden leveren.

### Stap 3 – Bouw de complete sensormonitor (25 minuten)

Maak een nieuwe sketch. De volgende basiscode verbindt de sensoren met een webserver. **Pas het testnetwerk aan op je eigen situatie.**

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <Wire.h>
#include <DHT.h>
#include <BH1750.h>

const char* WIFI_NAAM = "JOUW_TESTNETWERK";
const char* WIFI_WACHTWOORD = "JOUW_TESTWACHTWOORD";

const int DHT_PIN = 27;
#define DHT_TYPE DHT11

DHT dht(DHT_PIN, DHT_TYPE);
BH1750 lichtsensor;
WebServer server(80);

float temperatuur = NAN;
float vochtigheid = NAN;
float lux = NAN;
unsigned long laatsteMeting = 0;
const unsigned long MEET_INTERVAL = 2500;
bool lichtsensorWerkt = false;

String toonWaarde(float waarde, String eenheid) {
  if (isnan(waarde)) return "Geen geldige meting";
  return String(waarde, 1) + " " + eenheid;
}

void meetSensoren() {
  temperatuur = dht.readTemperature();
  vochtigheid = dht.readHumidity();

  if (lichtsensorWerkt && lichtsensor.measurementReady()) {
    float nieuweLux = lichtsensor.readLightLevel();
    lux = (nieuweLux >= 0) ? nieuweLux : NAN;
  }

  Serial.print("Temperatuur: ");
  Serial.println(toonWaarde(temperatuur, "C"));
  Serial.print("Luchtvochtigheid: ");
  Serial.println(toonWaarde(vochtigheid, "%"));
  Serial.print("Lichtsterkte: ");
  Serial.println(toonWaarde(lux, "lux"));
  Serial.println("-------------------");
}

void startpagina() {
  String html = "<!doctype html><html><head>";
  html += "<meta name='viewport' content='width=device-width'>";
  html += "<meta charset='utf-8'>";
  html += "<meta http-equiv='refresh' content='5'>";
  html += "<title>Mijn sensormonitor</title></head><body>";
  html += "<h1>Smart Technology - Sensormonitor</h1>";

  html += "<p>Temperatuur: " + toonWaarde(temperatuur, "°C") + "</p>";
  html += "<p>Luchtvochtigheid: " + toonWaarde(vochtigheid, "%") + "</p>";
  html += "<p>Lichtsterkte: " + toonWaarde(lux, "lux") + "</p>";

  html += "<p><a href='/status'>Bekijk JSON</a></p>";
  html += "</body></html>";

  server.send(200, "text/html; charset=utf-8", html);
}

void statusJson() {
  String json = "{";

  json += "\"temperatuur\":";
  json += isnan(temperatuur) ? "null" : String(temperatuur, 1);

  json += ",\"luchtvochtigheid\":";
  json += isnan(vochtigheid) ? "null" : String(vochtigheid, 1);

  json += ",\"licht_lux\":";
  json += isnan(lux) ? "null" : String(lux, 1);

  json += "}";
  server.send(200, "application/json", json);
}

void setup() {
  Serial.begin(115200);

  dht.begin();
  Wire.begin(21, 22);

  lichtsensorWerkt =
      lichtsensor.begin(BH1750::CONTINUOUS_HIGH_RES_MODE);

  if (!lichtsensorWerkt) {
    Serial.println("BH1750 niet gevonden.");
  }

  // Eerst sensoren lezen; daarna wifi verbinden
  delay(2000);
  meetSensoren();
  laatsteMeting = millis();

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_NAAM, WIFI_WACHTWOORD);

  unsigned long start = millis();
  while (WiFi.status() != WL_CONNECTED &&
         millis() - start < 15000) {
    delay(500);
    Serial.print(".");
  }

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("\nGeen wifi, controleer je testnetwerk.");
    return;
  }

  Serial.print("\nOpen op de laptop: http://");
  Serial.println(WiFi.localIP());

  server.on("/", startpagina);
  server.on("/status", statusJson);
  server.begin();
}

void loop() {
  if (millis() - laatsteMeting >= MEET_INTERVAL) {
    laatsteMeting = millis();
    meetSensoren();
  }

  if (WiFi.status() == WL_CONNECTED) {
    server.handleClient();
  }
}
```

**Wat moet je zien?** In de Serial Monitor verschijnen drie meetwaarden. Wanneer wifi verbonden is, toont de ESP32 zijn IP-adres.

Open op je laptop `http://<het-IP-adres-van-je-ESP32>/`. De pagina ververst zichzelf ongeveer elke vijf seconden. Op `/status` zie je dezelfde gegevens als JSON.

![Voorbeeld website sensormonitor](images/les-10/sensormonitor-dashboard.svg)

*Afbeelding 3. Een voorbeeld van de lokale webpagina. De getoonde cijfers zijn illustratief.*

### Stap 4 – Maak je prototype slimmer (20 minuten)

**Opdracht A (verplicht):** voeg op de website een eigen projecttitel toe, bijvoorbeeld `Monitor lokaal B1.12`.

**Opdracht B (verplicht):** voeg bij de temperatuur een eenvoudige waarschuwing toe:

- boven **25 °C**: `Waarschuwing: warme ruimte`;
- bij **25 °C of lager**: `Temperatuur normaal`;
- bij een ongeldige meting: `Temperatuur onbekend`.

Je mag hiervoor een `if/else` in `startpagina()` gebruiken. Voeg ook een zichtbaar bericht voor een mislukte lichtmeting toe als dat nodig is.

**Test:** pas tijdelijk de grenswaarde aan, zodat je beide meldingen kunt demonstreren; zet hem daarna terug op 25 °C.

### Stap 5 – Demonstratie en afronding (15 minuten)

Laat aan de docent zien dat je prototype werkt:

1. De ESP32 meet temperatuur, luchtvochtigheid en lichtsterkte.
2. De waarden verschijnen in de Serial Monitor.
3. De browser toont de webpagina via wifi.
4. `/status` geeft dezelfde metingen als JSON.
5. Een waarschuwing verandert wanneer je de grenswaarde of meetomstandigheden aanpast.

Bespreek één verbetering die je voor een **volgende versie** zou willen maken.

## ⭐ Bonusopdracht

Breid je monitor uit met één van de volgende onderdelen:

- **Bodemvochtsensor:** toon de analoge meting van les 7 op de webpagina.
- **LED-waarschuwing:** laat een LED op GPIO18 branden wanneer het te donker is.
- **Extra HTTP-route:** voeg een pagina `/info` toe met de naam van je project en de sensoren die je gebruikt.

**Kies één bonus; alles tegelijk hoeft niet.**

## 💻 GitHub

Bewaar je definitieve, opgeschoonde code bij voorkeur in je eigen repository. Zet **nooit echte wifiwachtwoorden** in een publieke repository. Bewaar voor het inleveren alleen de plaatsaanduidingen `JOUW_TESTNETWERK` en `JOUW_TESTWACHTWOORD`.

## 🤖 AI-opdracht

Laat een AI-assistent één verbetering voorstellen die jouw sensormonitor betrouwbaarder maakt. Controleer of de verbetering technisch klopt en of je begrijpt welke code je zou moeten aanpassen. Je hoeft hier geen afzonderlijk verslag van te maken.

## 🧠 Reflectie

Bespreek kort:

1. Welke onderdelen uit de vorige lessen heb je opnieuw gebruikt?
2. Wat gebeurt er met de webpagina als één sensor geen geldige waarde geeft?
3. Welke uitbreiding zou jouw prototype nuttig maken voor een echte gebruiker?

## 📤 Inleveren

**Lever één bestand in:** `les10_sensormonitor.ino` via Canvas.

Het bestand moet:

1. de DHT11 en BH1750 combineren;
2. de meetwaarden tonen in de Serial Monitor;
3. een **lokale webpagina** aanbieden met de meetwaarden;
4. een JSON-route `/status` aanbieden;
5. een temperatuurwaarschuwing bij **meer dan 25 °C** tonen.

**Laat het werkende prototype tijdens de les aan de docent zien.**

> [!WARNING]
> Verwijder vóór het inleveren alle **echte wifiwachtwoorden** uit de sketch. Lever geen screenshot of verslag in. Een GitHub-link is optioneel.

## 🧠 Kennischeck

1. Wat is de functie van de DHT11?
2. Welke meetwaarde levert de BH1750?
3. Welke aansluitingen delen de twee sensoren?
4. Waarom gebruiken we `millis()` om de metingen te plannen?
5. Wat doet de route `/status`?
6. Waarom stuurt de server soms `null` in plaats van een getal?
7. Wat is het verschil tussen een werkend prototype en een definitief product?

## 📚 Samenvatting

Gefeliciteerd! Je hebt de **tien lessen Smart Technology Basis** afgerond. Je hebt geleerd om de ESP32 te programmeren, componenten aan te sluiten, sensoren uit te lezen, gegevens te verwerken en een webpagina aan te bieden.

Met dit prototype heb je de basis om later verder te bouwen aan bijvoorbeeld een **slimme kweekkas**, een **alarmsysteem**, een **luchtkwaliteitsmonitor** of een ander project dat je zelf ontwerpt.

In de vervolgreeksen leer je onder meer **motoren, pompen en ventilatoren aansturen** en **behuizingen 3D-ontwerpen**.
