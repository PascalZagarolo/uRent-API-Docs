# uRent API – Inserat erstellen (`POST /api/inserat`)

Erstellt ein neues Inserat auf dem uRent-Marktplatz (Pkw, Lkw, Transporter, Anhänger).

- **Base URL:** `https://urent-rental.de`
- **Endpoint:** `/api/inserat`
- **Methode:** `POST`
- **Content-Type:** `application/json`
- **Auth:** Erfordert eingeloggten Benutzer (Session-basiert). Ohne gültige Sitzung → `401 Unauthorized`.

---

## Verhalten

Beim Erstellen liest die Route **nur** diese Felder aus dem Request-Body:

- `category` *(string, erforderlich)*
- `title` *(string, erforderlich)*
- `userId` *(string, erforderlich)*

Serverseitig werden zusätzlich feste Werte gesetzt:

- `annual` wird **immer** auf `true` gesetzt (unabhängig vom Request).
- `isDaily` wird **immer** auf `true` gesetzt (unabhängig vom Request).

> **Hinweis:** Alle weiteren Eigenschaften des Inserats stammen aus Datenbank-Defaults oder werden später in anderen Flows gepflegt. Der Response enthält das vollständige, von der Datenbank zurückgegebene Inserat.

---

## Request

### JSON-Body (Minimal)
```json
{
  "category": "Transporter",
  "title": "3,5t Transporter – Tagesmiete",
  "userId": "user_abc123"
}
```

### cURL
```bash
curl -X POST "https://urent-rental.de/api/inserat"   -H "Content-Type: application/json"   --data '{
    "category": "Transporter",
    "title": "3,5t Transporter – Tagesmiete",
    "userId": "user_abc123"
  }'   -i
```

### Fetch (TypeScript/JS)
```ts
await fetch("https://urent-rental.de/api/inserat", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    category: "Transporter",
    title: "3,5t Transporter – Tagesmiete",
    userId: "user_abc123"
  }),
  credentials: "include" // Session-Cookies mitschicken
});
```

---

## Response

- **Erfolg:** `200 OK` mit dem erstellten Inserat (JSON)
- **Nicht eingeloggt:** `401 Unauthorized` (Text)
- **Serverfehler:** `500 Interner Server Error` (Text)

> **Wichtig:** In der aktuellen Implementierung wird kein `201 Created` gesetzt, sondern `200 OK`.

### Beispiel-Antwort *(InseratsObjekt)*

> **Beispiel:** Der folgende Payload zeigt ein volles Inseratsobjekt. Konkrete Werte hängen von DB-Defaults und späteren Aktualisierungen ab. **Bei der Erstellung setzt die Route standardmäßig `annual: true` und `isDaily: true`; diese Beispielwerte können daher abweichen.**

```json
{
  "id": "c2f1a0d6-3b79-4e5c-9f3b-5e2d7a4b1c2d",
  "title": "3,5t Transporter – Tagesmiete",
  "description": "Geräumiger Transporter, Nichtraucherfahrzeug. Abholung in Berlin.",
  "category": "Transporter",
  "price": null,
  "isPublished": false,
  "multi": true,
  "amount": 3,
  "isHighlighted": true,

  "emailAddress": "vermieter@example.com",
  "phoneNumber": "+49 151 23456789",

  "priceType": "FREE",

  "begin": "2025-09-01",
  "end": "2025-09-30",
  "annual": false,
  "dailyPrice": true,

  "license": null,
  "caution": "500.00",
  "reqAge": 21,
  "minTime": 1,

  "priceHour": "12.00",
  "priceWeekend": "150.00",
  "priceKilometer": "0.25",

  "color": "#00A3FF",

  "firstRelease": "2025-08-20",

  "views": 0,

  "createdAt": "2025-08-26",
  "updatedAt": "2025-08-26",

  "userId": "user_abc123",

  "addressId": "c9f4d7e2-1a3b-44d0-bc1f-6d0f6b7e5a21",

  "pkwId": null,
  "lkwId": null,
  "trailerId": null,
  "transportId": "0c1a2b3c-4d5e-678f-9012-3456789abcde"
}
```

---

## Statuscodes zusammengefasst

| Code | Bedeutung                              | Payload                 |
|------|----------------------------------------|-------------------------|
| 200  | Erfolgreich erstellt                   | JSON (Inserat)          |
| 401  | Nicht autorisiert (nicht eingeloggt)   | `"Unauthorized"`        |
| 500  | Unerwarteter Serverfehler              | `"Interner Server Error"` |

---

## Validierungs- & Implementierungsdetails

- **Erforderliche Felder:** `category`, `title`, `userId`
- **Serverseitige Defaults:** `annual: true`, `isDaily: true`
- **Antwortkörper:** Das von der Datenbank zurückgegebene Inserat (`returning()`), inkl. generierter IDs/Defaults.
- **Sicherheit:** Zugriff nur für authentifizierte Nutzer (`getCurrentUser()` muss einen Nutzer liefern).

---

## Changelog

- **2025-08-26:** Erste Dokumentation der Route `/api/inserat` (POST) für uRent.
