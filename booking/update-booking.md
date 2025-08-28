# API: Booking bearbeiten (Update) – Minutenbasierte Zeit-Slots

**Route:** `PATCH /api/v1/booking/{bookingId}`  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `BOOKING_UPDATE`

Aktualisiert eine bestehende **Buchung** per Teil-Update. Unterstützt **minutenbasierte Zeit-Slots** über `startPeriod`/`endPeriod` (Strings mit Minuten seit 00:00, z. B. `"600"` = 10:00). Optional prüft der Server, ob die Änderungen **kollisionsfrei** sind.


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
  - [Fahrzeug zuweisen & Minuten-Slots setzen](#fahrzeug-zuweisen--minuten-slots-setzen)
  - [Konfliktfall (409)](#konfliktfall-409)
- [JSON Schemas](#json-schemas)
  - [Request-Body](#request-body-schema)
  - [200-Response (vereinfacht)](#200-response-vereinfacht)
  - [409-Response (Konflikt)](#409-response-konflikt)
- [Hinweise](#hinweise)
- [Changelog](#changelog)


---

## Zweck

- **Teil-Update**: Es werden nur übergebene Felder geändert.
- **Minutenbasierte Slots**: `startPeriod`/`endPeriod` sind **Strings** mit Minuten seit Mitternacht (z. B. `"600"` = 10:00, `"720"` = 12:00).
- **Optionale Kollisionserkennung** (Standard: `checkAvailability=true`).


---

## Zugriff & Authentifizierung

- Einer der folgenden Header ist erforderlich:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`BOOKING_UPDATE`**
- **Ownership-Prüfung (externe Apps):** Der Key muss zur Installation gehören, die Zugriff auf den **Owner des Inserats** der Buchung hat.
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
  "startPeriod": "600",
  "endPeriod": "720",
  "isAvailability": false,
  "checkAvailability": true
}
```

### Validierung & Regeln

- Alle Felder **optional** (Teil-Update).
- `name`: string, min. 1 (falls gesetzt).
- `startDate`, `endDate`: ISO‑Zeit (falls gesetzt).
- Wenn **beide** gesetzt: `startDate < endDate`.
- **Minuten-Slots** (`startPeriod`, `endPeriod`): String mit ganzzahligem Inhalt, empfohlen **0–1440**; Intervall ist **halb‑offen** `[start, end)` in Minuten.
- **Gleiches Datum** & **beide Seiten** haben Slots ⇒ Overlap, wenn **nicht** gilt `newStart >= existingEnd` **oder** `newEnd <= existingStart`.
- **Teilweise Slots** (nur eine Seite hat Slots) ⇒ wird als **potenzieller Konflikt** behandelt.
- `checkAvailability`: Default `true`.

### Verfügbarkeitsprüfung

- Die Kollisionsprüfung wird **nur** ausgeführt, wenn eines von `startDate`, `endDate` oder `vehicleId` im Request enthalten ist **und** `checkAvailability=true` ist.
- Fehlen im Request Zeitwerte, werden die **aktuellen** Werte der Buchung verwendet (effektiver Zeitraum/Fahrzeug).
- Eigene Buchung wird bei der Prüfung **ausgeschlossen**.

> **Wichtig:** Änderst du **nur** `startPeriod`/`endPeriod`, triggert das **allein** keine Prüfung. Sende zusätzlich `startDate` (und/oder `endDate`), um die Slot-Revalidierung auszulösen.


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
        "startPeriod": "660",
        "endPeriod": "780",
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
| startPeriod     | string (Minuten) | Slot-Start, Minuten seit 00:00 (z. B. `"600"` = 10:00). |
| endPeriod       | string (Minuten) | Slot-Ende, Minuten seit 00:00 (halb-offen). |
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

### Fahrzeug zuweisen & Minuten-Slots setzen

```bash
curl -s -X PATCH "https://urent-rental.de/api/v1/booking/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "vehicleId": "0c7e7d2c-9a0b-4db8-8e9c-2d1f0a3b5c7d",
    "startDate": "2025-09-01T00:00:00Z",
    "endDate": "2025-09-01T23:59:59Z",
    "startPeriod": "600",
    "endPeriod": "720",
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
    "startPeriod": "660",
    "endPeriod": "780",
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
        "endDate": "2025-09-03T08:00:00.000Z",
        "startPeriod": "600",
        "endPeriod": "720"
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
    "startPeriod": { "type": "string", "description": "Minuten seit 00:00, z. B. \"600\" = 10:00" },
    "endPeriod": { "type": "string", "description": "Minuten seit 00:00, z. B. \"720\" = 12:00" },
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
- **Minuten-Slots:** Einheitliche Darstellung für UI/Integrationen; halboffenes Intervall `[start, end)`.
- **Slot-Revalidierung:** Für reine Slot-Änderungen zusätzlich `startDate` oder `endDate` mitsenden, um die Prüfung zu starten.
- **Idempotenz (Empfehlung):** `Idempotency-Key` für wiederholte PATCH-Aufrufe mit Nebenwirkungen verwenden.
- **Audit:** Aktion und Ergebnis protokollieren (Wer/Was/Wann/Erfolg).


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation (minutenbasierte Slots).
