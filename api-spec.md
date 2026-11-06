# Cupify – API-specifikation

## 1. Introduktion

Detta dokument beskriver de endpoints, datamodeller och regler som ska implementeras i REST API:et för Cupify.

API:et ska hantera tre resurser:

- **Tournaments** – Turneringar
- **Teams** – Deltagande lag
- **Matches** – Matcher och matchresultat

Samtliga obligatoriska endpoints ska implementeras enligt specifikationen.

Hur koden organiseras, hur Lambda-funktionerna delas upp och hur DynamoDB-tabellerna konfigureras är upp till respektive grupp, så länge kraven i examinationen uppfylls.

## 2. Generella regler

### Base URL

Alla endpoints utgår från den URL som genereras vid driftsättning till AWS.

Exempel:

`https://example.execute-api.eu-north-1.amazonaws.com`

### Request och Response

- Request Body ska skickas som JSON.
- Response Body ska returneras som JSON, förutom vid `204 No Content`.
- Alla resurser ska identifieras med unika ID:n.
- ID:n ska genereras av backend när resurser skapas.
- ID:n ska inte kunna ändras vid uppdatering.
- Alla datum ska anges i ISO 8601-format, exempelvis `2026-11-20` eller `2026-11-20T14:00:00Z`.
- Saknade resurser ska returnera `404 Not Found`.
- Felaktig indata ska returnera `400 Bad Request`.

### HTTP-statuskoder

| Statuskod | Betydelse |
|---|---|
| `200 OK` | Anropet lyckades |
| `201 Created` | En ny resurs skapades |
| `204 No Content` | Resursen raderades |
| `400 Bad Request` | Felaktig eller saknad indata |
| `404 Not Found` | Resursen finns inte |
| `409 Conflict` | Anropet bryter mot en affärsregel |
| `500 Internal Server Error` | Ett oväntat serverfel inträffade |

### Felmeddelanden

Alla fel ska returneras i ett konsekvent JSON-format.

Exempel:

```json
{
  "error": "Team not found"
}
```

Felmeddelandet ska tydligt beskriva vad som gått fel. Den exakta formuleringen får variera.

### Listor

Endpoints som hämtar flera resurser ska returnera en JSON-array.

Om inga resurser finns ska en tom array returneras:

```json
[]
```

## 3. Datamodeller

Följande modeller beskriver informationen som API:et ska hantera.

Ytterligare attribut får användas om de behövs för implementationen.

### Tournament

| Fält | Datatyp | Beskrivning |
|---|---|---|
| `id` | string | Unikt ID, genereras av backend |
| `name` | string | Turneringens namn |
| `sport` | string | Idrott |
| `location` | string | Plats |
| `startDate` | string | Startdatum |
| `status` | string | `upcoming`, `ongoing` eller `completed` |

Exempel:

```json
{
  "id": "t-123",
  "name": "Värmland Cup",
  "sport": "Innebandy",
  "location": "Karlstad",
  "startDate": "2026-11-20",
  "status": "upcoming"
}
```

### Team

| Fält | Datatyp | Beskrivning |
|---|---|---|
| `id` | string | Unikt ID, genereras av backend |
| `tournamentId` | string | ID för turneringen |
| `name` | string | Lagets namn |
| `city` | string | Hemstad |
| `coach` | string | Tränarens namn |

Exempel:

```json
{
  "id": "team-123",
  "tournamentId": "t-123",
  "name": "Grums IBK",
  "city": "Grums",
  "coach": "Anna Svensson"
}
```

### Match

| Fält | Datatyp | Beskrivning |
|---|---|---|
| `id` | string | Unikt ID, genereras av backend |
| `tournamentId` | string | ID för turneringen |
| `homeTeamId` | string | Hemmalagets ID |
| `awayTeamId` | string | Bortalagets ID |
| `date` | string | Datum och tid |
| `status` | string | `scheduled` eller `completed` |
| `homeScore` | number/null | Hemmalagets mål |
| `awayScore` | number/null | Bortalagets mål |

Exempel:

```json
{
  "id": "match-123",
  "tournamentId": "t-123",
  "homeTeamId": "team-123",
  "awayTeamId": "team-456",
  "date": "2026-11-20T14:00:00Z",
  "status": "scheduled",
  "homeScore": null,
  "awayScore": null
}
```

En match som är färdigspelad ska ha status `completed` och ett registrerat resultat.

## 4. Endpoints – Turneringar

### GET /tournaments

**Beskrivning:** Hämtar samtliga turneringar.

**Request Body:** Ingen.

**Response – 200 OK**

