# Les 17 – Alles bedienen via een webserver

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), servo, driver+ventilator, relais+pomp

## 🎯 Leerdoelen

Na deze les kun je:

- een webserver op de ESP32 draaien met de bibliotheek **WebServer**;
- een eenvoudige HTML-pagina met knoppen serveren vanaf de ESP32;
- via HTTP-verzoeken de servo, ventilator en pomp bedienen;
- uitleggen hoe URL-parameters (`?servo=90`) werken;
- je volledige opstelling bedienen vanaf je telefoon of laptop.

## 📖 Introductie

In les 9 haalde je data van internet met HTTP. Nu draait de rollen om: **de ESP32 wordt zelf een webserver**. Iedereen in hetzelfde Wi-Fi-netwerk kan een pagina openen en met een klik de servo draaien, de ventilator aanzetten of de pomp starten.

Zo werkt het: de browser vraagt `http://<esp-ip>/aan?apparaat=ventilator`. De ESP32 ontvangt dat verzoek, voert de bijbehorende code uit en stuurt een bevestiging terug.

**Benodigd:** je volledige opstelling van de lessen 11–16 (servo, driver+ventilator, relais+pomp) en een telefoon of laptop in hetzelfde Wi-Fi-netwerk.

> [!NOTE]
> Gebruik hetzelfde 2,4 GHz Wi-Fi-netwerk als in les 9. De ESP32 werkt niet met 5 GHz-only netwerken.

## 💡 Theorie

### Webserver op de ESP32

| Begrip | Betekenis |
|---|---|
| WebServer | Bibliotheek die HTTP-verzoeken afhandelt |
| Route | Een URL-pad met bijbehorende functie, bijv. `/aan` |
| URL-parameter | Extra info in de URL, bijv. `?hoek=90` |
| `server.send()` | Stuur HTML of tekst terug naar de browser |
| IP-adres | Het "telefoonnummer" van je ESP32 in het netwerk |

### Beknopte UI

De HTML-pagina in deze les is bewust simpel: een titel en een paar knoppen als links. Hoe je een mooiere interface bouwt leer je in je webdevelopment-lessen; hier gaat het om de techniek erachter.

## 🛠️ Opdracht

### Stap 1 – Alles aansluiten

Sluit aan zoals in de vorige lessen: servo op GPIO13, driver IA op GPIO26, relais op GPIO33, drukknop/noodstop op GPIO4.

### Stap 2 – Testprogramma

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <ESP32Servo.h>

const char* WIFI_NAAM = "jouw-netwerk";
const char* WIFI_WACHTWOORD = "jouw-wachtwoord";

const int PIN_SERVO = 13;
const int PIN_IA = 26;
const int PIN_IB = 25;
const int PIN_RELAIS = 33;
const bool RELAIS_AAN_LAAG = true;

WebServer server(80);
Servo mijnServo;
bool ventilatorAan = false;
bool pompAan = false;

void zetPomp(bool aan) {
  pompAan = aan;
  if (RELAIS_AAN_LAAG) {
    digitalWrite(PIN_RELAIS, aan ? LOW : HIGH);
  } else {
    digitalWrite(PIN_RELAIS, aan ? HIGH : LOW);
  }
}

String pagina() {
  String p = "<!DOCTYPE html><html lang='nl'><head><meta charset='utf-8'>";
  p += "<meta name='viewport' content='width=device-width,initial-scale=1'>";
  p += "<title>Smart Technology</title></head><body>";
  p += "<h1>Smart Technology bediening</h1><p>";
  p += "<a href='/servo?hoek=0'>Servo 0</a> | ";
  p += "<a href='/servo?hoek=90'>Servo 90</a> | ";
  p += "<a href='/servo?hoek=180'>Servo 180</a><br><br>";
  p += "<a href='/ventilator?staat=aan'>Ventilator aan</a> | ";
  p += "<a href='/ventilator?staat=uit'>Ventilator uit</a><br><br>";
  p += "<a href='/pomp?staat=aan'>Pomp 3s aan</a> | ";
  p += "<a href='/pomp?staat=uit'>Pomp uit</a>";
  p += "</p></body></html>";
  return p;
}

