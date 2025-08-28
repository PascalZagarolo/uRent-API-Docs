# API: Booking erstellen

**Route:** `POST /api/v1/booking`  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `BOOKING_CREATE`

Erstellt eine **Buchung** (oder optional einen **Verfügbarkeitsblock**) für ein Inserat. Vor dem Anlegen kann automatisch geprüft werden, ob der Zeitraum frei ist.


---

## Inhalt

- [Zweck](#zweck)
- [Zugriff & Authentifizierung](#zugriff--authentifizierung)
- [Request](#request)
  - [HTTP-Methode](#http-methode)
  - [Header](#header)
  - [Body](#body)
  - [Validierung](#validierung)
  - [Verfügbarkeitsprüfung](#verfügbarkeitsprüfung)
- [Responses](#responses)
  - [201 – Erstellt](#201--erstellt)
  - [400 – Ungültiger Body / Datumsfehler](#400--ungültiger-body--datumsfehler)
  - [401 – Nicht autorisiert](#401--nicht-autorisiert)
  - [403 – Unzureichende Berechtigungen / Ownership](#403--unzureichende-berechtigungen--ownership)
  - [404 – Inserat nicht gefunden](#404--inserat-nicht-gefunden)
  - [409 – Zeitliche Kollision](#409--zeitliche-kollision)
  - [429 – Rate Limit überschritten](#429--rate-limit-überschritten)
  - [500 – Interner Fehler](#500--interner-fehler)
- [Feldbeschreibung (Request/Response)](#feldbeschreibung-requestresponse)
- [Beispiele](#beispiele)
  - [Minimal (mit Auto-Check)](#minimal-mit-auto-check)
  - [Mit Fahrzeugbindung & Zeit-Slots](#mit-fahrzeugbindung--zeit-slots)
  - [Nur Verfügbarkeitsblock anlegen](#nur-verfügbarkeitsblock-anlegen)
  - [Konfliktfall (409)](#konfliktfall-409)
- [JSON Schemas](#json-schemas)
  - [Request-Body](#request-body-schema)
  - [201-Response (vereinfacht)](#201-response-vereinfacht)
  - [409-Response (Konflikt)](#409-response-konflikt)
- [Changelog](#changelog)


---

## Zweck

- Anlage einer **Buchung** zu einem Inserat (optional für ein spezifisches Fahrzeug).
- Optionaler **Verfügbarkeitscheck** gegen bestehende Buchungen (standardmäßig **aktiv**).
- Unterstützt sowohl **ganztägige** als auch **Slot-basierte** Buchungen über `startPeriod`/`endPeriod` (beide als **String** mit ganzzahligem Inhalt).


---

## Zugriff & Authentifizierung

- Einer der folgenden Header ist erforderlich:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`BOOKING_CREATE`**
- **Ownership-Prüfung:** Bei externen Apps wird zusätzlich geprüft, ob der API Key für den **Owner des Inserats** buchen darf.
- Bei Rate-Limit-Verletzung kann ein `Retry-After`-Header gesetzt werden.


---

## Request

### HTTP-Methode
`POST`

### Header

| Header        | Erforderlich | Beispiel               | Beschreibung |
|---------------|-------------:|------------------------|--------------|
| Authorization | bedingt      | `Bearer sk_live_...`   | API Key (Alternative zu `x-api-key`). |
| x-api-key     | bedingt      | `sk_live_...`          | API Key (Alternative zu `Authorization`). |
| Content-Type  | ja           | `application/json`     | |
| Accept        | nein         | `application/json`     | |

> **Hinweis:** Einer der beiden Auth-Header muss vorhanden sein.

### Body

```json
{
  "inseratId": "c8a5f2d4-6a1e-4b2f-9a3e-0e1d2c3b4a5f",
  "vehicleId": "0c7e7d2c-9a0b-4db8-8e9c-2d1f0a3b5c7d",
  "name": "Buchung Max Mustermann",
  "buchungsnummer": "B-2025-00042",
  "content": "Abholung 08:00",
  "startDate": "2025-09-01T08:00:00Z",
  "endDate": "2025-09-03T08:00:00Z",
  "startPeriod": "08",
  "endPeriod": "12",
  "isAvailability": false,
  "checkAvailability": true,
  "requestId": "4e8c5c0a-7b29-4f8e-9ce0-0c6a0d7a9b2f"
}
```

### Validierung

- `inseratId`: **UUID** (Pflicht)
- `vehicleId`: **UUID** (optional)
- `name`: **string**, min. 1 (Pflicht)
- `startDate`, `endDate`: **ISO 8601 Date-Time** (Pflicht), **`startDate < endDate`**
- `startPeriod`, `endPeriod`: **string** mit ganzzahligem Inhalt (optional). Werden als **Halb-offene Intervalle** interpretiert: Überlappung, wenn **nicht** gilt `newStart >= existingEnd` **oder** `newEnd <= existingStart`.
- `isAvailability`: boolean (Default: `false`)
- `checkAvailability`: boolean (Default: `true`)
- `requestId`: UUID (optional)

### Verfügbarkeitsprüfung

Wenn `checkAvailability = true`:
- Es werden bestehende Buchungen des **Inserats** (und ggf. des **Fahrzeugs**) geladen.
- Prüfkriterien:
  - **Datumsüberlappung**: Konflikt, wenn Zeiträume sich schneiden (keine Lücke).
  - **Gleiches Datum** *und* beide Buchungen haben Slots (`startPeriod`/`endPeriod`): Konflikt, wenn Slot-Intervalle sich schneiden (siehe oben).
  - Fehlen Slots auf einer Seite bei gleichem Datum ⇒ Konflikt wird als **ganztägig** gewertet.
- Bei Konflikt: **409** mit Liste der **konfligierenden Buchungen**.


---

## Responses

### 201 – Erstellt

```json
{
  "success": true,
  "booking": {
    "...": "vollständiger Booking-Datensatz wie angelegt (inkl. startDate/endDate, isAvailability usw.)"
  }
}
```

> `booking.userId` wird – sofern vorhanden – anhand des authentifizierten Nutzers gesetzt, sonst `null`.  
> Wenn `requestId` übergeben wurde, wird der entsprechende **bookingRequest**-Eintrag nach erfolgreicher Erstellung **gelöscht**.

---

### 400 – Ungültiger Body / Datumsfehler

Ungültiges JSON:
```json
{ "success": false, "error": "Invalid JSON in request body" }
```

Schema- oder Feldfehler:
```json
{ "success": false, "error": "Invalid request body format" }
```

Datumslogik:
```json
{ "success": false, "error": "Start date must be before end date" }
```

---

### 401 – Nicht autorisiert

```json
{ "success": false, "error": "API key required" }
```

---

### 403 – Unzureichende Berechtigungen / Ownership

Fehlende Permission:
```json
{
  "success": false,
  "error": "Insufficient permissions",
  "missingPermissions": ["BOOKING_CREATE"]
}
```

Kein Zugriff auf Owner des Inserats (bei externen Apps):
```json
{ "success": false, "error": "No permission to create bookings for this inserat" }
```

---

### 404 – Inserat nicht gefunden

```json
{ "success": false, "error": "Inserat not found" }
```

---

### 409 – Zeitliche Kollision

```json
{
  "success": false,
  "error": "Inserat is not available for the requested time period",
  "availabilityConflict": {
    "conflictingBookings": [
      {
        "id": "bok_123",
        "startDate": "2025-09-01T08:00:00.000Z",
        "endDate": "2025-09-03T08:00:00.000Z",
        "startPeriod": "08",
        "endPeriod": "12",
        "name": "Buchung Max Mustermann"
      }
    ],
    "message": "The inserat is already booked for the requested time period. Found 1 conflicting booking(s)."
  }
}
```

---

### 429 – Rate Limit überschritten

Header: `Retry-After: <sekunden>`

```json
{ "success": false, "error": "Rate limit exceeded" }
```

---

### 500 – Interner Fehler

```json
{ "success": false, "error": "Internal server error" }
```


---

## Feldbeschreibung (Request/Response)

### Request-Felder

| Feld            | Typ                     | Pflicht | Beschreibung |
|-----------------|-------------------------|:------:|--------------|
| inseratId       | UUID                    | ✔      | Ziel-Inserat der Buchung. |
| vehicleId       | UUID                    | –      | Optional: Beschränkt die Buchung auf ein bestimmtes Fahrzeug. |
| name            | string                  | ✔      | Anzeigename/Titel der Buchung. |
| buchungsnummer  | string                  | –      | Optionale externe/öffentliche Nummer. |
| content         | string                  | –      | Notiz/Freitext. |
| startDate       | ISO 8601 Date-Time      | ✔      | Beginn (UTC). |
| endDate         | ISO 8601 Date-Time      | ✔      | Ende (UTC); muss **nach** `startDate` liegen. |
| startPeriod     | string (Integer)        | –      | Slot-Start (z. B. `"08"`); nur relevant, wenn **beide** Seiten Slots nutzen. |
| endPeriod       | string (Integer)        | –      | Slot-Ende (z. B. `"12"`); halb-offen \[`start`,`end`). |
| isAvailability  | boolean                 | –      | Wenn `true`, wird **kein** Kundenauftrag, sondern ein **Verfügbarkeitsblock** angelegt. |
| checkAvailability | boolean               | –      | Standard `true`. Bei `false` wird **keine** Kollisionserkennung durchgeführt. |
| requestId       | UUID                    | –      | Optional: zu löschende `bookingRequest`-ID nach erfolgreicher Erstellung. |

### Response-Felder (201)

| Feld     | Typ    | Beschreibung |
|----------|--------|--------------|
| success  | boolean | Immer `true` bei 201. |
| booking  | object | Neu angelegte Buchung (DB-Form). |


---

## Beispiele

### Minimal (mit Auto-Check)

```bash
curl -s -X POST "https://urent-rental.de/api/v1/booking" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "inseratId": "9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12",
    "name": "Buchung Max Mustermann",
    "startDate": "2025-09-01T08:00:00Z",
    "endDate": "2025-09-03T08:00:00Z"
  }'
```

---

### Mit Fahrzeugbindung & Zeit-Slots

```bash
curl -s -X POST "https://urent-rental.de/api/v1/booking" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "inseratId": "9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12",
    "vehicleId": "0c7e7d2c-9a0b-4db8-8e9c-2d1f0a3b5c7d",
    "name": "Buchung mit Slot",
    "startDate": "2025-09-01T00:00:00Z",
    "endDate": "2025-09-01T23:59:59Z",
    "startPeriod": "08",
    "endPeriod": "12"
  }'
```

---

### Nur Verfügbarkeitsblock anlegen

```bash
curl -s -X POST "https://urent-rental.de/api/v1/booking" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "inseratId": "9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12",
    "name": "Werkstatt",
    "startDate": "2025-09-10T00:00:00Z",
    "endDate": "2025-09-12T00:00:00Z",
    "isAvailability": true
  }'
```

---

### Konfliktfall (409)

```bash
curl -s -X POST "https://urent-rental.de/api/v1/booking" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "inseratId": "9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12",
    "name": "Konflikt-Test",
    "startDate": "2025-09-01T08:00:00Z",
    "endDate": "2025-09-03T08:00:00Z"
  }'
```

**Response 409**
```json
{
  "success": false,
  "error": "Inserat is not available for the requested time period",
  "availabilityConflict": {
    "conflictingBookings": [
      {
        "id": "bok_123",
        "startDate": "2025-09-01T08:00:00.000Z",
        "endDate": "2025-09-03T08:00:00.000Z",
        "startPeriod": "08",
        "endPeriod": "12",
        "name": "Buchung Max Mustermann"
      }
    ],
    "message": "The inserat is already booked for the requested time period. Found 1 conflicting booking(s)."
  }
}
```


---

## JSON Schemas

### Request-Body (Zod-Äquivalent)

```json
{
  "type": "object",
  "properties": {
    "inseratId": { "type": "string", "format": "uuid" },
    "vehicleId": { "type": "string", "format": "uuid" },
    "name": { "type": "string", "minLength": 1 },
    "buchungsnummer": { "type": "string" },
    "content": { "type": "string" },
    "startDate": { "type": "string", "format": "date-time" },
    "endDate": { "type": "string", "format": "date-time" },
    "startPeriod": { "type": "string" },
    "endPeriod": { "type": "string" },
    "isAvailability": { "type": "boolean", "default": false },
    "checkAvailability": { "type": "boolean", "default": true },
    "requestId": { "type": "string", "format": "uuid" }
  },
  "required": ["inseratId", "name", "startDate", "endDate"],
  "additionalProperties": false
}
```

### 201-Response (vereinfacht)

```json
{
  "type": "object",
  "properties": {
    "success": { "type": "boolean" },
    "booking": { "type": "object" }
  },
  "required": ["success"]
}
```

### 409-Response (Konflikt)

```json
{
  "type": "object",
  "properties": {
    "success": { "type": "boolean" },
    "error": { "type": "string" },
    "availabilityConflict": {
      "type": "object",
      "properties": {
        "conflictingBookings": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "id": { "type": "string" },
              "startDate": { "type": "string", "format": "date-time" },
              "endDate": { "type": "string", "format": "date-time" },
              "startPeriod": { "type": ["string", "null"] },
              "endPeriod": { "type": ["string", "null"] },
              "name": { "type": ["string", "null"] }
            },
            "required": ["id", "startDate", "endDate"]
          }
        },
        "message": { "type": "string" }
      },
      "required": ["conflictingBookings", "message"]
    }
  },
  "required": ["success", "error", "availabilityConflict"]
}
```


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation hinzugefügt.