```json
[
  {
    "id": "t-123",
    "name": "Värmland Cup",
    "sport": "Innebandy",
    "location": "Karlstad",
    "startDate": "2026-11-20",
    "status": "upcoming"
  }
]
```

**Felkoder:** `500`

### GET /tournaments/{id}

**Beskrivning:** Hämtar en specifik turnering.

**Path Parameters:**

- `id` – Turneringens ID.

**Request Body:** Ingen.

**Response – 200 OK**

```json
{
  "id": "t-123",
  "name": "Värmland Cup",
  "sport": "Innebandy",
  "location": "Karlstad",
  "startDate": "2026-11-20",
  "status": "upcoming"
}
```

**Felkoder:** `404`, `500`

### POST /tournaments

**Beskrivning:** Skapar en ny turnering.

**Request Body:**

```json
{
  "name": "Värmland Cup",
  "sport": "Innebandy",
  "location": "Karlstad",
  "startDate": "2026-11-20"
}
```

**Response – 201 Created**

```json
{
  "id": "t-123",
  "name": "Värmland Cup",
  "sport": "Innebandy",
  "location": "Karlstad",
  "startDate": "2026-11-20",
  "status": "upcoming"
}
```

**Regler:**

- Alla fält i Request Body är obligatoriska.
- `name`, `sport` och `location` får inte vara tomma.
- `startDate` ska vara ett giltigt datum.
- `status` sätts automatiskt till `upcoming`.

**Felkoder:** `400`, `500`

### PATCH /tournaments/{id}/status

**Beskrivning:** Uppdaterar status för en befintlig turnering.

**Path Parameters:**

- `id` – Turneringens ID.

**Request Body:**

```json
{
  "status": "ongoing"
}
```

**Response – 200 OK**

```json
{
  "id": "t-123",
  "name": "Värmland Cup",
  "sport": "Innebandy",
  "location": "Karlstad",
  "startDate": "2026-11-20",
  "status": "ongoing"
}
```

**Regler:**

- Turneringen måste existera.
- `status` är obligatoriskt.
- Tillåtna värden är `upcoming`, `ongoing` och `completed`.
- Endast turneringens status ska uppdateras.
- Övriga attribut ska förbli oförändrade.
- Turneringens namn, sport, plats och startdatum kan inte ändras efter att den skapats.

**Felkoder:** `400`, `404`, `500`

### DELETE /tournaments/{id}

**Beskrivning:** Raderar en turnering.

**Request Body:** Ingen.

**Response – 204 No Content**

Ingen Response Body.

**Regler:**

- Turneringen får endast raderas om den saknar både lag och matcher.

**Felkoder:** `404`, `409`, `500`

## 5. Endpoints – Lag

Alla lag tillhör en specifik turnering.

### GET /tournaments/{id}/teams

**Beskrivning:** Hämtar samtliga lag i en turnering.

**Request Body:** Ingen.

**Response – 200 OK**

```json
[
  {
    "id": "team-123",
    "tournamentId": "t-123",
    "name": "Grums IBK",
    "city": "Grums",
    "coach": "Anna Svensson"
  },
  {
    "id": "team-456",
    "tournamentId": "t-123",
    "name": "Karlstad IBF",
    "city": "Karlstad",
    "coach": "Erik Andersson"
  }
]
```

**Felkoder:** `404` om turneringen saknas, `500`

### GET /tournaments/{id}/teams/{teamId}

**Beskrivning:** Hämtar ett specifikt lag i en turnering.

**Request Body:** Ingen.

**Response – 200 OK**

```json
{
  "id": "team-123",
  "tournamentId": "t-123",
  "name": "Grums IBK",
  "city": "Grums",
  "coach": "Anna Svensson"
}
```

**Felkoder:** `404`, `500`

### POST /tournaments/{id}/teams

**Beskrivning:** Registrerar ett nytt lag i turneringen.

**Request Body:**

```json
{
  "name": "Grums IBK",
  "city": "Grums",
  "coach": "Anna Svensson"
}
```

**Response – 201 Created**

```json
{
  "id": "team-123",
  "tournamentId": "t-123",
  "name": "Grums IBK",
  "city": "Grums",
  "coach": "Anna Svensson"
}
```

**Regler:**

- Turneringen måste existera.
- Alla fält är obligatoriska och får inte vara tomma.
- `id` genereras av backend.
- `tournamentId` hämtas från URL:en.

**Felkoder:** `400`, `404`, `500`

### PUT /tournaments/{id}/teams/{teamId}

**Beskrivning:** Ersätter informationen för ett befintligt lag.

