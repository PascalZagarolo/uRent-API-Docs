# API: Booking bearbeiten (Update)

**Route:** `PATCH /api/v1/booking/{bookingId}` *(je nach Routing ggf. `/api/v1/booking/{bookingId}/edit`)*  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `BOOKING_UPDATE`

Aktualisiert Felder einer bestehenden **Buchung** (teilweise Updates). Optional kann geprüft werden, ob der neue Zeitraum/Fahrzeugbezug **kollisionsfrei** ist.


---

## Inhalt

- [Zweck](#zweck)
- [Zugriff & Authentifizierung](#zugriff--authentifizierung)
- [Request](#request)
  - [HTTP-Methode](#http-methode)
  - [Pfadparameter](#pfadparameter)
  - [Header](#header)
  - [Body](#body)
  - [Validierung & Regeln](#validierung--regeln)
  - [Verfügbarkeitsprüfung](#verfügbarkeitsprüfung)
- [Responses](#responses)
  - [200 – OK](#200--ok)
  - [400 – Ungültiges ID-Format / Body / Datumsfehler](#400--ungültiges-id-format--body--datumsfehler)
  - [401 – Nicht autorisiert](#401--nicht-autorisiert)
  - [403 – Unzureichende Berechtigungen / Ownership](#403--unzureichende-berechtigungen--ownership)
  - [404 – Booking oder Inserat nicht gefunden](#404--booking-oder-inserat-nicht-gefunden)
  - [409 – Zeitliche Kollision](#409--zeitliche-kollision)
  - [429 – Rate Limit überschritten](#429--rate-limit-überschritten)
  - [500 – Interner Fehler](#500--interner-fehler)
- [Feldbeschreibung](#feldbeschreibung)
- [Beispiele](#beispiele)
  - [Zeitraum ändern (mit Availability-Check)](#zeitraum-ändern-mit-availability-check)
  - [Nur Notiz/Titel ändern](#nur-notiztitel-ändern)
  - [Fahrzeug zuweisen & Slots setzen](#fahrzeug-zuweisen--slots-setzen)
  - [Konfliktfall (409)](#konfliktfall-409)
- [JSON Schemas](#json-schemas)
  - [Request-Body](#request-body-schema)
  - [200-Response (vereinfacht)](#200-response-vereinfacht)
  - [409-Response (Konflikt)](#409-response-konflikt)
- [Hinweise](#hinweise)
- [Changelog](#changelog)


---

## Zweck

- **Teil-Update** einer Buchung: Nur übergebene Felder werden geändert.
- **Optionale Kollisionserkennung** gegen bestehende Buchungen (Standard: `checkAvailability=true`).


---

## Zugriff & Authentifizierung

- Einer der folgenden Header ist erforderlich:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`BOOKING_UPDATE`**
- **Ownership-Prüfung (externe Apps):** Es wird erzwungen, dass der Key zur Installation gehört, die Zugriff auf den **Owner des Inserats** der Buchung hat.
- Bei Rate-Limit-Verletzung kann ein `Retry-After`-Header gesetzt werden.


---

## Request

### HTTP-Methode
`PATCH`

### Pfadparameter

| Name       | Typ    | Erforderlich | Beschreibung |
|------------|--------|--------------|--------------|
| bookingId  | UUIDv4 | ja           | Zu aktualisierende Buchung. |

### Header

| Header        | Erforderlich | Beispiel               | Beschreibung |
|---------------|-------------:|------------------------|--------------|
| Authorization | bedingt      | `Bearer sk_live_...`   | API Key (Alternative zu `x-api-key`). |
| x-api-key     | bedingt      | `sk_live_...`          | API Key (Alternative zu `Authorization`). |
| Content-Type  | ja           | `application/json`     | |
| Accept        | nein         | `application/json`     | |

### Body

```json
{
  "vehicleId": "0c7e7d2c-9a0b-4db8-8e9c-2d1f0a3b5c7d",
  "name": "Neue Bezeichnung",
  "buchungsnummer": "B-2025-00042",
  "content": "Hinweis aktualisiert",
  "startDate": "2025-09-02T08:00:00Z",
  "endDate": "2025-09-04T08:00:00Z",
  "startPeriod": "08",
  "endPeriod": "12",
  "isAvailability": false,
  "checkAvailability": true
}
```

### Validierung & Regeln

- Alle Felder **optional** (Teil-Update).  
- `name`: string, min. 1 (falls gesetzt)  
- `startDate`, `endDate`: ISO‑Zeit (falls gesetzt)  
- Wenn **beide** gesetzt: `startDate < endDate`  
- `startPeriod`, `endPeriod`: **string** mit ganzzahligem Inhalt (Slots; halb-offen \[`start`,`end`)).  
- `checkAvailability`: Default `true`.

### Verfügbarkeitsprüfung

- Die Serverlogik prüft Kollisionen **nur**, wenn mindestens eines von
  `startDate`, `endDate` oder `vehicleId` im Request steht **und** `checkAvailability=true` ist.
- Für die Prüfung werden **fehlende** Werte mit den **aktuellen** Booking-Werten ergänzt (effektiver Zeitraum/Fahrzeug).
- Bei **gleichem Datum** und vorhandenen Slots auf beiden Seiten wird **Slot-Overlap** geprüft; ansonsten gilt **ganztägiger** Konflikt.
- Die eigene Buchung wird von der Kollisionsermittlung **ausgeschlossen**.

> **Hinweis:** Änderst du **nur** `startPeriod`/`endPeriod`, löst das **allein** keine Prüfung aus. Sende zusätzlich `startDate` (und/oder `endDate`), um eine Slot-Revalidierung zu erzwingen.


---

## Responses

### 200 – OK

```json
{
  "success": true,
  "booking": {
    "...": "aktualisierter Booking-Datensatz"
  }
}
```

---

### 400 – Ungültiges ID-Format / Body / Datumsfehler

```json
{ "success": false, "error": "Invalid booking ID format" }
```
```json
{ "success": false, "error": "Invalid JSON in request body" }
```
```json
{ "success": false, "error": "Invalid request body format" }
```
```json
{ "success": false, "error": "Start date must be before end date" }
```
```json
{ "success": false, "error": "Start date and end date are required for availability check" }
```

---

### 401 – Nicht autorisiert

```json
{ "success": false, "error": "API key required" }
```

---

### 403 – Unzureichende Berechtigungen / Ownership

```json
{
  "success": false,
  "error": "Insufficient permissions",
  "missingPermissions": ["BOOKING_UPDATE"]
}
```
```json
{ "success": false, "error": "No permission to update bookings for this inserat" }
```

---

### 404 – Booking oder Inserat nicht gefunden

```json
{ "success": false, "error": "Booking not found" }
```
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
        "name": "Kollidierende Buchung"
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

## Feldbeschreibung

### Request-Felder

| Feld            | Typ              | Beschreibung |
|-----------------|------------------|--------------|
| vehicleId       | UUID             | Fahrzeugbezug der Buchung. |
| name            | string           | Titel/Bezeichnung. |
| buchungsnummer  | string           | Optionale externe/öffentliche Nummer. |
| content         | string           | Freitext/Notiz. |
| startDate       | ISO 8601         | Startzeit (UTC). |
| endDate         | ISO 8601         | Endzeit (UTC). |
| startPeriod     | string (Integer) | Slot-Start (halb-offen). |
| endPeriod       | string (Integer) | Slot-Ende (halb-offen). |
| isAvailability  | boolean          | Verfügbarkeitsblock statt Kundenbuchung. |
| checkAvailability | boolean        | Default `true`; triggert Kollisionsprüfung bei Zeit-/Fahrzeugänderung. |

### Response-Felder (200)

| Feld     | Typ     | Beschreibung |
|----------|---------|--------------|
| success  | boolean | `true` bei erfolgreichem Update. |
| booking  | object  | Aktualisierter Datensatz. |


---

## Beispiele

### Zeitraum ändern (mit Availability-Check)

```bash
curl -s -X PATCH "https://urent-rental.de/api/v1/booking/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "startDate": "2025-09-02T08:00:00Z",
    "endDate": "2025-09-04T08:00:00Z",
    "checkAvailability": true
  }'
```

---

### Nur Notiz/Titel ändern

```bash
curl -s -X PATCH "https://urent-rental.de/api/v1/booking/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{ "content": "Neue Info", "name": "Neue Bezeichnung" }'
```

---

### Fahrzeug zuweisen & Slots setzen

```bash
curl -s -X PATCH "https://urent-rental.de/api/v1/booking/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "vehicleId": "0c7e7d2c-9a0b-4db8-8e9c-2d1f0a3b5c7d",
    "startDate": "2025-09-01T00:00:00Z",
    "endDate": "2025-09-01T23:59:59Z",
    "startPeriod": "08",
    "endPeriod": "12",
    "checkAvailability": true
  }'
```

---

### Konfliktfall (409)

```bash
curl -s -X PATCH "https://urent-rental.de/api/v1/booking/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "startDate": "2025-09-01T08:00:00Z",
    "endDate": "2025-09-03T08:00:00Z",
    "checkAvailability": true
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
        "endDate": "2025-09-03T08:00:00.000Z"
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
    "vehicleId": { "type": "string", "format": "uuid" },
    "name": { "type": "string", "minLength": 1 },
    "buchungsnummer": { "type": "string" },
    "content": { "type": "string" },
    "startDate": { "type": "string", "format": "date-time" },
    "endDate": { "type": "string", "format": "date-time" },
    "startPeriod": { "type": "string" },
    "endPeriod": { "type": "string" },
    "isAvailability": { "type": "boolean" },
    "checkAvailability": { "type": "boolean", "default": true }
  },
  "required": [],
  "additionalProperties": false
}
```

### 200-Response (vereinfacht)

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

## Hinweise

- **Teil-Update:** Nur übergebene Felder werden geändert.
- **Slot-Revalidierung:** Wenn du ausschließlich Slots änderst, setze zusätzlich `startDate` (und/oder `endDate`), um die Kollisionsprüfung zu triggern.
- **Idempotenz (Empfehlung):** Für PATCH-Requests mit Nebenwirkungen kannst du `Idempotency-Key` verwenden, um Retries zu entdoppeln.
- **Audit:** Aktion und Ergebnis protokollieren (Wer/Was/Wann/Erfolg).


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation hinzugefügt.
