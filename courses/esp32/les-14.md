# Les 14 – Een pomp schakelen met een relais

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), relaismodule, 5 V mini-dompelpomp

## 🎯 Leerdoelen

Na deze les kun je:

- uitleggen wat een relais is en wanneer je het gebruikt in plaats van een motordriver;
- een relaismodule veilig aansluiten op de ESP32;
- een 5 V-waterpomp via het relais aan en uit zetten;
- een millis()-gebaseerd interval programmeren (water geven op tijd);
- veilig werken rond water en elektronica.

## 📖 Introductie

Een **relais** is een elektronische schakelaar: een klein signaal van de ESP32 laat in het relais een elektromagneet klikken, waardoor een grotere stroomkring wordt gesloten of geopend. Je hoort en ziet het relais schakelen.

Waarom een relais en geen motordriver? Een dompelpomp heeft veel stroom nodig en hoeft alleen **aan of uit**. Een relais is simpel, goedkoop en scheidt de aansturing volledig van de belasting.

**Benodigd:** ESP-32D-board, relaismodule met optocoupler, 5 V mini-dompelpomp met slang, bakje water, breadboard en jumper wires.

> [!WARNING]
> In deze les schakelen we **uitsluitend laagspanning** (5 V). Sluit nooit netspanning (230 V) aan. Dat is levensgevaarlijk en niet toegestaan in deze lessen.

## 💡 Theorie

### Hoe werkt een relais?

| Begrip | Betekenis |
|---|---|
| Relais | Elektronisch aangestuurde schakelaar met een spoel |
| Optocoupler | Onderdeel dat de ESP32 elektrisch scheidt van het relais |
| NO (Normally Open) | Contact dat pas sluit bij inschakeling |
| NC (Normally Closed) | Contact dat opent bij inschakeling |
| COM | Gemeenschappelijke aansluiting van het contact |

We gebruiken de **NO-contacten**: de pomp loopt alleen als de ESP32 het relais inschakelt. Stopt je programma of crasht de ESP32, dan stopt de pomp vanzelf. Dat heet *fail-safe*.

### Meestal actief LAAG

Veel relaismodules schakelen het relais in bij een **LAAG** signaal (relay LOW = aan). Controleer op jouw module het LEDje: brandt het bij `digitalWrite(relaisPin, LOW)`, dan is jouw module actief LAAG.

### Water en elektronica

- Houd de ESP32 en het breadboard droog en apart van het waterbakje.
- Laat de pomp nooit droog draaien: water koelt de pomp.
- Wikkel de connectoren af met isolatietape.

## 🛠️ Opdracht

### Stap 1 – Aansluiten

| Relaismodule | Naar ESP32 |
|---|---|
| VCC | 3V3 |
| GND | GND |
| IN | GPIO33 |

De pomp sluit je aan op de schakelcontacten:

- 5V (VIN) → **COM** van het relais;
- **NO** van het relais → rode pompdraad;
- zwarte pompdraad → GND.

Zorg dat de stroomkring van de pomp loopt: 5V → COM, NO → pomp → GND.

### Stap 2 – Testprogramma

```cpp
const int PIN_RELAIS = 33;
const bool RELAIS_AAN_LAAG = true; // pas aan op je eigen module

void zetPomp(bool aan) {
  if (RELAIS_AAN_LAAG) {
    digitalWrite(PIN_RELAIS, aan ? LOW : HIGH);
  } else {
    digitalWrite(PIN_RELAIS, aan ? HIGH : LOW);
  }
}

void setup() {
  pinMode(PIN_RELAIS, OUTPUT);
  zetPomp(false);
  Serial.begin(115200);
}

void loop() {
  Serial.println("Pomp 3 seconden aan");
  zetPomp(true);
  delay(3000);
  Serial.println("Pomp 10 seconden uit");
  zetPomp(false);
  delay(10000);
}
```

**Let op:** controleer eerst zonder water of je het relais hoort klikken. Zet de pomp pas daarna in het water.

### Stap 3 – Zelfstandig aanpassen

**Opdracht A (verplicht):** vervang `delay()` door een **millis()-planner**: de pomp staat 3 seconden aan en 30 seconden uit. Gebruik twee `unsigned long`-variabelen, zoals je in les 10 deed.

**Opdracht B (verplicht):** voeg een **noodknop** toe (drukknop op GPIO4 naar GND met `INPUT_PULLUP`). Zolang de knop is ingedrukt, staat de pomp **altijd uit**, ook als de planner pomp-tijd zegt. Print `Noodstop actief`.

## ⭐ Bonusopdracht

Tel bij elke pompbeurt het aantal keren dat er water is gegeven en print het totaal: `Water gegeven: 4 keer`.

## 💻 GitHub

Bewaar je opgeschoonde sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-14`.

## 🤖 AI-opdracht

Vraag een AI-assistent waarom we de NO-contacten gebruiken en niet de NC-contacten voor een waterpomp. Controleer: wat gebeurt er met de pomp als je de USB-kabel trekt terwijl de code in de "aan"-fase zit?

## 🧠 Reflectie

1. Wat is het verschil tussen een relais en een motordriver?
2. Waarom is een pomp via NO fail-safe?
3. Waarom gebruiken we geen netspanning in deze lessen?

## 📤 Inleveren

**Lever één bestand in:** `les14_pomp.ino`. Laat de werkende pomp met noodknop aan de docent zien.

## 🧠 Kennischeck

1. Wat betekenen NO en NC?
2. Waarom zou een relais actief LAAG kunnen zijn?
3. Wat is een optocoupler en waarom zit die op de module?
4. Waarom mag een dompelpomp niet droog draaien?

## 📚 Samenvatting

Je kunt nu een apparaat met veel stroom, zoals een waterpomp, veilig schakelen met een relais en een millis()-planner met noodstop. In **les 15** combineer je de ventilator met een temperatuursensor tot een automatische thermostaat.