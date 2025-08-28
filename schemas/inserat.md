# uRent – Schema Dokumentation: **Inserat**

**Base URL (Kontext):** `urent-rental.de/api/v1/..`  
*Hinweis:* Diese Datei beschreibt **ausschließlich das Schema** rund um das Inserat: Felder, Enums, Beziehungen, Einbettungs-Formate und Beispiel-JSONs. Es listet **keine Endpunkte** auf.

---

## Zweck

Ein **Inserat** repräsentiert ein Angebot zur Vermietung eines Fahrzeugs/Transportmittels oder eines Anhängers innerhalb von uRent. Es enthält allgemeine Felder (Titel, Preis, Sichtbarkeit), optionale Preislogiken sowie **kategoriespezifische Attribute** (PKW, LKW, TRAILER, TRANSPORT). Ein Inserat gehört genau **einem** Benutzer (Vermieter) und kann Bilder, Adresse, Zusatzoptionen, Preisprofile und Buchungs-/Anfrage-Daten referenzieren.

---

## Kernmodell: Tabelle `inserat`

**Primärschlüssel:** `id` (`uuid`, default `gen_random_uuid()`)

| Feld | Typ | Pflicht | Default | Beschreibung |
|---|---|---:|---|---|
| `id` | `uuid` | – | generiert | Eindeutige Inserat-ID |
| `title` | `text` | **Ja** | – | Titel des Inserats |
| `description` | `text` | – | – | Beschreibung |
| `category` | `enum category` | – | – | Kategorie: `PKW` \| `LKW` \| `TRAILER` \| `TRANSPORT` |
| `price` | `decimal` | – | – | Basispreis (String-Repräsentation empfohlen) |
| `isPublished` | `boolean` | **Ja** | `false` | Sichtbarkeit |
| `multi` | `boolean` | **Ja** | `false` | Mehrfachverfügbarkeit (gleiche Einheit mehrfach vorhanden) |
| `amount` | `integer` | **Ja** | `1` | Anzahl verfügbarer Einheiten (nur relevant bei `multi`) |
| `isHighlighted` | `boolean` | **Ja** | `false` | Hervorhebung/Featured |
| `emailAddress` | `text` | – | – | Kontakt-E-Mail (anzeigen abhängig von Berechtigungen) |
| `phoneNumber` | `text` | – | – | Kontakt-Telefon |
| `priceType` | `enum priceType` | – | `FREE` | Tarif/Plan: `FREE` \| `BASIS` \| `PREMIUM` \| `ENTERPRISE` |
| `begin` / `end` | `timestamp` | – | – | Gültigkeits-/Verfügbarkeitszeitraum |
| `annual` | `boolean` | **Ja** | `false` | Jahrespreislogik aktiviert |
| `dailyPrice` | `boolean` | **Ja** | `false` | Tagespreislogik aktiviert |
| `license` | `enum license` | – | – | Erforderlicher Führerschein: `B`, `BE`, `C1`, `C`, `CE`, `CE1` |
| `caution` | `decimal` | – | – | Kaution |
| `reqAge` | `integer` | – | – | Mindestalter |
| `minTime` | `integer` | – | – | Mindestmietdauer (projektspezifische Einheit, oft Stunden) |
| `priceHour` | `decimal` | – | – | Stundensatz |
| `priceWeekend` | `decimal` | – | – | Wochenendpauschale |
| `priceKilometer` | `decimal` | – | – | Kilometerpreis |
| `color` | `text` | – | – | Farbe |
| `firstRelease` | `timestamp` | – | – | Erstzulassung |
| `views` | `integer` | **Ja** | `0` | View-Zähler |
| `createdAt` | `timestamp` | – | `now()` | Erstellzeit |
| `updatedAt` | `timestamp` | – | `now()` | Änderungszeit |
| `userId` | `text` FK→`user.id` | **Ja** | – | Eigentümer (Vermieter) |
| `addressId` | `uuid` FK→`address.id` | – | – | Adresse/Geoposition |
| `pkwId` | `uuid` FK→`pkwAttribute.id` | – | – | PKW-Attribute |
| `lkwId` | `uuid` FK→`lkwAttribute.id` | – | – | LKW-Attribute |
| `trailerId` | `uuid` FK→`trailerAttribute.id` | – | – | Anhänger-Attribute |
| `transportId` | `uuid` FK→`transportAttribute.id` | – | – | Transporter-Attribute |

### Enums (relevant für Inserat)
- **category:** `PKW`, `LKW`, `TRAILER`, `TRANSPORT`
- **priceType:** `FREE`, `BASIS`, `PREMIUM`, `ENTERPRISE`
- **license:** `B`, `BE`, `C1`, `C`, `CE`, `CE1`

### Datentyp-Konventionen
- **Decimal**-Werte in API-Repräsentationen als **Strings** (z. B. `"89.00"`) zur Vermeidung von Rundungsfehlern.
- **Zeitstempel** als ISO‑8601 (`UTC`), z. B. `"2025-08-20T09:10:04Z"`.

