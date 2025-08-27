# API: Inserat-Buchungen listen

**Route:** `GET /api/v1/inserat/{inseratId}/bookings`  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `BOOKING_READ`

Diese Route liefert **alle eingetragenen Buchungen** zu einem Inserat. Optional kann nach Zeitraum gefiltert und die verknüpften **User**- und **Vehicle**-Daten können einbezogen werden. Ergebnisse sind standardmäßig **aufsteigend nach `startDate`** sortiert.


---

## Inhalt

- [Zweck](#zweck)
- [Zugriff & Authentifizierung](#zugriff--authentifizierung)
- [Request](#request)
  - [HTTP-Methode](#http-methode)
  - [Pfadparameter](#pfadparameter)
  - [Header](#header)
  - [Query-Parameter](#query-parameter)
- [Responses](#responses)
  - [200 – OK](#200--ok)
  - [400 – Ungültige Inserat-ID](#400--ungültige-inserat-id)
  - [400 – Ungültige Query-Parameter](#400--ungültige-query-parameter)
  - [401 – Nicht autorisiert](#401--nicht-autorisiert)
  - [403 – Unzureichende Berechtigungen](#403--unzureichende-berechtigungen)
  - [404 – Inserat nicht gefunden](#404--inserat-nicht-gefunden)
  - [429 – Rate Limit überschritten](#429--rate-limit-überschritten)
  - [500 – Interner Fehler](#500--interner-fehler)
- [Feldbeschreibung (200-Response)](#feldbeschreibung-200-response)
- [Filterlogik & Sortierung](#filterlogik--sortierung)
- [Beispiele](#beispiele)
  - [Alle Buchungen eines Inserats](#alle-buchungen-eines-inserats)
  - [Zeitraumfilter (Start & Ende)](#zeitraumfilter-start--ende)
  - [Nur ab einem Startdatum](#nur-ab-einem-startdatum)
  - [Nur bis zu einem Enddatum](#nur-bis-zu-einem-enddatum)
  - [Mit verknüpften User- & Vehicle-Daten](#mit-verknüpften-user---vehicle-daten)
- [JSON Schema (200-Response)](#json-schema-200-response)
- [Changelog](#changelog)


---

## Zweck

Abrufen der **Buchungen** zu einem Inserat – inklusive optionaler **Zeitfilter** und **angereicherter Daten** (User, Vehicle). Zusätzlich werden **Gesamtzahlen** zurückgegeben:
- `total`: Anzahl **aller** Buchungen des Inserats
- `filtered`: Anzahl der **gefilterten** Buchungen der aktuellen Abfrage


---

## Zugriff & Authentifizierung

- Einer der folgenden Header ist erforderlich:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`BOOKING_READ`**
- Bei Rate-Limit-Verletzung kann ein `Retry-After`-Header gesetzt werden.


---

## Request

### HTTP-Methode
`GET`

### Pfadparameter

| Name      | Typ    | Erforderlich | Beschreibung                               |
|-----------|--------|--------------|--------------------------------------------|
| inseratId | UUIDv4 | ja           | Eindeutige Inserat-ID (muss gültige UUID sein). |

### Header

| Header        | Erforderlich | Beispiel               | Beschreibung                                   |
|---------------|-------------:|------------------------|------------------------------------------------|
| Authorization | bedingt      | `Bearer sk_live_...`   | API Key (Alternative zu `x-api-key`).          |
| x-api-key     | bedingt      | `sk_live_...`          | API Key (Alternative zu `Authorization`).      |
| Accept        | nein         | `application/json`     |                                                |

> **Hinweis:** Einer der beiden Auth-Header muss vorhanden sein.

### Query-Parameter

| Parameter      | Typ / Format         | Erforderlich | Beschreibung |
|----------------|----------------------|--------------|--------------|
| startDate      | ISO 8601 Date-Time    | nein         | Filtert Buchungen mit `booking.startDate >= startDate`. |
| endDate        | ISO 8601 Date-Time    | nein         | Filtert Buchungen mit `booking.endDate <= endDate`. |
| includeUser    | `"true"` \| `"false"` | nein         | Bei `"true"` werden User-Daten eingebettet. Standard: `false` (weglassen oder `"false"`). |
| includeVehicle | `"true"` \| `"false"` | nein         | Bei `"true"` werden Vehicle-Daten eingebettet. Standard: `false` (weglassen oder `"false"`). |

> **Validierung:** Wenn **beide** Datumsparameter gesetzt sind, muss `startDate < endDate` gelten.  
> **Tipp:** Wenn du `includeUser`/`includeVehicle` **nicht** benötigst, **lasse den Parameter weg** (nicht als leeren Wert senden).


---

## Responses

### 200 – OK

```json
{
  "bookings": [
    {
      "id": "bok_123",
      "name": "Max Mustermann",
      "buchungsnummer": "B-2025-00042",
      "content": "Abholung 08:00",
      "startDate": "2025-09-01T08:00:00.000Z",
      "endDate": "2025-09-03T08:00:00.000Z",
      "startPeriod": "MORNING",
      "endPeriod": "MORNING",
      "isAvailability": false,
      "createdAt": "2025-08-10T11:05:00.000Z",
      "userId": "usr_456",
      "vehicleId": "veh_789",
      "vehicle": {
        "id": "veh_789",
        "title": "Mercedes Sprinter L2H2",
        "registration": "B-AB 1234",
        "internalId": "FLOTTE-42"
      },
      "user": {
        "id": "usr_456",
        "name": "Max Mustermann",
        "email": "max@example.com"
      }
    }
  ],
  "total": 12,
  "filtered": 4,
  "timePeriod": {
    "startDate": "2025-09-01T00:00:00.000Z",
    "endDate": "2025-09-30T23:59:59.000Z"
  }
}
```

> `vehicle` und/oder `user` erscheinen **nur**, wenn die entsprechenden `include*`-Parameter auf `"true"` stehen.  
> `isAvailability: true` kennzeichnet **Verfügbarkeitsblöcke** (keine echte Kundenbuchung).


---

### 400 – Ungültige Inserat-ID

```json
{ "error": "Invalid inserat ID" }
```

---

### 400 – Ungültige Query-Parameter

```json
{
  "error": "Invalid query parameters",
  "details": [
    {
      "code": "invalid_string",
      "message": "Invalid date time",
      "path": ["startDate"]
    }
  ]
}
```

oder

```json
{ "error": "Start date must be before end date" }
```

---

### 401 – Nicht autorisiert

Fehlender API Key:
```json
{ "error": "API key required" }
```

Ungültige/abgewiesene Authentifizierung:
```json
{ "error": "Invalid API key" }
```
*(Fehlermeldung kann variieren.)*

---

### 403 – Unzureichende Berechtigungen

```json
{
  "error": "Insufficient permissions",
  "missingPermissions": ["BOOKING_READ"]
}
```

---

### 404 – Inserat nicht gefunden

```json
{ "error": "Inserat not found" }
```

---

### 429 – Rate Limit überschritten

Header: `Retry-After: <sekunden>`

```json
{ "error": "Rate limit exceeded" }
```

---

### 500 – Interner Fehler

```json
{ "error": "Internal server error" }
```


---

## Feldbeschreibung (200-Response)

### Root

| Feld         | Typ    | Beschreibung |
|--------------|--------|--------------|
| bookings     | array  | Liste der Buchungen im angeforderten Kontext. |
| total        | number | Anzahl **aller** Buchungen zum Inserat (ohne Filter). |
| filtered     | number | Anzahl der Buchungen **nach Anwendung** der Filter. |
| timePeriod   | object \| fehlt | Nur vorhanden, wenn `startDate` oder `endDate` übergeben wurde. |

### `bookings[]`

| Feld            | Typ                 | Beschreibung |
|-----------------|---------------------|--------------|
| id              | string              | Buchungs-ID. |
| name            | string \| null      | Optionaler Name/Betreff der Buchung. |
| buchungsnummer  | string \| null      | Interne/öffentliche Buchungsnummer. |
| content         | string \| null      | Freitext/Notiz. |
| startDate       | string \| null      | ISO‑Zeitpunkt (UTC) für Start. |
| endDate         | string \| null      | ISO‑Zeitpunkt (UTC) für Ende. |
| startPeriod     | string \| null      | Optionaler Start-Zeitbereich (z. B. `MORNING`). |
| endPeriod       | string \| null      | Optionaler End-Zeitbereich. |
| isAvailability  | boolean             | `true` für Verfügbarkeitsblock statt Kundenbuchung. |
| createdAt       | string              | ISO‑Zeitpunkt (UTC) der Erstellung. |
| userId          | string \| null      | Referenz auf Nutzer. |
| vehicleId       | string \| null      | Referenz auf Fahrzeug. |
| vehicle         | object \| null      | Nur bei `includeVehicle="true"`. Felder: `id`, `title`, `registration`, `internalId`. |
| user            | object \| null      | Nur bei `includeUser="true"`. Felder: `id`, `name`, `email`. |

### `timePeriod`

| Feld      | Typ    | Beschreibung |
|-----------|--------|--------------|
| startDate | string | Übergebener Wert (falls gesetzt). |
| endDate   | string | Übergebener Wert (falls gesetzt). |


---

## Filterlogik & Sortierung

- **Zeitraumfilter:**
  - Wenn `startDate` gesetzt: `booking.startDate >= startDate`
  - Wenn `endDate` gesetzt: `booking.endDate <= endDate`
  - Wenn **beide** gesetzt: zusätzlich Validierung `startDate < endDate` (ansonsten `400`).
- **Sortierung:** `ORDER BY startDate ASC`
- **Counts:**
  - `total`: alle Buchungen des Inserats (ohne Filter)
  - `filtered`: Buchungen nach Anwendung der übergebenen Filter
- **Pagination:** Aktuell **nicht** vorhanden (alle passenden Einträge werden geliefert).


---

## Beispiele

### Alle Buchungen eines Inserats

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/bookings" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Accept: application/json"
```

---

### Zeitraumfilter (Start & Ende)

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/bookings?startDate=2025-09-01T00:00:00Z&endDate=2025-09-30T23:59:59Z" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Accept: application/json"
```

---

### Nur ab einem Startdatum

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/bookings?startDate=2025-09-01T00:00:00Z" \
  -H "Authorization: Bearer <API_KEY>"
```

---

### Nur bis zu einem Enddatum

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/bookings?endDate=2025-09-30T23:59:59Z" \
  -H "Authorization: Bearer <API_KEY>"
```

---

### Mit verknüpften User- & Vehicle-Daten

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/bookings?includeUser=true&includeVehicle=true" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Accept: application/json"
```


---

## JSON Schema (200-Response)

> Vereinfachtes Schema für erfolgreiche **200**-Antworten. Fehlerantworten der Statuscodes 4xx/5xx können abweichen.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "BookingsResponse",
  "type": "object",
  "properties": {
    "bookings": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {"type": "string"},
          "name": {"type": ["string", "null"]},
          "buchungsnummer": {"type": ["string", "null"]},
          "content": {"type": ["string", "null"]},
          "startDate": {"type": ["string", "null"], "format": "date-time"},
          "endDate": {"type": ["string", "null"], "format": "date-time"},
          "startPeriod": {"type": ["string", "null"]},
          "endPeriod": {"type": ["string", "null"]},
          "isAvailability": {"type": "boolean"},
          "createdAt": {"type": "string", "format": "date-time"},
          "userId": {"type": ["string", "null"]},
          "vehicleId": {"type": ["string", "null"]},
          "vehicle": {
            "type": ["object", "null"],
            "properties": {
              "id": {"type": "string"},
              "title": {"type": ["string", "null"]},
              "registration": {"type": ["string", "null"]},
              "internalId": {"type": ["string", "null"]}
            },
            "additionalProperties": false
          },
          "user": {
            "type": ["object", "null"],
            "properties": {
              "id": {"type": "string"},
              "name": {"type": ["string", "null"]},
              "email": {"type": "string"}
            },
            "additionalProperties": false
          }
        },
        "required": ["id", "isAvailability", "createdAt"]
      }
    },
    "total": {"type": "number"},
    "filtered": {"type": "number"},
    "timePeriod": {
      "type": "object",
      "properties": {
        "startDate": {"type": "string"},
        "endDate": {"type": "string"}
      },
      "required": [],
      "additionalProperties": false
    }
  },
  "required": ["bookings", "total", "filtered"],
  "additionalProperties": true
}
```


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation hinzugefügt.
