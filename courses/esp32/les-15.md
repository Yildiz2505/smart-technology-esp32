# Les 15 – Automatische ventilatie op temperatuur

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), DHT11, L9110S met ventilator

## 🎯 Leerdoelen

Na deze les kun je:

- een sensor en een actuator combineren tot een automatisch systeem;
- uitleggen wat **hysterese** is en waarom je dat nodig hebt;
- een automatisch ventilatiesysteem bouwen met de DHT11;
- de drempelwaarden zelf kiezen en verantwoorden;
- je systeem testen met een vast testscenario.

## 📖 Introductie

In les 6 mat je temperatuur en in les 12 zette je een ventilator aan. Nu combineren we die twee: **wordt het te warm, dan gaat de ventilator automatisch aan**. Zo werkt een echte thermostaat, een serverruimte-koeling of een kasklimaat.

Maar er zit een addertje onder het gras. Stel: de ventilator gaat aan bij 25,0 °C. Door de koeling zakt de temperatuur naar 24,9 °C → ventilator uit → temperatuur stijgt naar 25,0 °C → aan → uit → aan... De ventilator **flakkert** aan en uit. Dat heet *katerend* schakelen en is slecht voor motor én relais.

**Benodigd:** de opstelling van les 12 (driver + ventilator), plus je DHT11-sensor uit les 6.

## 💡 Theorie

### Hysterese

De oplossing is **hysterese**: een marge tussen de aan- en de uitschakelwaarde.

| Begrip | Betekenis |
|---|---|
| Drempel aan | Temperatuur waarbij de ventilator inschakelt (bijv. 26,0 °C) |
| Drempel uit | Temperatuur waarbij hij uitschakelt (bijv. 24,0 °C) |
| Hysterese | Het verschil (hier 2,0 °C) |

Zolang de temperatuur tussen de twee drempels zit, **verandert de ventilator niet** van staat. Daardoor schakelt hij rustig en slijt hij niet.

### Regelcyclus

Een automatisch systeem leest telkens de sensor en beslist dan:

1. meet temperatuur;
2. vergelijk met de drempels;
3. zet de ventilator aan of uit;
4. wacht een vaste tijd (bijv. 2 s) en herhaal.

## 🛠️ Opdracht

### Stap 1 – Aansluiten

- DHT11: data op **GPIO14**, VCC op 3V3, GND op GND (zoals les 6).
- L9110S met ventilator zoals in les 12 (IA op GPIO26, IB op GND-niveau in code).

### Stap 2 – Testprogramma

```cpp
#include <DHT.h>

const int PIN_DHT = 14;
const int PIN_IA = 26;
const int PIN_IB = 25;

const float DREMPEL_AAN = 26.0;
const float DREMPEL_UIT = 24.0;

DHT dht(PIN_DHT, DHT11);
bool ventilatorAan = false;

void setup() {
  Serial.begin(115200);
  dht.begin();
  pinMode(PIN_IA, OUTPUT);
  pinMode(PIN_IB, OUTPUT);
  digitalWrite(PIN_IA, LOW);
  digitalWrite(PIN_IB, LOW);
}

void loop() {
  float temp = dht.readTemperature();

  if (isnan(temp)) {
    Serial.println("Sensorfout!");
  } else {
    if (!ventilatorAan && temp >= DREMPEL_AAN) {
      ventilatorAan = true;
      Serial.println("Te warm: ventilator AAN");
    } else if (ventilatorAan && temp <= DREMPEL_UIT) {
      ventilatorAan = false;
      Serial.println("Afgekoeld: ventilator UIT");
    }
    digitalWrite(PIN_IA, ventilatorAan ? HIGH : LOW);
    digitalWrite(PIN_IB, LOW);
    Serial.print("Temp: ");
    Serial.print(temp);
    Serial.print(" \u00b0C | Ventilator: ");
    Serial.println(ventilatorAan ? "AAN" : "UIT");
  }
  delay(2000);
}
```

**Let op:** in het programma moet in de Serial-print \u00b0 vervangen worden door het teken \u00b0 zelf; Arduino IDE begrijpt beide, maar kopiëren uit de les kan rare teksten geven.

### Stap 3 – Zelfstandig aanpassen

**Opdracht A (verplicht):** test je systeem door de DHT11 zachtjes in je hand te warmen. Noteer in de Serial Monitor op welke temperaturen de ventilator aan- en uitschakelt. Klopt dat met je drempels?

**Opdracht B (verplicht):** maak de drempels aanpasbaar via **Serial Input**: stuur `aan 27` of `uit 25` om de drempels live te wijzigen zonder het programma opnieuw te uploaden.

## ⭐ Bonusopdracht

Laat de ventilatorsnelheid meeschalen met de temperatuur met `map()`: bij de aan-drempel 40 %, bij 30 °C vol gas (100 %).

## 💻 GitHub

Bewaar je opgeschoonde sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-15`.

## 🤖 AI-opdracht

Vraag een AI-assistent om twee echte voorbeelden van hysterese in huis-tuin-en-keukenapparatuur (bijv. koelkast, verwarming). Controleer de antwoorden: is er echt een marge tussen aan- en uitschakelen? Bespreek met een medestudent.

## 🧠 Reflectie

1. Waarom flakkerde de ventilator zonder hysterese?
2. Wat is het nadeel van een heel grote hysterese?
3. Hoe zorg je ervoor dat een sensorfout niet meteen de ventilator aanzet?

## 📤 Inleveren

**Lever één bestand in:** `les15_ventilatie.ino`. Laat het automatische systeem aan de docent zien, inclusief je gekozen drempels.

## 🧠 Kennischeck

1. Wat is hysterese?
2. Welke twee drempels gebruik je in deze les?
3. Waarom controleer je `isnan(temp)`?
4. Wat betekent "katerend schakelen"?

## 📚 Samenvatting

Je kunt nu een sensor en een actuator combineren tot een automatisch systeem met hysterese. In **les 16** bouw je op dezelfde manier een automatische plantenwatering met bodemvochtsensor en pomp.