### Validierungs-/Konsistenzregeln (empfohlen)
- `title` nicht leer; `userId` muss existieren.
- **Kategorie-Attribute:** Zu einer `category` gehört **genau eine** passende Attributtabelle (siehe unten).  
- `decimal`-Felder ≥ 0.  
- Bei `multi = true` → `amount ≥ 1`.
- Falls `begin` & `end` gesetzt sind: `begin ≤ end`.

---

## Beziehungen rund um Inserat

Ein Inserat referenziert oder wird referenziert von mehreren Entitäten. Hier ein Überblick:

```
user (1) ──< inserat (n)

inserat (1) ──< images (n)
inserat (1) ──1 address (0..1)

inserat (1) ──1 pkwAttribute (0..1)
inserat (1) ──1 lkwAttribute (0..1)
inserat (1) ──1 trailerAttribute (0..1)
inserat (1) ──1 transportAttribute (0..1)

inserat (1) ──< priceprofile (n)
inserat (1) ──< extrasObject (n)

inserat (1) ──< bookingRequest (n)
inserat (1) ──< booking (n)
inserat (1) ──< rentRequest (n) ──1 paymentIntent (0..1) ──< updatesPaymentIntent (n)
                           └─1 payment (0..1)

inserat (1) ──< favourite (n)
inserat (1) ──< message (n)
```

**Lösch-/Update-Verhalten (Auszug):**
- Viele Relationen sind mit `ON DELETE CASCADE` definiert (z. B. `images`, `booking`, `bookingRequest`, `priceprofile`, `extrasObject`).  
- Kategorie-Attribut-Tabellen sind jeweils per FK an `inserat.id` gebunden (in der Attribut-Tabelle `...Attribute.inseratId`).

---

## Kategorie-spezifische Attribut-Tabellen

> Genau **eine** der folgenden Strukturen ist für ein Inserat relevant – abhängig von `category`.

### 1) `pkwAttribute`
Wichtige Felder (Auszug):  
`brand (enum brand)`, `model (text)`, `seats (int)`, `doors (int)`,  
`transmission (enum transmission)`, `type (enum carType)`, `fuel (enum fuelType)`,  
`power (int)`, `ahk (bool)`, diverse Lade-/Maßangaben.

- Enums (Auszug):  
  - `transmission`: `MANUAL`, `AUTOMATIC`, `SEMI_AUTOMATIC`  
  - `carType`: `KOMBI`, `COUPE`, `PICKUP`, `SUV`, `LIMOUSINE`, `VAN`, `KASTENWAGEN`, `KLEINBUS`, `CABRIO`, `KLEIN`, `SPORT`, `SUPERSPORT`  
  - `fuelType`: `ELEKTRISCH`, `DIESEL`, `BENZIN`, `HYBRID`

### 2) `lkwAttribute`
Wichtige Felder (Auszug):  
`lkwBrand (enum lkwBrand)`, `model`, `axis (int)`, `payload (int)`,  
`drive (enum drive)`, `application (enum application)`, `transmission`, `fuel`, `power`, Maße/Ladeangaben, `ahk`.

### 3) `trailerAttribute`
Wichtige Felder (Auszug):  
`type (enum trailer)`, `coupling (enum coupling)`, `loading (enum loading)`, `extraType (enum extraType)`,  
`axis (int)`, `weightClass (int)`, `payload (int)`, `brake (bool)`, Maße/Ladeangaben.

### 4) `transportAttribute`
Wichtige Felder (Auszug):  
`transportBrand (enum transportBrand)`, `seats (int)`, `doors (int)`, `payload (int)`,  
`transmission`, `fuel`, `power (int)`, `ahk (bool)`, Maße/Ladeangaben.

> **Hinweis:** Die vollständigen Enum-Werte (Marken/Arten) entnimmst du den Tabellen `brandEnum`, `lkwBrandEnum`, `trailerEnum`, `transportBrandEnum`, `loadingEnum`, `extraTypeEnum`, `driveEnum`, `applicationEnum` etc. im Datenmodell.

---

## Unterstützende Entitäten (Inserat-nah)

### `address`
| Feld | Typ | Beschreibung |
|---|---|---|
| `postalCode` | `text` | PLZ |
| `state` | `text` | Bundesland/Kürzel |
| `locationString` | `text` | Freitext-Ort |
| `latitude`, `longitude` | `text` | Koordinaten als String |
| `inseratId` | `uuid` | FK auf Inserat |

### `images`
| Feld | Typ | Beschreibung |
|---|---|---|
| `url` | `text` | Bild-URL |
| `position` | `integer` | Reihenfolge |
| `inseratId` | `uuid` | FK auf Inserat |

### `priceprofile`
Preisprofile erlauben flexible Tarife (z. B. Wochenende, Tag, Stunde).  
Wichtige Felder: `title`, `description`, `price (decimal)`, `durationValue (int)`, `durationUnit (enum durationUnit: HOUR, DAY, WEEK, MONTH, YEAR)`, `weekdays (text[])`, `isArchived (bool)`, `position (int)`, `inseratId`.

### `extrasObject`
Zusatzoptionen/Zubehör pro Inserat.  
Felder: `title`, `description`, `price (decimal)`, `inseratId`.

