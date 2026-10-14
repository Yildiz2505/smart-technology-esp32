# Les 9 – ESP32 verbinden met wifi en HTTP

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat **wifi**, een **IP-adres**, een **server** en een **client** zijn;
- de ESP32 verbinden met een geschikt **2,4GHz-wifinetwerk**;
- een eenvoudige HTTP-server op de ESP32 starten;
- vanuit de browser een webpagina op de ESP32 openen;
- zelf twee HTTP-routes maken, waaronder één die JSON terugstuurt;
- uitleggen waarom je wachtwoorden nooit publiceert in GitHub.

## 📖 Introductie

Tot nu toe las je sensorwaarden uit in de **Serial Monitor** op de laptop. Maar een slim apparaat hoeft niet aan een computer vast te zitten om zijn gegevens te tonen.

In deze les maakt de ESP32 via wifi verbinding met een netwerk. Vervolgens open je op je laptop een **webpagina die door de ESP32 zelf wordt aangeboden**.

**Benodigd:**

- ESP-32D WiFi+BT met USB-C-datakabel;
- laptop met Arduino IDE 2;
- toegang tot een **2,4GHz-wifinetwerk**, waarvan de laptop en ESP32 elkaar lokaal mogen bereiken;
- bij voorkeur een eigen test-hotspot of een door de docent beschikbaar gesteld onderwijsnetwerk.

> [!WARNING]
> Veel schoolnetwerken gebruiken een inlogportaal, WPA-Enterprise of apparaat-isolatie. Daarmee werkt dit lesvoorbeeld niet altijd. Test vooraf een apart 2,4GHz-testnetwerk waarop laptop en ESP32 met elkaar mogen communiceren. Vraag bij gebruik van een hotspot toestemming volgens de schoolregels.

## 💡 Theorie

### 1. Hoe werkt de verbinding?

![Van browser naar ESP32 via wifi](images/les-09/wifi-http-overzicht.svg)

*Afbeelding 1. De laptop (client) vraagt via het lokale netwerk een webpagina op van de ESP32 (server).*

| Begrip | Betekenis |
|---|---|
| **Wifi** | Draadloze verbinding met het netwerk |
| **SSID** | Naam van het wifinetwerk |
| **IP-adres** | Het netwerkadres van een apparaat, bijvoorbeeld `192.168.1.35` |
| **HTTP** | Protocol waarmee browser en server verzoeken en antwoorden uitwisselen |
| **Server** | Apparaat dat een verzoek ontvangt en antwoord geeft |
| **Client** | Browser of programma dat een verzoek doet |
| **Route** | Een pad zoals `/` of `/status` |
| **JSON** | Tekstformaat om gestructureerde gegevens uit te wisselen |

### 2. Wat gebeurt er bij een HTTP-verzoek?

Je browser vraagt bijvoorbeeld:

`GET http://192.168.1.35/`

De ESP32 antwoordt met een HTML-pagina.

Bij:

`GET http://192.168.1.35/status`

kan de ESP32 in plaats daarvan JSON terugsturen, zoals:

```json
{
  "apparaat": "ESP32",
  "status": "online"
}
```

Dat is handig: in les 10 sturen we via dezelfde techniek **echte meetwaarden** naar een webpagina.

### 3. Wifi en veiligheid

- Een ESP32 in deze opstelling gebruikt **HTTP, niet HTTPS**. Gebruik hem daarom uitsluitend op een **vertrouwd, lokaal testnetwerk**.
- Maak **geen portforwarding** naar het internet.
- Zet een echt wifiwachtwoord **nooit in een publiek GitHub-repository**.
- Gebruik voor de les een testnetwerk, en verwijder echte inloggegevens voordat je code deelt.

## 🛠️ Opdracht

### Stap 1 – Maak verbinding met wifi (20 minuten)

Open Arduino IDE en maak een nieuwe sketch:

```cpp
#include <WiFi.h>

const char* WIFI_NAAM = "JOUW_TESTNETWERK";
const char* WIFI_WACHTWOORD = "JOUW_TESTWACHTWOORD";

void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_NAAM, WIFI_WACHTWOORD);

  Serial.print("Verbinden met wifi");

  unsigned long startTijd = millis();
  while (WiFi.status() != WL_CONNECTED &&
         millis() - startTijd < 15000) {
    Serial.print(".");
    delay(500);
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWifi verbonden!");
    Serial.print("IP-adres: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("\nWifi niet verbonden. Controleer je netwerk.");
  }
}

void loop() {
}
```

Vervang de twee voorbeeldteksten door de gegevens van het **goedgekeurde testnetwerk**. Upload de code en open de Serial Monitor op **115200 baud**.

> [!NOTE]
> Een gewone ESP32-WROOM-32D gebruikt **2,4GHz-wifi**. Een netwerk dat alleen 5 GHz aanbiedt, werkt niet. Wanneer er na 15 seconden geen verbinding is, kijk dan naar SSID, wachtwoord, netwerkbeveiliging en signaal.

### Stap 2 – Maak een eigen webserver (20 minuten)

We gebruiken nu de ingebouwde `WebServer.h`-bibliotheek. Vervang de vorige sketch door:

