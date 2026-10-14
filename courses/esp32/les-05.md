# Les 5 – Geluid maken met een buzzer

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C)

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat een buzzer is en waar je hem gebruikt;
- het verschil tussen een **actieve buzzer** en een **passieve buzzer** uitleggen;
- een kleine **passieve piezo-buzzer** veilig aansluiten op de ESP32;
- het begrip **frequentie (hertz, Hz)** beschrijven;
- met `ledcWriteTone()` tonen van verschillende hoogtes programmeren;
- een kort waarschuwingsgeluid en een eenvoudige sirene maken.

## 📖 Introductie

In les 4 kon je ESP32 de afstand tot een voorwerp meten. Vandaag laat je de ESP32 **geluid maken**. Denk aan een piep bij een deur, een waarschuwing bij een apparaat of een sirene.

Je sluit een kleine passieve buzzer aan op je ESP32. Daarna schrijf je een programma waarmee je verschillende tonen laat horen.

**Benodigd:**

- ESP-32D WiFi+BT en USB-C-datakabel;
- je eigen laptop met Arduino IDE 2;
- breadboard en jumper wires;
- een **kleine passieve piezo-buzzer / piezo-transducer**, geschikt voor directe aansturing met 3,3V-logica en een lage belasting van de GPIO.

> [!WARNING]
> Gebruik voor deze les **geen luidspreker van 4 of 8 Ω**, grote sirene of andere buzzer die meer stroom vraagt dan een ESP32-GPIO veilig kan leveren. Die onderdelen hebben een geschikte driver of versterker nodig. Controleer het type buzzer vóór het aansluiten met je docent.

## 💡 Theorie

### Wat is een buzzer?

Een buzzer is een onderdeel dat geluid kan produceren. Je vindt buzzers bijvoorbeeld in wekkers, timers, meetapparaten en alarmsystemen.

Er zijn twee veelvoorkomende soorten:

| Type | Hoe werkt het? | Geschikt voor deze les? |
|---|---|---|
| **Actieve buzzer** | Maakt vaak vanzelf een vaste toon wanneer je hem van de juiste voeding voorziet | Nee, niet voor verschillende toonhoogtes |
| **Passieve buzzer / piezo** | Heeft een wisselend stuursignaal nodig; de microcontroller bepaalt de toonhoogte | **Ja, mits geschikt voor 3,3V en lage GPIO-belasting** |

![Verschil actieve en passieve buzzer](images/les-05/buzzer-soorten.svg)

*Afbeelding 1. Bij een passieve piezo-buzzer bepaalt de ESP32 de toonfrequentie. Het uiterlijk alleen vertelt niet altijd welk type je hebt; controleer de specificaties.*

### Wat is frequentie?

Een toon bestaat uit trillingen. Het aantal trillingen per seconde heet **frequentie**. We meten dat in **hertz (Hz)**.

- **400 Hz** klinkt relatief laag.
- **800 Hz** klinkt hoger.
- **1200 Hz** klinkt nog hoger.

Een hogere frequentie betekent dus een **hogere toon**. Een hogere frequentie betekent **niet automatisch harder geluid**.

### Hoe maakt de ESP32 een toon?

De ESP32 kan met een ingebouwde functie snel wisselen tussen HIGH en LOW. Dat noemen we een **PWM-signaal**.

In de huidige Arduino-ESP32-boardondersteuning (versie **3.x**) gebruiken we:

- `ledcAttach(pin, frequentie, resolutie)`: stelt de PWM-uitgang in;
- `ledcWriteTone(pin, frequentie)`: laat een toon klinken;
- `ledcWriteTone(pin, 0)`: stopt de toon.

> [!NOTE]
> Deze les gebruikt de Arduino-ESP32 **3.x API** van Espressif. Controleer via de Boards Manager dat `esp32 by Espressif Systems` versie 3.x is geïnstalleerd. Oudere voorbeelden gebruiken soms `ledcSetup()` en `ledcAttachPin()`; die horen bij de vorige API en werken niet in versie 3.x.

## 🛠️ Opdracht

### Stap 1 – Sluit de passieve buzzer aan (15 minuten)

**Haal eerst de USB-kabel uit de ESP32.**

Voor deze les gebruiken we **GPIO25** en **GND**. Het gaat alleen om een kleine, goedgekeurde passieve piezo-buzzer met lage stroombelasting.

1. Zoek de **plus (+)** en **min (−)** van de passieve piezo-buzzer. Sommige transducers zijn niet gepolariseerd; volg dan de productspecificaties.
2. Verbind de **plus/signaalzijde** met **GPIO25**.
3. Verbind de **minzijde** met **GND**.
4. Controleer of de buzzer volgens zijn specificaties direct vanuit een ESP32-GPIO mag worden aangestuurd.
5. Laat de schakeling controleren en sluit de USB-kabel weer aan.

![Aansluitschema ESP32 en passieve piezo-buzzer](images/les-05/esp32-buzzer-schema.svg)

*Afbeelding 2. Vereenvoudigd elektrisch schema: GPIO25 naar de passieve piezo-transducer en de andere aansluiting naar GND. **Alleen voor een geschikt laagvermogen-piezo-onderdeel.***

> [!TIP]
> Heb je een **buzzer-module met drie aansluitingen** (VCC, GND, SIG) of een actieve buzzer? Sluit deze dan **niet zomaar volgens dit tweepinsschema aan**. Vraag aan de docent welk type je hebt.

### Stap 2 – Laat je eerste pieptoon horen (15 minuten)

