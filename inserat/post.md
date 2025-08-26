# uRent – `/inserat` API (POST)

Erstellt ein neues **Inserat** auf der uRent-Fahrzeugmietplattform.  
Bei uRent können **PKW, Lkw, Transporter & Anhänger** gemietet und/oder vermietet werden.

> **Basis-URL:** `https://<dein-host>/inserat`  
> *(Bei Next.js Route Handlers ggf. `/api/inserat` verwenden.)*

---

## Authentifizierung

- Erfordert eingeloggten Nutzer (`getCurrentUser()`).
- Ohne Session: **401 Unauthorized**

---

## Anfrage

**Methode:** `POST`  
**Header:**
- `Content-Type: application/json`
- Session-/Auth-Cookie bzw. Header je nach Setup

**Body (JSON):**
```json
{
  "category": "string",
  "title": "string",
  "userId": "string"
}
