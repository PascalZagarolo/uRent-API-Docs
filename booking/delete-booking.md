# API: Booking löschen

**Route:** `DELETE /api/v1/booking/{bookingId}/delete`  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `BOOKING_DELETE`

Löscht eine bestehende **Buchung**. Für externe Apps wird zusätzlich geprüft, ob die Installation Zugriff auf den **Owner** des zugehörigen Inserats hat.


---

## Inhalt

- [Zweck](#zweck)
- [Zugriff & Authentifizierung](#zugriff--authentifizierung)
- [Request](#request)
  - [HTTP-Methode](#http-methode)
  - [Pfadparameter](#pfadparameter)
  - [Header](#header)
  - [Body](#body)
- [Responses](#responses)
  - [200 – OK](#200--ok)
  - [400 – Ungültiges ID-Format](#400--ungültiges-id-format)
  - [401 – Nicht autorisiert](#401--nicht-autorisiert)
  - [403 – Unzureichende Berechtigungen](#403--unzureichende-berechtigungen)
  - [403 – Kein Zugriff auf dieses Inserat](#403--kein-zugriff-auf-dieses-inserat)
  - [404 – Booking nicht gefunden](#404--booking-nicht-gefunden)
  - [404 – Inserat nicht gefunden](#404--inserat-nicht-gefunden)
  - [429 – Rate Limit überschritten](#429--rate-limit-überschritten)
  - [500 – Interner Fehler](#500--interner-fehler)
- [Feldbeschreibung (200-Response)](#feldbeschreibung-200-response)
- [Beispiele](#beispiele)
- [Hinweise](#hinweise)
- [Changelog](#changelog)


---

## Zweck

Dauerhaftes Entfernen einer Buchung.  
Die Route validiert zunächst die **Booking-ID**, prüft das Vorhandensein der Buchung und des zugehörigen **Inserats** und erzwingt eine **Mandantentrennung** für externe Integrationen.


---

## Zugriff & Authentifizierung

- Einer der folgenden Header ist erforderlich:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`BOOKING_DELETE`**
- Externe Apps: Zusätzlich wird geprüft, ob der API‑Key/Installation Zugriff auf den **Owner (`userId`)** des betroffenen Inserats hat.
- Bei Rate-Limit-Verletzung kann ein `Retry-After`-Header gesetzt werden.


---

## Request

### HTTP-Methode
`DELETE`

### Pfadparameter

| Name       | Typ    | Erforderlich | Beschreibung                                      |
|------------|--------|--------------|---------------------------------------------------|
| bookingId  | UUIDv4 | ja           | Booking-ID. Muss dem UUIDv4-Format entsprechen.  |

### Header

| Header        | Erforderlich | Beispiel               | Beschreibung |
|---------------|-------------:|------------------------|--------------|
| Authorization | bedingt      | `Bearer sk_live_...`   | API Key (Alternative zu `x-api-key`). |
| x-api-key     | bedingt      | `sk_live_...`          | API Key (Alternative zu `Authorization`). |
| Accept        | nein         | `application/json`     | |

> **Hinweis:** Einer der beiden Auth-Header muss vorhanden sein.

### Body
Kein Body.


---

## Responses

### 200 – OK

```json
{
  "success": true,
  "message": "Booking deleted successfully"
}
```

---

### 400 – Ungültiges ID-Format

```json
{ "success": false, "error": "Invalid booking ID format" }
```

---

### 401 – Nicht autorisiert

Fehlender API Key:
```json
{ "success": false, "error": "API key required" }
```

Ungültige/abgewiesene Authentifizierung:
```json
{ "success": false, "error": "Invalid API key" }
```
*(Fehlermeldung kann variieren.)*

---

### 403 – Unzureichende Berechtigungen

```json
{
  "success": false,
  "error": "Insufficient permissions",
  "missingPermissions": ["BOOKING_DELETE"]
}
```

---

### 403 – Kein Zugriff auf dieses Inserat

*(Nur bei externen Apps, wenn die Installation nicht zum Inserat-Owner passt)*
```json
{ "success": false, "error": "No permission to delete bookings for this inserat" }
```

---

### 404 – Booking nicht gefunden

```json
{ "success": false, "error": "Booking not found" }
```

---

### 404 – Inserat nicht gefunden

```json
{ "success": false, "error": "Inserat not found" }
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

## Feldbeschreibung (200-Response)

| Feld     | Typ     | Beschreibung |
|----------|---------|--------------|
| success  | boolean | `true`, wenn die Buchung gelöscht wurde. |
| message  | string  | Erfolgsnachricht. |


---

## Beispiele

### Booking löschen

```bash
curl -s -X DELETE \
  "https://urent-rental.de/api/v1/booking/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/delete" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Accept: application/json"
```

**Mögliche Antwort (200)**
```json
{ "success": true, "message": "Booking deleted successfully" }
```


---

## Hinweise

- **Nicht idempotent:** Wird eine bereits gelöschte oder nicht vorhandene `bookingId` übergeben, antwortet der Server mit **404** (kein „ok wenn schon weg“ Verhalten).
- **Mandantentrennung:** Bei externen Integrationen muss die Installation zum **Owner** des Inserats passen (zusätzliche Prüfung im Code vorhanden).
- **Audit-Empfehlung:** Für Compliance die Aktion mitführen: `who (installation/key)`, `what (route, bookingId)`, `when`, `result`.
- **Rate Limit:** Bei Überschreitung kann `Retry-After` in Sekunden gesetzt sein.


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation hinzugefügt.