### Buchung & Anfragen (nur Kontext)
- `booking` / `bookingRequest`: Belegung/Anfragen mit `startDate/endDate`, Perioden, optional `vehicle`.  
- `rentRequest`: Angebots-/Zahlungsfluss mit `priceProfileId`, `paymentIntentId`, `totalPrice`, `status` (`SENT|REJECTED|ACCEPTED|EXPIRED|COUNTER`), `expired_at`.  
- `paymentIntent`, `payment`: Zahlungsdetails/Stati (ausführlich in ihren jeweiligen Tabellen).

---

## JSON-Beispiele

### A) Kompakte Inserat-Repräsentation
```json
{
  "id": "e7a4b7a1-3a62-4a6f-9a3c-1a4c1b8d6f01",
  "title": "VW Crafter L3H3",
  "description": "Hoher Kastenwagen, ideal für Umzug",
  "category": "TRANSPORT",
  "price": "89.00",
  "priceType": "BASIS",
  "isPublished": true,
  "isHighlighted": false,
  "multi": false,
  "amount": 1,
  "license": "B",
  "priceHour": "12.00",
  "priceWeekend": "0.00",
  "priceKilometer": "0.35",
  "reqAge": 18,
  "minTime": 2,
  "views": 123,
  "userId": "user_123",
  "createdAt": "2025-08-10T08:35:20Z",
  "updatedAt": "2025-08-20T09:10:04Z"
}
```

### B) Ausführliches Inserat mit Einbettungen
```json
{
  "id": "e7a4b7a1-3a62-4a6f-9a3c-1a4c1b8d6f01",
  "title": "Mercedes Sprinter 316",
  "description": "3,5t, lang/hoch, ideal für Umzüge",
  "category": "TRANSPORT",
  "price": "95.00",
  "priceType": "BASIS",
  "isPublished": true,
  "multi": false,
  "amount": 1,
  "annual": false,
  "dailyPrice": true,
  "license": "B",
  "priceHour": "12.00",
  "priceKilometer": "0.35",
  "reqAge": 18,
  "minTime": 2,
  "color": "weiß",
  "firstRelease": "2022-06-01T00:00:00Z",
  "views": 456,
  "userId": "user_123",
  "createdAt": "2025-08-10T08:35:20Z",
  "updatedAt": "2025-08-20T09:10:04Z",

  "address": {
    "id": "addr_1",
    "postalCode": "70173",
    "state": "BW",
    "locationString": "Stuttgart",
    "latitude": "48.778",
    "longitude": "9.180"
  },

  "attributes": {
    "transport": {
      "id": "ta_1",
      "transportBrand": "Mercedes_Benz",
      "seats": 3,
      "doors": 4,
      "payload": 1200,
      "transmission": "MANUAL",
      "fuel": "DIESEL",
      "power": 120,
      "ahk": true,
      "loading_l": "3.2",
      "loading_b": "1.7",
      "loading_h": "1.9",
      "loading_volume": "10.3"
    }
  },

  "images": [
    { "id": "img1", "position": 1, "url": "https://cdn.example/sprinter-front.jpg" },
    { "id": "img2", "position": 2, "url": "https://cdn.example/sprinter-rear.jpg" }
  ],

  "priceprofiles": [
    { "id": "pp1", "title": "Tagestarif", "price": "95.00", "durationValue": 1, "durationUnit": "DAY" },
    { "id": "pp2", "title": "Wochenende", "price": "199.00", "durationValue": 2, "durationUnit": "DAY" }
  ],

  "extras": [
    { "id": "ex1", "title": "Sackkarre", "description": "Faltbar", "price": "5.00" },
    { "id": "ex2", "title": "Zurrgurte", "price": "3.00" }
  ],

  "stats": {
    "views": 456
  }
}
```

---

## Hinweise zur Darstellung/Serialisierung

- **Attribute-Union:** In der JSON-Repräsentation sollte unter `attributes` **genau ein** Objekt erscheinen, dessen Schlüssel der Kategorie entspricht (`pkw` \| `lkw` \| `trailer` \| `transport`).  
- **Dezimalzahlen** als String; **Zeitstempel** als ISO‑8601 in UTC.  
- **Kontaktdaten** (`emailAddress`, `phoneNumber`) nur anzeigen, wenn Datenschutz-/Berechtigungsregeln erfüllt sind.  
- **views** atomar inkrementieren (z. B. per `UPDATE ... SET views=views+1`).

---

## Zusammenfassung

- **Inserat** ist das zentrale Angebotsobjekt mit allgemeinem Preis-/Sichtbarkeits-Setup.  
- **Kategorie-Attribute** sind strikt einzuordnen (PKW/LKW/Trailer/Transport).  
- **Beziehungen** zu Bildern, Adresse, Preisprofilen, Extras sowie Buchungs- und Zahlungs-Entitäten ermöglichen eine vollständige Vermietlogik.  
- **API-Repräsentationen** sollten konsistente Datentypen (Decimal als String, ISO‑Zeit) und eine klare Attribute-Union liefern.
