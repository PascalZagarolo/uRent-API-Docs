# API: Inserat – Veröffentlichungsbereitschaft prüfen

**Route:** `GET /api/v1/inserat/{inseratId}/check-ready-for-release`  
**Stabilität:** Beta  
**Version:** v1  
**Basis-URL (Prod):** `https://urent-rental.de/api/v1`  
**Auth:** API Key (`Authorization: Bearer <key>` **oder** `x-api-key: <key>`)  
**Erforderliche Berechtigung:** `INSERAT_READ`

Diese Route prüft, ob ein Inserat **veröffentlicht** werden kann. Neben `ready: true|false` liefert die Antwort Informationen zu **fehlenden Attributen** und zur **Subscription** (Abo-Limits, Ablaufstatus, aktueller Verbrauch).


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
  - [200 – OK (prüfergebnis)](#200--ok-prüfergebnis)
  - [400 – Ungültige Inserat-ID](#400--ungültige-inserat-id)
  - [401 – Nicht autorisiert](#401--nicht-autorisiert)
  - [403 – Unzureichende Berechtigungen](#403--unzureichende-berechtigungen)
  - [404 – Inserat nicht gefunden](#404--inserat-nicht-gefunden)
  - [429 – Rate Limit überschritten](#429--rate-limit-überschritten)
  - [500 – Interner Fehler](#500--interner-fehler)
- [Feldbeschreibung (200-Response)](#feldbeschreibung-200-response)
- [Erforderliche Felder & Regeln](#erforderliche-felder--regeln)
- [Beispiele](#beispiele)
  - [Beispiel: Erfolgreich & bereit](#beispiel-erfolgreich--bereit)
  - [Beispiel: Fehlende Felder](#beispiel-fehlende-felder)
  - [Beispiel: Abo abgelaufen](#beispiel-abo-abgelaufen)
  - [Beispiel: Abo-Limit erreicht](#beispiel-abo-limit-erreicht)
  - [Beispiel: 401 Kein API Key](#beispiel-401-kein-api-key)
  - [Beispiel: 403 Fehlende Berechtigungen](#beispiel-403-fehlende-berechtigungen)
  - [Beispiel: 404 Nicht gefunden](#beispiel-404-nicht-gefunden)
  - [Beispiel: 400 Ungültige ID](#beispiel-400-ungültige-id)
- [JSON Schema (200-Response)](#json-schema-200-response)
- [Changelog](#changelog)


---

## Zweck

Valider Check, ob ein Inserat **veröffentlichungsbereit** ist. Die Antwort enthält:
- `ready: true|false`
- optional **Fehler-/Hinweistexte** in `error`
- **fehlende Attribute** (grundlegend, kategoriespezifisch, Adresse, Bilder)
- **Abo-Informationen** inkl. aktuellem Verbrauch und Limit


---

## Zugriff & Authentifizierung

- Erlaubte Header für API Key:
  - `Authorization: Bearer <api-key>`
  - **oder** `x-api-key: <api-key>`
- Erforderliche Berechtigung: **`INSERAT_READ`**  
- Bei Rate-Limit-Verletzung kann ein `Retry-After`-Header gesetzt werden.


---

## Request

### HTTP-Methode
`GET`

### Pfadparameter

| Name        | Typ    | Erforderlich | Beschreibung                                   |
|-------------|--------|--------------|------------------------------------------------|
| inseratId   | UUIDv4 | ja           | Eindeutige Inserat-ID. Muss eine gültige UUID sein. |

### Header

| Header        | Erforderlich | Beispiel               | Beschreibung                                   |
|---------------|-------------:|------------------------|------------------------------------------------|
| Authorization | bedingt      | `Bearer sk_live_...`   | API Key (Alternative zu `x-api-key`).          |
| x-api-key     | bedingt      | `sk_live_...`          | API Key (Alternative zu `Authorization`).      |
| Accept        | nein         | `application/json`     |                                                |

> **Hinweis:** Einer der beiden Auth-Header (`Authorization` **oder** `x-api-key`) muss vorhanden sein.

### Query-Parameter
Keine.


---

## Responses

### 200 – OK (Prüfergebnis)

**Bereit zur Veröffentlichung:**

```json
{
  "ready": true,
  "subscription": {
    "hasSubscription": true,
    "subscriptionType": "PREMIUM",
    "currentInserate": 2,
    "maxInserate": 10,
    "subscriptionExpired": false
  }
}
```

**Nicht bereit (fehlende Angaben):**

```json
{
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
    "currentInserate": 1,
    "maxInserate": 3,
    "subscriptionExpired": false
  }
}
```

**Nicht bereit (Abo abgelaufen):**

```json
{
  "ready": false,
  "error": "Subscription has expired",
  "subscription": {
    "hasSubscription": true,
    "subscriptionType": "PREMIUM",
    "currentInserate": 2,
    "maxInserate": 10,
    "subscriptionExpired": true
  }
}
```

**Nicht bereit (Abo-Limit erreicht):**

```json
{
  "ready": false,
  "error": "Maximum number of published inserate (3) reached for your subscription",
  "subscription": {
    "hasSubscription": true,
    "subscriptionType": "BASIS",
    "currentInserate": 3,
    "maxInserate": 3,
    "subscriptionExpired": false
  }
}
```

> **Hinweis:** Für `subscriptionType = "ENTERPRISE"` ist das Limit **unbegrenzt**. In JSON kann `maxInserate` hierbei als `null` erscheinen (unlimited).


---

### 400 – Ungültige Inserat-ID

```json
{
  "ready": false,
  "error": "Invalid inserat ID"
}
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
*(Fehlermeldung kann je nach Auth-Fehler variieren.)*

---

### 403 – Unzureichende Berechtigungen

```json
{
  "error": "Insufficient permissions",
  "missingPermissions": ["INSERAT_READ"]
}
```

---

### 404 – Inserat nicht gefunden

```json
{
  "ready": false,
  "error": "Inserat not found"
}
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
{
  "ready": false,
  "error": "Internal server error"
}
```


---

## Feldbeschreibung (200-Response)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| ready | boolean | `true`, wenn Veröffentlichung möglich; sonst `false`. |
| error | string \| undefined | Begründung, warum `ready=false` (falls vorhanden). |
| missingAttributes | object \| undefined | Detail zu fehlenden Pflichtangaben. |
| missingAttributes.basic | string[] \| undefined | Fehlende **grundlegende** Felder (siehe unten). |
| missingAttributes.category | string[] \| undefined | Fehlende **kategoriespezifische** Felder. |
| missingAttributes.address | boolean \| undefined | `true`, wenn **Adresse** fehlt. |
| missingAttributes.images | boolean \| undefined | `true`, wenn **mindestens ein Bild** fehlt. |
| subscription | object | Zusammenfassung der Abo-Prüfung. |
| subscription.hasSubscription | boolean | Ob ein aktives Abo vorhanden ist. |
| subscription.subscriptionType | "FREE" \| "BASIS" \| "PREMIUM" \| "ENTERPRISE" \| null | Typ des Abos. |
| subscription.currentInserate | number | Aktuelle Anzahl **veröffentlichter** Inserate des Nutzers. |
| subscription.maxInserate | number \| null | Maximale erlaubte Veröffentlichungen. **Bei `ENTERPRISE` ggf. `null` (unbegrenzt).** |
| subscription.subscriptionExpired | boolean | `true`, wenn das Abo abgelaufen ist. |


---

## Erforderliche Felder & Regeln

### Grundlegende Pflichtfelder (alle Kategorien)
- `title`
- `category` (einer von `PKW`, `LKW`, `TRAILER`, `TRANSPORT`)
- `price`
- `emailAddress`
- `phoneNumber`

### Kategoriespezifische Pflichtfelder

**PKW**
- `brand`, `model`, `seats`, `doors`, `transmission`, `fuel`

**LKW**
- `lkwBrand`, `model`, `seats`, `axis`, `power`, `fuel`, `transmission`, `weightClass`, `payload`

**TRAILER**
- `type`, `coupling`, `axis`, `weightClass`, `payload`, `brake`

**TRANSPORT** (Transporter)
- `transportBrand`, `seats`, `doors`, `fuel`, `transmission`, `weightClass`, `payload`

> Die kategoriespezifischen Felder werden in den jeweiligen Attribut-Objekten erwartet (`pkwAttribute`, `lkwAttribute`, `trailerAttribute`, `transportAttribute`).

### Weitere Anforderungen
- **Adresse** muss vorhanden sein (`address`).  
- **Mindestens ein Bild** muss hinterlegt sein (`images.length > 0`).

### Abo-Limits (Überblick)
- `FREE`: **0** Veröffentlichungen (Publizieren nicht erlaubt).
- `BASIS`: **3** Veröffentlichungen.
- `PREMIUM`: **10** Veröffentlichungen.
- `ENTERPRISE`: **unbegrenzt** (JSON `maxInserate` kann `null` sein).

> Zusätzlich wird geprüft, ob das Abo **abgelaufen** ist (`subscriptionExpired`).


---

## Beispiele

### Beispiel: Erfolgreich & bereit

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/check-ready-for-release" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Accept: application/json"
```

**Response 200**
```json
{
  "ready": true,
  "subscription": {
    "hasSubscription": true,
    "subscriptionType": "PREMIUM",
    "currentInserate": 2,
    "maxInserate": 10,
    "subscriptionExpired": false
  }
}
```

---

### Beispiel: Fehlende Felder

**Response 200**
```json
{
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
    "currentInserate": 1,
    "maxInserate": 3,
    "subscriptionExpired": false
  }
}
```

---

### Beispiel: Abo abgelaufen

**Response 200**
```json
{
  "ready": false,
  "error": "Subscription has expired",
  "subscription": {
    "hasSubscription": true,
    "subscriptionType": "PREMIUM",
    "currentInserate": 2,
    "maxInserate": 10,
    "subscriptionExpired": true
  }
}
```

---

### Beispiel: Abo-Limit erreicht

**Response 200**
```json
{
  "ready": false,
  "error": "Maximum number of published inserate (3) reached for your subscription",
  "subscription": {
    "hasSubscription": true,
    "subscriptionType": "BASIS",
    "currentInserate": 3,
    "maxInserate": 3,
    "subscriptionExpired": false
  }
}
```

---

### Beispiel: 401 Kein API Key

```bash
curl -s -X GET \
  "https://urent-rental.de/api/v1/inserat/9b9f4a2d-1e88-4d1b-8b6c-6a1c9a3d6e12/check-ready-for-release"
```

**Response 401**
```json
{ "error": "API key required" }
```

---

### Beispiel: 403 Fehlende Berechtigungen

**Response 403**
```json
{
  "error": "Insufficient permissions",
  "missingPermissions": ["INSERAT_READ"]
}
```

---

### Beispiel: 404 Nicht gefunden

**Response 404**
```json
{
  "ready": false,
  "error": "Inserat not found"
}
```

---

### Beispiel: 400 Ungültige ID

**Response 400**
```json
{
  "ready": false,
  "error": "Invalid inserat ID"
}
```


---

## JSON Schema (200-Response)

> Vereinfachtes Schema für erfolgreiche **200**-Antworten (unabhängig von `ready`). Fehlerantworten der Statuscodes 4xx/5xx können abweichen (siehe Beispiele oben).

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "CheckReadyResponse",
  "type": "object",
  "properties": {
    "ready": { "type": "boolean" },
    "error": { "type": "string" },
    "missingAttributes": {
      "type": "object",
      "properties": {
        "basic": {
          "type": "array",
          "items": { "type": "string" }
        },
        "category": {
          "type": "array",
          "items": { "type": "string" }
        },
        "images": { "type": "boolean" },
        "address": { "type": "boolean" }
      },
      "additionalProperties": false
    },
    "subscription": {
      "type": "object",
      "properties": {
        "hasSubscription": { "type": "boolean" },
        "subscriptionType": {
          "type": ["string", "null"],
          "enum": ["FREE", "BASIS", "PREMIUM", "ENTERPRISE", null]
        },
        "currentInserate": { "type": "number" },
        "maxInserate": { "type": ["number", "null"] },
        "subscriptionExpired": { "type": "boolean" }
      },
      "required": [
        "hasSubscription",
        "subscriptionType",
        "currentInserate",
        "maxInserate",
        "subscriptionExpired"
      ],
      "additionalProperties": false
    }
  },
  "required": ["ready"],
  "additionalProperties": true
}
```


---

## Changelog

- **2025-08-28:** Erste Fassung der Routen-Dokumentation hinzugefügt.
