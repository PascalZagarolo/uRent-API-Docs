# uRent API – README (Inhaltsverzeichnis)

uRent ist ein Online-Mietmarktplatz für Fahrzeugmiete (PKW, LKW, Transporter & Anhänger).  
Diese README dient als **Inhaltsverzeichnis** und liefert kompakte Beispiele – nach **HTTP-Methode** gruppiert – mit klarer Trennung von **Request** und **Response**.

---

## Inhalt

- [Einführung](#einführung)
- [Basis-URLs & Versionierung](#basis-urls--versionierung)
- [Authentifizierung](#authentifizierung)
- [Ratenbegrenzung](#ratenbegrenzung)
- [Idempotenz](#idempotenz)
- [Paginierung](#paginierung)
- [Fehlerformat](#fehlerformat)
- [Webhooks](#webhooks)
- [GET](#get)
  - [/vehicles](#get-vehicles)
  - [/vehicles/{vehicleId}](#get-vehiclesvehicleid)
  - [/vehicles/{vehicleId}/availability](#get-vehiclesvehicleidavailability)
  - [/bookings](#get-bookings)
  - [/bookings/{bookingId}](#get-bookingsbookingid)
  - [/payments/{paymentId}](#get-paymentspaymentid)
- [POST](#post)
  - [/vehicles/{vehicleId}/quote](#post-vehiclesvehicleidquote)
  - [/bookings](#post-bookings)
  - [/bookings/{bookingId}/cancel](#post-bookingsbookingidcancel)
- [Schemas (Auszug)](#schemas-auszug)
- [Changelog & Deprecations](#changelog--deprecations)

---

## Einführung

- Alle Endpunkte sprechen JSON über HTTPS.
- Zeiten im ISO-8601-Format (`YYYY-MM-DDThh:mm:ssZ`), Währungen in ISO-4217 (z. B. `EUR`).

---

## Basis-URLs & Versionierung

- **Produktion:** `https://api.urent.com/v1`  
- **Sandbox:** `https://sandbox.api.urent.com/v1`  
- Hauptversion im Pfad (`/v1`). Breaking Changes ⇒ neue Hauptversion (`/v2`).

---

## Authentifizierung

- **Server-zu-Server:** Bearer Token (API Key)