**Request Body:**

```json
{
  "name": "Grums IBK Ungdom",
  "city": "Grums",
  "coach": "Anna Svensson"
}
```

**Response – 200 OK**

Returnerar det uppdaterade Team-objektet.

**Regler:**

- Alla fält är obligatoriska.
- `id` och `tournamentId` får inte ändras.

**Felkoder:** `400`, `404`, `500`

### DELETE /tournaments/{id}/teams/{teamId}

**Beskrivning:** Raderar ett lag från en turnering.

**Request Body:** Ingen.

**Response – 204 No Content**

Ingen Response Body.

**Regler:**

- Laget får inte förekomma som hemma- eller bortalag i någon match inom turneringen.

**Felkoder:** `404`, `409`, `500`

## 6. Endpoints – Matcher

Alla matcher tillhör en specifik turnering.

### GET /tournaments/{id}/matches

**Beskrivning:** Hämtar samtliga matcher i en turnering.

**Request Body:** Ingen.

**Response – 200 OK**

```json
[
  {
    "id": "match-123",
    "tournamentId": "t-123",
    "homeTeamId": "team-123",
    "awayTeamId": "team-456",
    "date": "2026-11-20T14:00:00Z",
    "status": "scheduled",
    "homeScore": null,
    "awayScore": null
  }
]
```

**Felkoder:** `404` om turneringen saknas, `500`

### GET /tournaments/{id}/matches/{matchId}

**Beskrivning:** Hämtar information om en specifik match.

**Request Body:** Ingen.

**Response – 200 OK**

```json
{
  "id": "match-123",
  "tournamentId": "t-123",
  "homeTeamId": "team-123",
  "awayTeamId": "team-456",
  "date": "2026-11-20T14:00:00Z",
  "status": "scheduled",
  "homeScore": null,
  "awayScore": null
}
```

**Felkoder:** `404`, `500`

### POST /tournaments/{id}/matches

**Beskrivning:** Skapar en match mellan två lag.

**Request Body:**

```json
{
  "homeTeamId": "team-123",
  "awayTeamId": "team-456",
  "date": "2026-11-20T14:00:00Z"
}
```

**Response – 201 Created**

```json
{
  "id": "match-123",
  "tournamentId": "t-123",
  "homeTeamId": "team-123",
  "awayTeamId": "team-456",
  "date": "2026-11-20T14:00:00Z",
  "status": "scheduled",
  "homeScore": null,
  "awayScore": null
}
```

**Regler:**

- Turneringen måste existera.
- Båda lagen måste existera inom turneringen.
- Ett lag får inte möta sig självt.
- Alla fält i Request Body är obligatoriska.
- Matchens status sätts automatiskt till `scheduled`.
- Resultatet sätts till `null` för båda lagen.

**Felkoder:** `400`, `404`, `500`

### PUT /tournaments/{id}/matches/{matchId}

**Beskrivning:** Uppdaterar en schemalagd match.

**Request Body:**

```json
{
  "homeTeamId": "team-123",
  "awayTeamId": "team-789",
  "date": "2026-11-21T16:00:00Z"
}
```

**Response – 200 OK**

Returnerar det uppdaterade Match-objektet.

**Regler:**

- Endast matcher med status `scheduled` får uppdateras.
- Båda lagen måste existera inom turneringen.
- Ett lag får inte möta sig självt.
- Samtliga fält är obligatoriska.
- `id`, `tournamentId`, `status` och resultat får inte ändras genom denna endpoint.

**Felkoder:** `400`, `404`, `409`, `500`

### PATCH /tournaments/{id}/matches/{matchId}/result

**Beskrivning:** Registrerar resultatet för en match och markerar den som färdigspelad.

**Request Body:**

```json
{
  "homeScore": 5,
  "awayScore": 3
}
```

**Response – 200 OK**

```json
{
  "id": "match-123",
  "tournamentId": "t-123",
  "homeTeamId": "team-123",
  "awayTeamId": "team-456",
  "date": "2026-11-20T14:00:00Z",
  "status": "completed",
  "homeScore": 5,
  "awayScore": 3
}
```

**Regler:**

- Båda resultaten måste anges.
- Resultaten måste vara heltal som är 0 eller större.
- Matchen måste ha status `scheduled` innan resultatet registreras.
- Status ska automatiskt ändras till `completed`.
- Ett redan registrerat resultat får inte ändras.

**Felkoder:** `400`, `404`, `409`, `500`

### DELETE /tournaments/{id}/matches/{matchId}

**Beskrivning:** Raderar en match från turneringen.

**Request Body:** Ingen.

