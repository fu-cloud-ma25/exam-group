# Gruppexamination – Cupify

* [Grupper](https://drive.google.com/file/d/1XPNy1gmh7XdaDQixLucy4-My6_u0xDVX/view?usp=sharing)
* [Inspelad genomgång](https://funet.sharepoint.com/:v:/s/MobilapplikationsutvecklareYhdistans-Klass25/IQAnyAxf5Py3RZOeznaUjSxYASgWFhSwNWUYsZrC8bxYQcY?e=SUVziO)

## Introduktion

I denna gruppexamination ska ni utveckla **Cupify**, ett REST API för att administrera idrottsturneringar. Systemet ska kunna hantera turneringar, deltagande lag, matcher och matchresultat.

Projektet genomförs i grupper om 4–5 personer och syftar till att ni ska få tillämpa era kunskaper inom backendutveckling, REST API, AWS Lambda, API Gateway och DynamoDB.

## Uppgiften

Ni har fått i uppdrag att utveckla backend-systemet för Cupify, en tjänst som hjälper idrottsföreningar och arrangörer att administrera sina turneringar.

Systemet ska kunna hantera flera turneringar samtidigt, exempelvis inom fotboll, innebandy och ishockey.

Varje turnering ska kunna innehålla ett antal deltagande lag och matcher. Matcherna spelas mellan två lag inom samma turnering, och när en match är färdigspelad ska resultatet kunna registreras.

### Funktionalitet

**Turneringar**

- Skapa nya turneringar.
- Hämta information om en eller flera turneringar.
- Uppdatera status för befintliga turneringar.
- Ta bort turneringar som inte innehåller några lag eller matcher.

**Lag**

- Registrera lag i en turnering.
- Hämta information om ett enskilt lag eller samtliga lag i en turnering.
- Uppdatera laginformation.
- Ta bort lag som inte förekommer i någon match.

**Matcher**

- Skapa matcher mellan två lag i samma turnering.
- Hämta information om enskilda matcher eller samtliga matcher i en turnering.
- Uppdatera matchinformation.
- Ta bort matcher.
- Registrera matchresultat och markera matcher som färdigspelade.

Samtliga endpoints, datamodeller, HTTP-metoder, förväntade JSON-strukturer och affärsregler beskrivs i den separata [API-specifikationen](./api-spec.md), och måste följas.

### Bonusuppgift – Tabellställning

För grupper som blir klara med de obligatoriska kraven finns möjlighet att vidareutveckla Cupify med en funktion för tabellställningar.

Funktionen ska kunna:

- Generera en tabellställning för en specifik turnering.
- Beräkna spelade matcher, vinster, oavgjorda matcher och förluster.
- Beräkna gjorda mål, insläppta mål, målskillnad och poäng.
- Sortera lagen efter poäng och därefter målskillnad.

Poängfördelningen ska vara 3 poäng för vinst, 1 poäng för oavgjort och 0 poäng för förlust.

Tabellställningen ska beräknas dynamiskt utifrån färdigspelade matcher och behöver inte lagras i databasen.

Detaljerade krav för bonusuppgiften finns i [API-specifikationen](./api-spec.md#7-bonusuppgift--tabellställning).

**Bonusuppgiften är helt frivillig och påverkar inte bedömningen för godkänt.**

## Grupparbete

Examinationen genomförs i grupper om **4–5 studerande**.

Ni ansvarar gemensamt för att planera, utveckla, testa och färdigställa projektet inom den angivna tidsramen.

Arbetet ska organiseras så att samtliga gruppmedlemmar deltar aktivt i utvecklingen och får möjlighet att bidra med kod.

Ni ska använda **Git och GitHub** för versionshantering och samarbete, och kan med fördel visualisera era arbetsuppgifter med hjälp av **GitHub Projects**.

Det innebär att:

- Projektet ska finnas i ett gemensamt GitHub-repository.
- Samtliga gruppmedlemmar ska bidra till projektets utveckling.
- Arbetet ska fördelas och planeras gemensamt.
- Ni ska arbeta med separata branches och använda pull requests för att sammanföra kod.
- Ni ansvarar gemensamt för att den färdiga applikationen fungerar som en helhet.

Det är tillåtet och uppmuntras att hjälpa varandra, diskutera lösningar och arbeta tillsammans med svårare delar av projektet.

## Tekniska Krav

Projektet ska utvecklas med följande tekniker:

- **Node.js och JavaScript** för backendutveckling.
- **AWS Lambda** för att exekvera backend-funktionerna.
- **Amazon API Gateway** för att exponera REST API.
- **Amazon DynamoDB** för datalagring.
- **Serverless Framework** för konfigurering och driftsättning.
- **Git och GitHub** för versionshantering.

### Databas

Databasen ska utformas enligt **Multi Table Design** med tre separata tabeller:

- `Tournaments` – information om turneringar.
- `Teams` – information om deltagande lag.
- `Matches` – information om matcher och resultat.

Relationer mellan entiteter ska hanteras genom identifierare (ID).

Databasen ska utformas så att information inte behöver lagras på flera ställen i onödan. Exempelvis ska en match referera till deltagande lag genom deras ID i stället för att lagra lagens namn.

Det finns inget krav på att använda Global Secondary Indexes (GSI).

### REST API

API ska följa REST-principer och använda de HTTP-metoder som anges i [API-specifikationen](./api-spec.md).

API ska:

- Ta emot och returnera data i JSON-format.
- Använda lämpliga HTTP-statuskoder.
- Hantera felaktiga anrop och saknade resurser.
- Validera inkommande data.
- Ha en tydlig och konsekvent struktur.

Samtliga obligatoriska endpoints ska implementeras enligt [API-specifikationen](./api-spec.md#8-sammanställning-av-endpoints).

### Avgränsningar

Följande ingår **inte** i examinationen:

- Frontend eller grafiskt användargränssnitt.
- Inloggning, registrering eller autentisering.
- Automatisk generering av spelscheman.
- Hantering av spelare eller individuell spelarstatistik.
- Slutspel eller avancerade turneringsformat.

## Krav för Godkänt

För att examinationen ska bedömas som godkänd ska följande krav vara uppfyllda.

### Funktionella krav

- [ ] Det ska gå att skapa, hämta och ta bort turneringar samt uppdatera deras status.
- [ ] Det ska gå att registrera, hämta, uppdatera och ta bort lag inom en turnering.
- [ ] Det ska gå att skapa, hämta, uppdatera och ta bort matcher inom en turnering.
- [ ] En match ska innehålla två olika lag som båda tillhör samma turnering.
- [ ] Det ska gå att registrera ett matchresultat och markera matchen som färdigspelad.
- [ ] Matchresultat får inte innehålla negativa mål.
- [ ] En turnering får inte kunna raderas om den innehåller lag eller matcher.
- [ ] Ett lag får inte kunna raderas om det förekommer i en match.
- [ ] Samtliga obligatoriska funktioner och affärsregler i [API-specifikationen](./api-spec.md) ska vara implementerade.

### Tekniska krav

- [ ] Backend ska vara utvecklad med Node.js och JavaScript.
- [ ] API ska använda AWS Lambda och API Gateway.
- [ ] Projektet ska vara konfigurerat och driftsatt med Serverless Framework.
- [ ] Data ska lagras i DynamoDB med tre separata tabeller.
- [ ] API ska implementera de obligatoriska endpoints som anges i [API-specifikationen](./api-spec.md).
- [ ] API ska använda korrekta HTTP-metoder och lämpliga statuskoder.
- [ ] API ska hantera felaktig indata och saknade resurser på ett kontrollerat sätt.
- [ ] Projektet ska vara versionshanterat med Git och GitHub.
- [ ] Samtliga gruppmedlemmar ska ha bidragit aktivt till utvecklingsarbetet.
- [ ] Det färdiga API ska vara driftsatt och möjligt att testa via en publik API-adress.

## Inlämning

Examinationen lämnas in genom att en gruppmedlem skickar in följande:

1. **Länk till GitHub-repository** som innehåller projektets källkod.
2. **Länk till det driftsatta API** i AWS. Denna läggs i er Readme-fil.

GitHub-repositoryt ska innehålla en README-fil med instruktioner för hur projektet installeras och driftsätts.

**Deadline:** 13/11 kl 23:59

### Bedömning

Examinationen bedöms med betygen **Godkänt (G)** eller **Icke godkänt (IG)**.

För godkänt krävs att samtliga obligatoriska krav är uppfyllda och att projektet fungerar som en sammanhängande applikation.

Bedömningen utgår från både den färdiga lösningen och gruppmedlemmarnas deltagande i utvecklingsarbetet.
