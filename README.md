# uRent API – Übersicht & Inhaltsverzeichnis (nach Domänen gruppiert)

uRent ist ein Online-Mietmarktplatz für Fahrzeugmiete (PKW, LKW, Transporter & Anhänger).  
Dieses Dokument ist eine **kompakte Übersicht** und dient als **Inhaltsverzeichnis** – jetzt nach den Domänen **Profil**, **Inserate**, **Buchungen** und **Fahrzeuge** gruppiert. Die Detailseiten werden separat gepflegt und hier **relativ verlinkt**.

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
- [Profil](#profil)
- [Inserate](#inserate)
- [Buchungen](#buchungen)
- [Fahrzeuge](#fahrzeuge)
- [Schemas (Auszug)](#schemas-auszug)
- [Changelog & Deprecations](#changelog--deprecations)
- [GitHub: Interne Links](#github-interne-links)

---

## Einführung

Diese Übersicht erklärt die grundlegenden Konzepte (Auth, Versionierung, Limits usw.) und verlinkt auf **How-tos** sowie die **API-Referenz**, gruppiert nach Domänen.  
So finden Teams schnell die relevanten Themen je Geschäftsbereich.

---

## Leitplanken

### Basis-URLs & Versionierung
- Produktion: `https://urent-rental.de/api/v1`
- Versionierung über den Pfad (`/v1`). Breaking Changes ⇒ neue Hauptversion (`/v2`).

### Authentifizierung
- **Server-zu-Server:** Bearer Token (API Key)  
  Header: `Authorization: Bearer <token>` **oder** `x-api-key: <token>`

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

## Profil

> Benutzer- & Organisationskontext, API Keys, Abonnements

**How-tos**
- **API Key verwenden** → **[profil/api-key.md](profil/api-key.md)**
- **Profil lesen/aktualisieren** → **[profil/profil-verwalten.md](profil/profil-verwalten.md)**
- **Abo/Subscription prüfen** → **[profil/subscription.md](profil/subscription.md)**

**Referenz**
- **Profil** → **[reference/profil.md](reference/profil.md)**
- **Subscription** → **[reference/subscription.md](reference/subscription.md)**

**Schemas**
- **User** → **[schemas/user.md](schemas/user.md)**
- **Subscription** → **[schemas/subscription.md](schemas/subscription.md)**

---

## Inserate

> Erstellen, prüfen und verwalten von Inseraten (Angebote/Anzeigen)

**How-tos**
- **Inserat erstellen** → **[inserat/post.md](inserat/post.md)**
- **Inserat auf Veröffentlichungsbereitschaft prüfen** → **[inserat/check-ready-for-release.md](inserat/check-ready-for-release.md)**
- **Inserat veröffentlichen/deaktivieren** → **[inserat/publish-unpublish.md](inserat/publish-unpublish.md)**

**Referenz**
- **Inserate** → **[reference/inserate.md](reference/inserate.md)**  
  (enthält u. a. `GET/POST /inserat`, `GET /inserat/{inseratId}`, `GET /inserat/{inseratId}/check-ready-for-release`)

**Schemas**
- **Inserat** → **[schemas/inserat.md](schemas/inserat.md)**
- **Adresse** → **[schemas/address.md](schemas/address.md)**
- **Bilder** → **[schemas/images.md](schemas/images.md)**
- **Kategorie-Attribute** → **[schemas/inserat-attribute.md](schemas/inserat-attribute.md)**

---

## Buchungen

> End-to-end Booking-Flow inkl. Zahlungen

**How-tos**
- **Buchung erstellen** → **[how-tos/buchung-erstellen.md](how-tos/buchung-erstellen.md)**
- **Buchung stornieren** → **[how-tos/buchung-stornieren.md](how-tos/buchung-stornieren.md)**
- **Buchungen auflisten & Details** → **[how-tos/buchungen-listen-und-details.md](how-tos/buchungen-listen-und-details.md)**
- **Zahlungsstatus prüfen** → **[how-tos/zahlung-pruefen.md](how-tos/zahlung-pruefen.md)**

**Referenz**
- **Buchungen** → **[reference/buchungen.md](reference/buchungen.md)**  
  (enthält u. a. `GET /bookings`, `GET /bookings/{bookingId}`, `POST /bookings`, `POST /bookings/{bookingId}/cancel`)
- **Zahlungen (Payments)** → **[reference/payments.md](reference/payments.md)**  
  (enthält u. a. `GET /payments/{paymentId}`)

**Schemas**
- **Booking** → **[schemas/booking.md](schemas/booking.md)**
- **Payment** → **[schemas/payment.md](schemas/payment.md)**

---

## Fahrzeuge

> Fahrzeugbestand, Verfügbarkeiten & Preisfindung (Quote)

**How-tos**
- **Verfügbarkeit prüfen** → **[how-tos/verfuegbarkeit.md](how-tos/verfuegbarkeit.md)**
- **Angebot (Quote) berechnen** → **[how-tos/quote-berechnen.md](how-tos/quote-berechnen.md)**

**Referenz**
- **Fahrzeuge** → **[reference/vehicles.md](reference/vehicles.md)**  
  (enthält u. a. `GET /vehicles`, `GET /vehicles/{vehicleId}`, `GET /vehicles/{vehicleId}/availability`, `POST /vehicles/{vehicleId}/quote`)

**Schemas**
- **Vehicle** → **[schemas/vehicle.md](schemas/vehicle.md)**

---

## Schemas (Auszug)

- **User** → **[schemas/user.md](schemas/user.md)**
- **Subscription** → **[schemas/subscription.md](schemas/subscription.md)**
- **Inserat** → **[schemas/inserat.md](schemas/inserat.md)**
- **Inserat-Attribute** → **[schemas/inserat-attribute.md](schemas/inserat-attribute.md)**
- **Address** → **[schemas/address.md](schemas/address.md)**
- **Images** → **[schemas/images.md](schemas/images.md)**
- **Booking** → **[schemas/booking.md](schemas/booking.md)**
- **Payment** → **[schemas/payment.md](schemas/payment.md)**
- **Vehicle** → **[schemas/vehicle.md](schemas/vehicle.md)**
- **Error** → **[schemas/error.md](schemas/error.md)**

---

## Changelog & Deprecations

- **2025‑08‑28:** Übersicht nach Domänen (Profil, Inserate, Buchungen, Fahrzeuge) neu strukturiert.

---

## GitHub: Interne Links

- **Relative Links** verwenden, z. B.:  
  `[Inserat erstellen](inserat/post.md)` oder `[Fahrzeuge Referenz](reference/vehicles.md)`
- **Anker** auf Überschriften innerhalb einer Datei (kebab-case), z. B.:  
  `[Beispielabschnitt](reference/vehicles.md#beispielabschnitt)`
- Pfade sind **case-sensitive**. Funktioniert in README-Dateien und Wikis identisch.
