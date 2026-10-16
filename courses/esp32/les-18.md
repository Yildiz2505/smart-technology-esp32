# Les 18 – Eindopdracht: de slimme kweekkas

**Smart Technology · Software Developer niveau 4 · 90 minuten + afwerking**  
**Hardware:** ESP-32D WiFi+BT (HW-394, USB-C), DHT11, bodemvochtsensor, LDR of BH1750, servo (luik), driver+ventilator, relais+pomp

## 🎯 Leerdoelen

Na deze les kun je:

- een compleet smart-technology-project zelfstandig opbouwen;
- meerdere sensoren en actuatoren combineren in één programma;
- automatische regeling met hysterese toepassen op temperatuur én vocht;
- je project documenteren, demonstreren en verantwoorden;
- reflecteren op je eigen ontwerpkeuzes.

## 📖 De opdracht

Je bouwt een **slimme kweekkas**: een mini-kas die zijn klimaat zelf regelt en ook handmatig bedienbaar is.

### Eisen (functioneel)

| # | Eis | Bronles |
|---|---|---|
| 1 | Meet temperatuur en luchtvochtigheid (DHT11) | les 6 |
| 2 | Meet bodemvocht van de pot | les 7 |
| 3 | Ventilator gaat automatisch aan bij te hoge temperatuur, met hysterese | les 15 |
| 4 | Pomp geeft automatisch water bij droge aarde, met maximale aan-tijd | les 16 |
| 5 | Servo opent het luchtluik bij hoge temperatuur (of licht) | les 11 |
| 6 | Webpagina met handbediening: ventilator, pomp, luik + noodstop | les 17 |
| 7 | Alle tijdgeving met `millis()`, geen lange `delay()`-blokken | les 10 |
| 8 | Sensorfouten worden gedetecteerd en getoond | les 15 |

### Eisen (profi)

- Je code staat in één heldere sketch met functies per taak (`regelVentilatie()`, `regelWatering()`, ...).
- Geen wachtwoorden of persoonlijke gegevens hardgecodeerd in de repo (Wi-Fi-gegevens via `secrets.h` die je **niet** pusht).
- Een `README.md` met: draad-schema in tekst of tabel, gekozen drempels + waarom, en een korte gebruiksaanwijzing.

## 💡 Aanpak

1. **Plan (10 min):** maak een tabel: sensor/actuator → GPIO-pin → functie. Laat de docent je plan aftekenen.
2. **Bouw (30 min):** sluit alles aan, controleer spanningsrichtingen, test elk onderdeel apart met een mini-sketch.
3. **Programmeer (40 min):** begin met de regelingsfuncties, daarna de webserver. Test na elke stap.
4. **Test (20 min):** doorloop het testscenario hieronder en noteer de resultaten.
5. **Documenteer & demonstreer (afwerking):** README af, demo aan de docent.

### Testscenario

1. Verwarm de DHT11 met je hand → ventilator EN luik moeten reageren.
2. Droge aarde → pomp start, stopt bij voldoende vocht, alarm bij time-out.
3. Open de webpagina → alle handbediening werkt, noodstop zet alles uit.
4. Trek de sensor los → systeem toont een fout en schakelt veilig (pompen dicht).

## ⭐ Bonusuitdagingen

- Log elke minuut de metingen naar de webpagina als tabel (laatste 10 metingen).
- Laat het luik ook reageren op licht (LDR/BH1750 uit les 8): te veel zon → luik dicht.
- Voeg een automatische modus-schakelaar toe op de webpagina (auto / handbediening).

## 🤖 AI-opdracht

Laat een AI-assistent je Arduino-code **reviewen** op veiligheid: wat gebeurt er bij een sensorfout, wat bij een vastlopende loop? Neem minimaal één suggestie over en beschrijf in je README wat je veranderd hebt en waarom. De code schrijf je zelf; de AI gebruik je alleen als reviewer.

## 📤 Inleveren

- `les18_eindproject.ino` (+ eventueel `secrets.h` **zonder** echte wachtwoorden) via Canvas;
- `README.md` met schema, drempels en gebruiksaanwijzing in je eigen repository;
- een **demo van 5 minuten** aan de docent, waarin je het testscenario doorloopt.

## 🧠 Beoordelingscriteria

| Criterium | Gewicht |
|---|---|
| Werkt het systeem volgens de 8 functionele eisen? | 40 % |
| Codekwaliteit: functies, namen, commentaar, geen wachtwoorden in de repo | 25 % |
| Correcte en veilige regeling (hysterese, time-out, sensorfouten) | 20 % |
| Documentatie (README) en demo | 15 % |

## 🧠 Reflectie

1. Welk onderdeel van het project vond je het moeilijkst, en waarom?
2. Als je nog een week had, wat zou je dan verbeteren?
3. Welke keuzes heb je gemaakt die een gebruiker van je kas zou opvallen?

## 📚 Samenvatting

Met deze eindopdracht laat je zien dat je sensoren, actuatoren, regeling, netwerk en documentatie kunt combineren tot één werkend smart-technology-project. Dat is precies de bagage die je meeneemt naar je vervolgopleiding of stage.