Maak in Arduino IDE een nieuwe sketch:

```cpp
const int BUZZER_PIN = 25;

void setup() {
  Serial.begin(115200);

  if (!ledcAttach(BUZZER_PIN, 1000, 8)) {
    Serial.println("PWM kon niet worden gestart");
    while (true) {
      delay(1000);
    }
  }
}

void loop() {
  ledcWriteTone(BUZZER_PIN, 1000);  // 1000 Hz
  delay(500);                       // halve seconde

  ledcWriteTone(BUZZER_PIN, 0);     // stil
  delay(1000);                      // één seconde pauze
}
```

**Test je programma:** je hoort een piep van een halve seconde, gevolgd door een seconde stilte. Dit herhaalt zich.

**Wat doet de code?**

- `BUZZER_PIN = 25`: hiermee kiezen we GPIO25.
- `ledcAttach(...)`: maakt de pin geschikt voor een PWM-geluidssignaal.
- `ledcWriteTone(BUZZER_PIN, 1000)`: laat een toon van 1000 Hz horen.
- `ledcWriteTone(BUZZER_PIN, 0)`: maakt de buzzer stil.
- `delay(...)`: bepaalt hoe lang het geluid en de pauze duren.

### Stap 3 – Maak drie korte pieptonen (15 minuten)

**Opdracht A (verplicht):** pas je programma aan zodat je elke cyclus **drie korte pieptonen** hoort.

- Toonhoogte: **800 Hz**
- Iedere piep: **200 milliseconden**
- Pauze tussen de pieptonen: **200 milliseconden**
- Na de derde pieptoon: **2 seconden stilte**

> [!TIP]
> Maak een functie `piep(int frequentie, int duur)`. Dan hoef je dezelfde regels niet steeds opnieuw te schrijven.

### Stap 4 – Maak een sirenegeluid (20 minuten)

**Opdracht B (verplicht):** laat de buzzer afwisselend twee toonhoogtes produceren:

1. **600 Hz gedurende 400 ms**
2. **1200 Hz gedurende 400 ms**
3. Herhaal dit patroon drie keer.
4. Maak de buzzer daarna **2 seconden stil**.
5. Begin opnieuw.

Gebruik een `for`-lus of schrijf de benodigde regels zelf. Werk bij voorkeur met een kleine functie om herhaling te voorkomen.

**Controle:** je hoort een afwisselend laag en hoog geluid dat na drie herhalingen even stopt.

> [!WARNING]
> Houd het volume prettig in het klaslokaal. Een kleine piezo is bedoeld voor eenvoudige signalen, niet voor een harde alarmsirene. Sluit geen versterker of grote luidspreker rechtstreeks aan op GPIO25.

## ⭐ Bonusopdracht

Maak een **oplopende sirene**. Laat de frequentie bijvoorbeeld in stappen veranderen van **500 Hz tot 1500 Hz**, met steeds 100 ms tussen de stappen. Laat de frequentie daarna weer dalen.

Gebruik een `for`-lus om dit zo kort mogelijk te programmeren.

## 💻 GitHub

**Optioneel:** bewaar je code in je eigen repository, bijvoorbeeld in `smart-technology/les-05`. Een GitHub-upload is voor deze les niet verplicht.

## 🤖 AI-opdracht

Vraag een AI-assistent: **Wat is het verschil tussen een actieve buzzer en een passieve buzzer?** Controleer of de uitleg klopt met de theorie én het type onderdeel dat jij gebruikt. Je hoeft hier niets over in te leveren.

## 🧠 Reflectie

Bespreek kort met een medestudent:

1. Wat hoor je als je van 600 Hz naar 1200 Hz gaat?
2. Wat doet `ledcWriteTone(BUZZER_PIN, 0)`?
3. Waarom mogen we niet zomaar een grote luidspreker rechtstreeks aansluiten?

Je hoeft niets schriftelijk in te leveren.

## 📤 Inleveren

**Lever één bestand in:** `les5_buzzer.ino` via de Canvas-opdracht van les 5.

Je programma moet:

1. **GPIO25** gebruiken om een geschikte passieve piezo-buzzer aan te sturen;
2. een toon kunnen starten en weer stoppen;
3. het **sirenepatroon van opdracht B** afspelen: afwisselend 600 Hz en 1200 Hz, drie keer, gevolgd door 2 seconden stilte.

**Laat je werkende buzzer tijdens de les aan de docent horen.**

> [!NOTE]
> Geen screenshots, foto's of reflectieverslag nodig. De bonusopdracht is vrijwillig.

## 🧠 Kennischeck

1. Wat doet een buzzer?
2. Wat is het verschil tussen een actieve en passieve buzzer?
3. Wat betekent frequentie en in welke eenheid meten we die?
4. Welke toon klinkt hoger: 600 Hz of 1200 Hz?
5. Wat doet `ledcWriteTone(BUZZER_PIN, 0)`?
6. Waarom moet je het type buzzer controleren voordat je deze aansluit?
7. Wat is het voordeel van een functie `piep()` bij meerdere tonen?

## 📚 Samenvatting

Je hebt geleerd om een kleine passieve buzzer aan te sturen met een ESP32. Je kunt toonhoogtes programmeren, pauzes maken en meerdere tonen combineren tot een eenvoudig sirenegeluid.

In **les 6** ga je temperatuur en luchtvochtigheid meten met een sensor.

**Naslag:** [Officiële Espressif-documentatie over LEDC](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/ledc.html).
