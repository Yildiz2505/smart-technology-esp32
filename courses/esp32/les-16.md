# Les 16 – Automatische plantenwatering

**Smart Technology · Software Developer niveau 4 · 90 minuten**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), bodemvochtsensor, relais met dompelpomp

## 🎯 Leerdoelen

Na deze les kun je:

- de bodemvochtsensor uit les 7 combineren met de pomp uit les 14;
- een automatisch irrigatiesysteem bouwen met hysterese;
- de pomp beveiligen met een maximale aan-tijd (beveiliging tegen overstroming);
- je systeem testen en de drempels verantwoorden;
- uitleggen waarom een pomp nooit onbeperkt mag lopen.

## 📖 Introductie

In les 7 mat je bodemvocht en in les 14 schakelde je een pomp. Nu bouw je een **automatische plantenwatering**: is de aarde te droog, dan geeft het systeem kort water; is hij vochtig genoeg, dan stopt de pomp vanzelf.

Net als in les 15 gebruik je **hysterese**, want vochtmeting schommelt: zonder marge zou de pomp constant klik-aan-klik-uit gaan.

Er komt nog een beveiliging bij: als de sensor kapot is of de pot lekt, mag de pomp **nooit** eindeloos doorlopen. Daarom bouw je een **maximale aan-tijd** in.

**Benodigd:** je opstelling van les 14 (relais + pomp + waterbakje), plus de bodemvochtsensor uit les 7 en een pot met droge aarde.

## 💡 Theorie

### Het beslisdiagram

1. meet bodemvocht;
2. is het vocht **lager dan de aan-drempel** (droog) → pomp aan;
3. is het vocht **hoger dan de uit-drempel** (vochtig) → pomp uit;
4. anders: status ongewijzigd;
5. loopt de pomp langer dan de **maximale aan-tijd** → pomp uit + alarm in de Serial Monitor.

| Instelling | Voorbeeld | Waarom |
|---|---|---|
| Aan-drempel | vocht < 1800 | aarde voelt droog aan |
| Uit-drempel | vocht > 2200 | aarde is goed vochtig |
| Max aan-tijd | 10 s | beveiliging tegen overstroming |

Let op: bij de meeste sensors betekent een **lage** waarde een **droge** aarde. Test jouw sensor eerst, zoals in les 7.

### Maximale aan-tijd

Gebruik `millis()` om bij te houden hoelang de pomp al loopt. Bij elke beslissing vergelijk je `millis() - pompStartTijd` met de maximale aan-tijd.

## 🛠️ Opdracht

### Stap 1 – Aansluiten

- Bodemvochtsensor: analoge uitgang (AO) op **GPIO32**, VCC op 3V3, GND op GND.
- Relais met pomp zoals in les 14 (IN op GPIO33).

### Stap 2 – Testprogramma

```cpp
const int PIN_VOCHT = 32;
const int PIN_RELAIS = 33;
const bool RELAIS_AAN_LAAG = true;

const int DREMPEL_AAN = 1800;   // droger dan dit -> water geven
const int DREMPEL_UIT = 2200;   // natter dan dit -> stoppen
const unsigned long MAX_AAN_TIJD = 10000; // 10 seconden

bool pompAan = false;
unsigned long pompStart = 0;

void zetPomp(bool aan) {
  pompAan = aan;
  if (RELAIS_AAN_LAAG) {
    digitalWrite(PIN_RELAIS, aan ? LOW : HIGH);
  } else {
    digitalWrite(PIN_RELAIS, aan ? HIGH : LOW);
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(PIN_RELAIS, OUTPUT);
  zetPomp(false);
}

void loop() {
  int vocht = analogRead(PIN_VOCHT);
  Serial.print("Vocht: ");
  Serial.println(vocht);

  if (pompAan) {
    if (vocht >= DREMPEL_UIT) {
      zetPomp(false);
      Serial.println("Vochtig genoeg: pomp UIT");
    } else if (millis() - pompStart >= MAX_AAN_TIJD) {
      zetPomp(false);
      Serial.println("ALARM: maximale aan-tijd bereikt!");
    }
  } else {
    if (vocht <= DREMPEL_AAN) {
      zetPomp(true);
      pompStart = millis();
      Serial.println("Droog: pomp AAN");
    }
  }
  delay(1000);
}
```

### Stap 3 – Zelfstandig aanpassen

**Opdracht A (verplicht):** kalibreer jouw sensor. Meet de waarde bij droge aarde en bij vochtige aarde, en kies daaruit je eigen drempels. Noteer ze als commentaar bovenin je sketch.

**Opdracht B (verplicht):** voeg een pauze toe na het water geven: nadat de pomp is gestopt, wacht het systeem **60 seconden** voordat hij opnieuw mag starten (het water moet eerst in de aarde zakken). Gebruik `millis()`, geen `delay(60000)`.

## ⭐ Bonusopdracht

Print elke minuut een overzicht: `Vocht: 1950 | Pomp: uit | Laatste waterbeurt: 14:32`.

## 💻 GitHub

Bewaar je opgeschoonde sketch in je eigen repository, bijvoorbeeld in `smart-technology/les-16`.

## 🤖 AI-opdracht

Vraag een AI-assistent wat er mis kan gaan als je de aan-drempel en uit-drempel omdraait (aan > uit). Voorspel wat de pomp doet en controleer je voorspelling door de drempels in de code om te wisselen **met de pomp buiten het water**. Leg daarna aan een medestudent uit wat er gebeurde.

## 🧠 Reflectie

1. Waarom heeft dit systeem een maximale aan-tijd?
2. Wat betekent een lage sensorwaarde bij de meeste vochtsensors?
3. Waarom wacht je 60 seconden na een waterbeurt?

## 📤 Inleveren

**Lever één bestand in:** `les16_watering.ino`, inclusief je gekalibreerde drempels als commentaar. Laat het systeem werken aan de docent zien.

## 🧠 Kennischeck

1. Welke twee drempels gebruik je en wat doen ze?
2. Hoe bescherm je tegen een kapotte sensor die droog "ziet"?
3. Wat doet `millis() - pompStart`?
4. Waarom geen `delay(60000)` voor de pauze?

## 📚 Samenvatting

Je kunt nu sensordata gebruiken om zelfstandig een actuator te sturen, inclusief beveiliging. In **les 17** geef je jezelf de regie terug: je bouwt een webserver om alle actuatoren met de telefoon of laptop te bedienen.