**Response – 204 No Content**

Ingen Response Body.

**Regler:**

- Både schemalagda och färdigspelade matcher får raderas.
- En raderad match ska inte längre påverka en eventuell tabellställning.

**Felkoder:** `404`, `500`

## 7. Bonusuppgift – Tabellställning

Denna del är frivillig och ingår inte i kraven för godkänt.

### GET /tournaments/{id}/standings

**Beskrivning:** Genererar en tabellställning för turneringen baserat på färdigspelade matcher.

**Request Body:** Ingen.

**Response – 200 OK**

```json
[
  {
    "teamId": "team-123",
    "teamName": "Grums IBK",
    "played": 3,
    "wins": 2,
    "draws": 1,
    "losses": 0,
    "goalsFor": 12,
    "goalsAgainst": 5,
    "goalDifference": 7,
    "points": 7
  },
  {
    "teamId": "team-456",
    "teamName": "Karlstad IBF",
    "played": 3,
    "wins": 2,
    "draws": 0,
    "losses": 1,
    "goalsFor": 10,
    "goalsAgainst": 7,
    "goalDifference": 3,
    "points": 6
  }
]
```

**Regler:**

- Endast matcher med status `completed` ska räknas.
- Vinst ger 3 poäng.
- Oavgjort ger 1 poäng.
- Förlust ger 0 poäng.
- Tabellställningen ska sorteras efter poäng och därefter målskillnad.
- Alla registrerade lag ska visas, även om de inte har spelat någon match.
- Beräkningen ska göras utifrån befintliga lag och matchresultat.
- Tabellställningen ska inte behöva lagras i DynamoDB.

**Felkoder:** `404`, `500`

## 8. Sammanställning av endpoints

### Turneringar

| Metod | Endpoint | Beskrivning |
|---|---|---|
| GET | `/tournaments` | Hämta alla turneringar |
| GET | `/tournaments/{id}` | Hämta turnering |
| POST | `/tournaments` | Skapa turnering |
| PATCH | `/tournaments/{id}/status` | Uppdatera turneringens status |
| DELETE | `/tournaments/{id}` | Radera turnering |

### Lag

| Metod | Endpoint | Beskrivning |
|---|---|---|
| GET | `/tournaments/{id}/teams` | Hämta alla lag |
| GET | `/tournaments/{id}/teams/{teamId}` | Hämta lag |
| POST | `/tournaments/{id}/teams` | Skapa lag |
| PUT | `/tournaments/{id}/teams/{teamId}` | Uppdatera lag |
| DELETE | `/tournaments/{id}/teams/{teamId}` | Radera lag |

### Matcher

| Metod | Endpoint | Beskrivning |
|---|---|---|
| GET | `/tournaments/{id}/matches` | Hämta alla matcher |
| GET | `/tournaments/{id}/matches/{matchId}` | Hämta match |
| POST | `/tournaments/{id}/matches` | Skapa match |
| PUT | `/tournaments/{id}/matches/{matchId}` | Uppdatera match |
| PATCH | `/tournaments/{id}/matches/{matchId}/result` | Rapportera resultat |
| DELETE | `/tournaments/{id}/matches/{matchId}` | Radera match |

### Bonus

| Metod | Endpoint | Beskrivning |
|---|---|---|
| GET | `/tournaments/{id}/standings` | Generera tabellställning |

**Totalt: 16 obligatoriska endpoints och 1 frivillig endpoint.**

## 9. Viktiga affärsregler

Följande regler gäller för hela API:et:

1. Ett lag måste tillhöra en existerande turnering.
2. En match måste tillhöra en existerande turnering.
3. Båda lagen i en match måste tillhöra samma turnering som matchen.
4. Ett lag får inte möta sig självt.
5. En turnering får inte raderas om den innehåller lag eller matcher.
6. Ett lag får inte raderas om det förekommer i en match.
7. En match får endast få sitt resultat registrerat en gång.
8. Matchresultat måste bestå av icke-negativa heltal.
9. En färdigspelad match får inte uppdateras genom PUT.
10. En turnerings status ska uppdateras genom PATCH, utan att övriga attribut påverkas.
11. Alla resurser ska identifieras med ID:n som genereras av backend.

## 10. Övrigt

Det är tillåtet att använda ytterligare interna funktioner, hjälpfiler och valideringslogik för att lösa uppgiften.

Det finns inga krav på en specifik mappstruktur eller ett visst antal Lambda-funktioner.

API:ets beteende och JSON-strukturer ska däremot följa denna specifikation så att samtliga endpoints kan testas på ett konsekvent sätt.
