# uRent API – Übersicht & Inhaltsverzeichnis

uRent ist ein Online-Mietmarktplatz für Fahrzeugmiete (PKW, LKW, Transporter & Anhänger).  
Dieses Dokument ist eine **kompakte Übersicht** und dient als **Inhaltsverzeichnis** für die gesamte API-Dokumentation. Die Detailseiten zu einzelnen Aktionen und Ressourcen werden separat gepflegt und hier **relativ verlinkt**.

> **Standard-Basis-URL:** `https://urent-rental.de/api/v1`

---

## Inhalt

- [Einführung](#einführung)
- [Leitplanken](#leitplanken)
  - [Basis-URLs & Versionierung](#basis-urls--versionierung)
  - [Authentifizierung](#authentifizierung)
  - [Ratenbegrenzung](#ratenbegrenzung)
  - [Idempotenz](#idempotenz)
  - [Paginierung](#paginierung)
  - [Fehlerformat](#fehlerformat)
  - [Webhooks](#webhooks)
- [Aktionen (How-tos)](#aktionen-how-tos)
- [API-Referenz (nach Ressourcen)](#api-referenz-nach-ressourcen)
  - [Vehicles](#vehicles)
  - [Bookings](#bookings)
  - [Payments](#payments)
- [Schemas (Auszug)](#schemas-auszug)
- [Changelog & Deprecations](#changelog--deprecations)
- [GitHub: Interne Links](#github-interne-links)

---

## Einführung

Diese Übersicht erklärt die grundlegenden Konzepte (Auth, Versionierung, Limits usw.) und verlinkt auf **How-tos** (aktionsorientiert) sowie die **API-Referenz** (ressourcenorientiert).  
So finden Einsteiger schnell den Happy Path, während Fortgeschrittene zielgerichtet nachschlagen können.

---

## Leitplanken

### Basis-URLs & Versionierung
- Produktion: `https://urent-rental.de/api/v1`
- Versionierung über den Pfad (`/v1`). Breaking Changes ⇒ neue Hauptversion (`/v2`).

### Authentifizierung
- **Server-zu-Server:** Bearer Token (API Key)  
  Header: `Authorization: Bearer <token>`

### Ratenbegrenzung
- Empfohlene Response-Header (Beispiele):  
  `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- Bei Überschreitung: `429 Too Many Requests` (ggf. mit `Retry-After`).

### Idempotenz
- Schreibrouten akzeptieren `Idempotency-Key: <uuid>`.
- Gleiches Key+Body ⇒ gleiche Serveraktion/-antwort; bei Konflikt `409/422`.

### Paginierung
- Cursor- oder Seiten-Paginierung (z. B. `?limit=25&cursor=...`).
- Responses enthalten `nextCursor`/`prevCursor` (falls vorhanden).

### Fehlerformat
- Einheitliches JSON-Fehlerobjekt; Details siehe **[Schemas → Error](schemas/error.md)**.

### Webhooks
- Mögliche Events: `booking.created`, `booking.canceled`, `payment.succeeded`, `payment.failed`.
- Verifikation via Signatur-Header (z. B. `X-URent-Signature`) und Timestamp.  
  Details: **[Webhooks Übersicht](webhooks/overview.md)**.

---

## Aktionen (How-tos)

> Schritt-für-Schritt-Anleitungen. Jede Seite enthält die benötigten Endpunkte in Reihenfolge, typische Fehler & Hinweise zur Idempotenz.

- **Inserat erstellen** → **[inserat/post.md](inserat/post.md)**  
- **Verfügbarkeit prüfen** → **[how-tos/verfuegbarkeit.md](how-tos/verfuegbarkeit.md)**
- **Angebot (Quote) berechnen** → **[how-tos/quote-berechnen.md](how-tos/quote-berechnen.md)**
- **Buchung erstellen** → **[how-tos/buchung-erstellen.md](how-tos/buchung-erstellen.md)**
- **Buchung stornieren** → **[how-tos/buchung-stornieren.md](how-tos/buchung-stornieren.md)**
- **Buchungen auflisten & Details** → **[how-tos/buchungen-listen-und-details.md](how-tos/buchungen-listen-und-details.md)**
- **Zahlungsstatus prüfen** → **[how-tos/zahlung-pruefen.md](how-tos/zahlung-pruefen.md)**

---

## API-Referenz (nach Ressourcen)

> Vollständige Referenz mit Parametern, Statuscodes und Beispielen. Pro Ressource eine Seite.

### Vehicles
- Übersicht & Endpunkte: **[reference/vehicles.md](reference/vehicles.md)**  
  Enthält u. a.: `GET /vehicles`, `GET /vehicles/{vehicleId}`,  
  `GET /vehicles/{vehicleId}/availability`, `POST /vehicles/{vehicleId}/quote`

### Bookings
- Übersicht & Endpunkte: **[reference/bookings.md](reference/bookings.md)**  
  Enthält u. a.: `GET /bookings`, `GET /bookings/{bookingId}`,  
  `POST /bookings`, `POST /bookings/{bookingId}/cancel`

### Payments
- Übersicht & Endpunkte: **[reference/payments.md](reference/payments.md)**  
  Enthält u. a.: `GET /payments/{paymentId}`

---

## Schemas (Auszug)

- **Vehicle** → **[schemas/vehicle.md](schemas/vehicle.md)**
- **Booking** → **[schemas/booking.md](schemas/booking.md)**
- **Payment** → **[schemas/payment.md](schemas/payment.md)**
- **Error** → **[schemas/error.md](schemas/error.md)**

---

## Changelog & Deprecations

- **2025‑08‑28:** Erste Übersicht & Inhaltsverzeichnis angelegt.

---

## GitHub: Interne Links

- **Relative Links** verwenden, z. B.:  
  `[Inserat erstellen](inserat/post.md)`
- **Anker** auf Überschriften innerhalb einer Datei (kebab-case), z. B.:  
  `[Beispielabschnitt](reference/vehicles.md#beispielabschnitt)`
- Pfade sind **case-sensitive**. Funktioniert in README-Dateien und Wikis identisch.
