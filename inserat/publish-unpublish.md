# API: Inserat veröffentlichen / deaktivieren

**Route:** `PATCH /api/v1/inserat/{inseratId}/publish-unpublish`  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `INSERAT_WRITE`

Schaltet den Veröffentlichungsstatus eines Inserats **an** oder **aus**. Beim Veröffentlichen kann (standardmäßig) automatisch geprüft werden, ob das Inserat **bereit für die Veröffentlichung** ist. Zusätzlich gibt es Seiteneffekte wie **Geocoding der Adresse**, **Auto-Anlage fehlender Fahrzeuge** bei Multi-Inseraten und das **Zurücksetzen von Hervorhebungen** beim Deaktivieren.


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
  - [400 – Ungültige Inserat-ID](#400--ungültige-inserat-id)
  - [400 – Ungültiger Body](#400--ungültiger-body)
  - [400 – Validierung fehlgeschlagen](#400--validierung-fehlgeschlagen)
  - [400 – Zu viele Fahrzeuge auf einmal](#400--zu-viele-fahrzeuge-auf-einmal)
  - [401 – Nicht autorisiert](#401--nicht-autorisiert)
  - [403 – Unzureichende Berechtigungen](#403--unzureichende-berechtigungen)
  - [403 – Kein Zugriff auf dieses Inserat](#403--kein-zugriff-auf-dieses-inserat)
  - [404 – Inserat nicht gefunden](#404--inserat-nicht-gefunden)
  - [429 – Rate Limit überschritten](#429--rate-limit-überschritten)
  - [500 – Interner Fehler](#500--interner-fehler)
- [Seiteneffekte & Geschäftslogik](#seiteneffekte--geschäftslogik)
- [Beispiele](#beispiele)
  - [Veröffentlichen (mit Validierung)](#veröffentlichen-mit-validierung)
  - [Veröffentlichen (Validierung überspringen)](#veröffentlichen-validierung-überspringen)
  - [Deaktivieren (Unpublish)](#deaktivieren-unpublish)
  - [Beispiel: Validierung schlägt fehl](#beispiel-validierung-schlägt-fehl)
  - [Beispiel: Multi-Inserat Limit](#beispiel-multi-inserat-limit)
- [JSON Schemas](#json-schemas)
  - [Request-Body](#request-body-schema)
  - [200-Response (vereinfacht)](#200-response-vereinfacht)
- [Changelog](#changelog)


---

## Zweck

- **Publish/Unpublish** eines Inserats über einen einheitlichen Endpunkt.
- Optionaler **Vorab-Check** (standardmäßig aktiv) via `check-ready-for-release`.
- Automatische **Datenpflege** (z. B. Geocoding, Fahrzeuge anlegen, Highlights zurücksetzen).


---

## Zugriff & Authentifizierung

- Einer der folgenden Header ist erforderlich:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`INSERAT_WRITE`**
- Bei Rate-Limit-Verletzung kann ein `Retry-After`-Header gesetzt werden.


---

## Request

### HTTP-Methode
`PATCH`

### Pfadparameter

| Name      | Typ    | Erforderlich | Beschreibung                                   |
|-----------|--------|--------------|------------------------------------------------|
| inseratId | UUIDv4 | ja           | Eindeutige Inserat-ID. Muss eine gültige UUID sein. |

### Header

| Header        | Erforderlich | Beispiel               | Beschreibung                                   |
|---------------|-------------:|------------------------|------------------------------------------------|
| Authorization | bedingt      | `Bearer sk_live_...`   | API Key (Alternative zu `x-api-key`).          |
| x-api-key     | bedingt      | `sk_live_...`          | API Key (Alternative zu `Authorization`).      |
| Content-Type  | ja           | `application/json`     |                                                |
| Accept        | nein         | `application/json`     |                                                |

> **Hinweis:** Einer der beiden Auth-Header muss vorhanden sein.

### Body

```json
{
  "publish": true,
  "skipValidation": false
}
```

| Feld            | Typ     | Erforderlich | Standard | Beschreibung |
|-----------------|---------|--------------|---------:|--------------|
| publish         | boolean | ja           | –        | `true` = veröffentlichen, `false` = deaktivieren. |
| skipValidation  | boolean | nein         | `false`  | Wenn `true`, wird der Vorab-Check **nicht** aufgerufen. |


---

## Responses

### 200 – OK

```json
{
  "success": true,
  "inserat": {
    "...": "vollständiger Inserat-Datensatz inkl. isPublished/firstRelease etc."
  }
}
```

> Der zurückgegebene `inserat`-Datensatz entspricht dem Datenbankmodell und enthält u. a. `isPublished` (neuer Status) und `firstRelease` (beim ersten Publish gesetzt bzw. beibehalten).

---

### 400 – Ungültige Inserat-ID

```json
{ "success": false, "error": "Invalid inserat ID" }
```

---

### 400 – Ungültiger Body

Ungültiges JSON:
```json
{ "success": false, "error": "Invalid JSON in request body" }
```

Schema-Verletzung:
```json
{ "success": false, "error": "Invalid request body format" }
```

---

### 400 – Validierung fehlgeschlagen

(Ergebnis aus `check-ready-for-release` wird durchgereicht)

```json
{
  "success": false,
  "error": "Inserat is not ready for publication",
  "validationErrors": {
    "ready": false,
    "error": "Missing attributes",
    "missingAttributes": {
      "basic": ["title", "price"],
      "category": ["seats", "doors"],
      "address": true,
      "images": true
    },
    "subscription": {
      "hasSubscription": true,
      "subscriptionType": "BASIS",
      "currentInserate": 3,
      "maxInserate": 3,
      "subscriptionExpired": false
    }
  }
}
```

---

### 400 – Zu viele Fahrzeuge auf einmal

(Gilt nur für **Multi-Inserate**)

```json
{ "success": false, "error": "Too many vehicles added at once" }
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

---

### 403 – Unzureichende Berechtigungen

```json
{
  "success": false,
  "error": "Insufficient permissions",
  "missingPermissions": ["INSERAT_WRITE"]
}
```

---

### 403 – Kein Zugriff auf dieses Inserat

```json
{ "success": false, "error": "No permission to access this inserat" }
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

## Seiteneffekte & Geschäftslogik

1. **Vorab-Validierung (nur bei `publish=true` & `skipValidation=false`):**  
   Es wird `GET /api/v1/inserat/{inseratId}/check-ready-for-release` aufgerufen. Falls `ready=false`, wird **kein Publish** durchgeführt und der Fehler samt Details in `validationErrors` zurückgegeben.

2. **Multi-Inserat – Fahrzeuge automatisch anlegen:**  
   Wenn `inserat.multi = true` und weniger Fahrzeuge existieren als `inserat.amount`, werden bis zu **25** fehlende Einträge automatisch erzeugt (Titel: „Fahrzeug 01“, „Fahrzeug 02“, …). Überschreitet die Differenz **25**, wird mit `400` abgebrochen.

3. **Status-Update & Datumslogik:**  
   - `isPublished` wird auf den gewünschten Zustand gesetzt.  
   - `firstRelease` wird beim **ersten** Publish gesetzt (oder beibehalten, wenn bereits vorhanden).

4. **Unpublish – Darstellung zurücksetzen:**  
   Wenn deaktiviert (`publish=false`), werden `isHighlighted=false` und `color=null` gesetzt.

5. **Geocoding (nur beim Publish):**  
   Falls eine Adresse mit `postalCode` vorhanden ist, wird über *geocode.maps.co* versucht, `longitude`/`latitude` zu ermitteln und in der Adresse zu speichern. **Fehlschläge brechen den Publish nicht ab.**


---

## Beispiele

### Veröffentlichen (mit Validierung)

```bash
curl -s -X PATCH \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/publish-unpublish" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"publish": true}'
```

---

### Veröffentlichen (Validierung überspringen)

```bash
curl -s -X PATCH \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/publish-unpublish" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"publish": true, "skipValidation": true}'
```

---

### Deaktivieren (Unpublish)

```bash
curl -s -X PATCH \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/publish-unpublish" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"publish": false}'
```

---

### Beispiel: Validierung schlägt fehl

```bash
curl -s -X PATCH \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/publish-unpublish" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"publish": true}'
```

**Response 400**
```json
{
  "success": false,
  "error": "Inserat is not ready for publication",
  "validationErrors": {
    "ready": false,
    "error": "Missing attributes",
    "missingAttributes": {
      "basic": ["title", "price"],
      "category": ["seats", "doors"],
      "address": true,
      "images": true
    }
  }
}
```

---

### Beispiel: Multi-Inserat Limit

Wenn `amount - bestehendeFahrzeuge > 25`:

```json
{ "success": false, "error": "Too many vehicles added at once" }
```


---

## JSON Schemas

### Request-Body (Zod-Äquivalent)

```json
{
  "type": "object",
  "properties": {
    "publish": { "type": "boolean" },
    "skipValidation": { "type": "boolean", "default": false }
  },
  "required": ["publish"],
  "additionalProperties": false
}
```

### 200-Response (vereinfacht)

```json
{
  "type": "object",
  "properties": {
    "success": { "type": "boolean" },
    "inserat": { "type": ["object", "null"] },
    "error": { "type": ["string", "null"] },
    "validationErrors": { "type": ["object", "null"] }
  },
  "required": ["success"]
}
```


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation hinzugefügt.