void handleRoot() {
  server.send(200, "text/html; charset=utf-8", pagina());
}

void handleServo() {
  int hoek = server.arg("hoek").toInt();
  hoek = constrain(hoek, 0, 180);
  mijnServo.write(hoek);
  server.sendHeader("Location", "/");
  server.send(303);
}

void handleVentilator() {
  ventilatorAan = (server.arg("staat") == "aan");
  digitalWrite(PIN_IA, ventilatorAan ? HIGH : LOW);
  digitalWrite(PIN_IB, LOW);
  server.sendHeader("Location", "/");
  server.send(303);
}

void handlePomp() {
  if (server.arg("staat") == "aan") {
    zetPomp(true);
    Serial.println("Pomp via web aan");
  } else {
    zetPomp(false);
  }
  server.sendHeader("Location", "/");
  server.send(303);
}

void setup() {
  Serial.begin(115200);
  pinMode(PIN_IA, OUTPUT);
  pinMode(PIN_IB, OUTPUT);
  pinMode(PIN_RELAIS, OUTPUT);
  zetPomp(false);
  ESP32PWM::allocateTimer(0);
  mijnServo.setPeriodHertz(50);
  mijnServo.attach(PIN_SERVO, 500, 2400);

  WiFi.begin(WIFI_NAAM, WIFI_WACHTWOORD);
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
  Serial.println();
  Serial.print("Verbonden! Open: http://");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.on("/servo", handleServo);
  server.on("/ventilator", handleVentilator);
  server.on("/pomp", handlePomp);
  server.begin();
}

void loop() {
  server.handleClient();
}
```

Vul je eigen Wi-Fi-gegevens in, upload het programma en open het getoonde IP-adres in je browser.

### Stap 3 – Zelfstandig aanpassen

**Opdracht A (verplicht):** voeg een statusregel toe aan de pagina die toont wat aan staat, bijvoorbeeld: `Ventilator: AAN | Pomp: UIT`. Zet de tekst tussen de knoppen.

**Opdracht B (verplicht):** voeg een route `/noodstop` toe die alles direct uitzet (ventilator, pomp, servo naar 90). Voeg ook een knop **Noodstop** op de pagina toe.

## ⭐ Bonusopdracht

Voeg een slider toe (`<input type='range'>`) waarmee je de servo met JavaScript (`fetch('/servo?hoek='+waarde)`) vloeiend kunt bedienen.

## 💻 GitHub

Bewaar je opgeschoonde sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-17`.

## 🤖 AI-opdracht

Vraag een AI-assistent wat de veiligheidsrisico's zijn van een webserver zonder wachtwoord op een schoolnetwerk. Bedenk daarna zelf twee manieren om de pagina beter te beveiligen (tip: HTTP-authenticatie, of alleen verzoeken van jouw eigen toestel toestaan). Bespreek met een medestudent.

## 🧠 Reflectie

1. Wat gebeurt er als je per ongeluk twee keer `server.on("/", ...)` registreert?
2. Hoe geeft de browser een knopklik door aan de ESP32?
3. Waarom is een noodstop-knop op de webpagina handig?

## 📤 Inleveren

**Lever één bestand in:** `les17_webserver.ino`. Laat de docent de pagina op zijn/haar telefoon openen en bedienen.

## 🧠 Kennischeck

1. Op welke poort luistert de webserver standaard?
2. Wat doet `server.arg("hoek")`?
3. Wat betekent de HTTP-statuscode 303 in dit programma?
4. Waarom hoort `server.handleClient()` in `loop()`?

## 📚 Samenvatting

Je kunt nu je complete opstelling bedienen via een eigen webpagina op de ESP32. In **les 18** combineer je alles: sensoren, actuatoren, automatische regeling én webbediening in één eindproject.