```cpp
#include <WiFi.h>
#include <WebServer.h>

const char* WIFI_NAAM = "JOUW_TESTNETWERK";
const char* WIFI_WACHTWOORD = "JOUW_TESTWACHTWOORD";

WebServer server(80);

void pagina() {
  String html = "<!doctype html><html><head>";
  html += "<meta name='viewport' content='width=device-width'>";
  html += "<title>Mijn ESP32</title></head><body>";
  html += "<h1>Hallo vanaf mijn ESP32!</h1>";
  html += "<p>Mijn eerste Smart Technology-webpagina.</p>";
  html += "<p><a href='/status'>Bekijk JSON-status</a></p>";
  html += "</body></html>";

  server.send(200, "text/html; charset=utf-8", html);
}

void status() {
  server.send(200, "application/json",
              "{\"apparaat\":\"ESP32\",\"status\":\"online\"}");
}

void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_NAAM, WIFI_WACHTWOORD);

  Serial.print("Wifi verbinden");
  unsigned long startTijd = millis();
  while (WiFi.status() != WL_CONNECTED &&
         millis() - startTijd < 15000) {
    Serial.print(".");
    delay(500);
  }

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("\nGeen wifi. Herstart na controle.");
    return;
  }

  Serial.println("\nWifi verbonden!");
  Serial.print("Open: http://");
  Serial.println(WiFi.localIP());

  server.on("/", pagina);
  server.on("/status", status);
  server.begin();
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    server.handleClient();
  }
}
```

Open op je laptop het IP-adres dat je in de Serial Monitor ziet, bijvoorbeeld `http://192.168.1.35/`. Gebruik het **echte adres van jouw ESP32**, niet blind het voorbeeld.

![Voorbeeld van de ESP32-webpagina](images/les-09/esp32-webpagina.svg)

*Afbeelding 2. Voorbeeld van wat de browser toont wanneer je de webserver opent.*

**Controleer twee routes:**

- `http://<jouw-ip-adres>/` → HTML-pagina.
- `http://<jouw-ip-adres>/status` → JSON-status.

> [!TIP]
> Laadt de pagina niet? Controleer of je laptop en ESP32 op hetzelfde lokale netwerk zitten, het IP-adres klopt en het netwerk communicatie tussen apparaten toestaat.

### Stap 3 – Zelfstandig programmeren (20 minuten)

**Opdracht A (verplicht):** wijzig de kop van de webpagina naar `Welkom bij mijn ESP32` en voeg je eigen projectnaam toe.

**Opdracht B (verplicht):** maak een derde route `/info` die JSON terugstuurt met minimaal:

- `"project": "Smart Technology"`
- `"les": 9`

Voeg op de startpagina een link naar deze route toe.

### Stap 4 – Test en laat zien (15 minuten)

Controleer met een browser of alle drie de routes werken.

Test ook een onbekende route, bijvoorbeeld `/bestaatniet`. De standaardwebserver geeft dan doorgaans een foutmelding terug. Bespreek waarom een client niet iedere route zomaar kan gebruiken.

## ⭐ Bonusopdracht

Voeg een route `/tijd` toe die `millis()` terugstuurt in JSON, bijvoorbeeld:

`{"uptime_ms":123456}`

Hiermee laat je een **actuele waarde** zien die verandert wanneer je de pagina opnieuw opent.

## 💻 GitHub

**Optioneel:** bewaar de code in je eigen repository, maar **verwijder eerst de echte wifi-naam en het wachtwoord**. Laat alleen de plaatsaanduidingen `JOUW_TESTNETWERK` en `JOUW_TESTWACHTWOORD` staan. Voor deze les is een GitHub-upload niet verplicht.

## 🤖 AI-opdracht

Vraag een AI-assistent het verschil uit te leggen tussen een **HTTP-client** en een **HTTP-server**. Controleer de uitleg met het gedrag van jouw laptop en ESP32.

## 🧠 Reflectie

Bespreek kort: waarom heeft een ESP32 een IP-adres nodig? En wat is het verschil tussen de webpagina op `/` en de JSON-gegevens op `/status`?

## 📤 Inleveren

**Lever één bestand in:** `les9_wifi_http.ino` via Canvas.

Je programma moet:

1. verbinden met een geschikt lokaal 2,4GHz-wifinetwerk;
2. het IP-adres tonen in de Serial Monitor;
3. op `/` een eigen HTML-startpagina aanbieden;
4. op `/status` en `/info` een JSON-antwoord geven.

**Laat de webpagina en beide JSON-routes in de browser aan de docent zien.**

> [!WARNING]
> Verwijder **eerst de echte wifi-inloggegevens** uit het in te leveren bestand. Toon de werkende versie alleen aan de docent op je eigen laptop. Screenshots en een reflectieverslag zijn niet nodig.

## 🧠 Kennischeck

1. Waarvoor staat HTTP?
2. Wat is een IP-adres?
3. Is je browser de client of de server?
4. Wat betekent een GET-verzoek?
5. Wat is het verschil tussen HTML en JSON?
6. Waarom werkt dit voorbeeld mogelijk niet op een schoolnetwerk met apparaat-isolatie?
7. Waarom publiceer je nooit een echt wifiwachtwoord op GitHub?

## 📚 Samenvatting

Je hebt je ESP32 met wifi verbonden en een eenvoudige **lokale HTTP-server** gemaakt. Je kunt een HTML-pagina tonen én JSON-gegevens aanbieden.

In **les 10** combineer je dit met sensoren in een eigen **Smart Technology-sensormonitor**.

**Naslag:** [Espressif Arduino-ESP32 Wi-Fi API](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/wifi.html).
