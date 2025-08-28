# uRent – Schema Dokumentation: **Booking**

**Base URL (Kontext):** `urent-rental.de/api/v1/..`  
*Hinweis:* Diese Datei beschreibt **ausschließlich das Schema** rund um **Booking** (inkl. Availability-Blocks) – Felder, Beziehungen, Einbettungs-Formate und Beispiel-JSONs. **Keine Endpunkte**.

---

## Zweck

Ein **Booking**-Datensatz repräsentiert entweder
1) eine **feste Reservierung/Buchung** eines Inserats (typisch nach Angebot/Bezahlung), oder  
2) eine **Verfügbarkeitsblockierung** (*isAvailability = true*), z. B. Werkstatttermin, Reinigung, interne Sperre – **kein Kundentermin**.

> **Vehicle-Hinweis:** Das Feld `vehicleId` ist **nur relevant bei Mehrfachinseraten** (wenn ein Inserat mehrere konkrete Fahrzeuge verwaltet). Damit können Buchungen/Blocks **fahrzeuggenau** gesetzt werden.

---

## Kernmodell: Tabelle `booking`

**Primärschlüssel:** `id` (`uuid`, default `gen_random_uuid()`)

| Feld | Typ | Pflicht | Default | Beschreibung |
|---|---|---:|---|---|
| `id` | `uuid` | – | generiert | Eindeutige Booking-ID |
| `inseratId` | `uuid` FK→`inserat.id` | **Ja** | – | Zugehöriges Inserat |
| `userId` | `text` FK→`user.id` | – | – | Buchender Benutzer (bei Availability-Blocks oft `null`) |
| `vehicleId` | `uuid` FK→`vehicle.id` | – | – | **Nur relevant bei Mehrfachinseraten** (fahrzeuggenaue Belegung) |
| `name` | `text` | – | – | Freitext (z. B. Kundenname, Betreff) |
| `buchungsnummer` | `text` | – | – | Externe/Interne Buchungsnummer |
| `content` | `text` | – | – | Zusatzinfos/Notizen |
| `startDate` | `timestamp` (mit TZ) | – | – | Beginn der Belegung |
| `endDate` | `timestamp` (mit TZ) | – | – | Ende der Belegung |
| `startPeriod` | `text` | – | – | Freitext-Zeitfenster (z. B. „MORNING“ oder „08:00–12:00“) |
| `endPeriod` | `text` | – | – | Freitext-Zeitfenster |
| `isAvailability` | `boolean` | **Nein** | `false` | **true** = Blockierung (Werkstatt etc.), **false** = Kundenbuchung |
| `createdAt` | `timestamp` | – | `now()` | Erstellzeitpunkt |

### Datentyp-/Serialisierungs-Konventionen
- **Zeitstempel** als ISO‑8601 mit Zeitzone, z. B. `"2025-08-28T09:00:00Z"`.
- Empfohlene Intervalllogik: `startDate` **inklusive**, `endDate` **exklusiv**, um Rück‑zu‑Rück‑Belegungen ohne Konflikt zu erlauben.

### Validierungs-/Konsistenzregeln (empfohlen)
- Wenn `startDate` & `endDate` gesetzt: **`startDate < endDate`**.
- **Mehrfachinserate:** Wenn pro physischem Fahrzeug geplant → `vehicleId` setzen, um Überbuchungen je Fahrzeug zu vermeiden.
- **Availability‑Blocks:** `isAvailability = true` → kein Kunde nötig; `userId` optional/leer.
- **Periodenfelder:** Freitext (z. B. „MORNING/AFTERNOON“, „08:00–12:00“) – appseitig validieren/normalisieren, falls nötig.

---

## Beziehungen rund um Booking

```
inserat (1) ──< booking (n)
user    (1) ──< booking (n)           (optional, bei Kundenbuchung)
vehicle (1) ──< booking (n)           (optional; nur bei Mehrfachinseraten sinnvoll)
```

**Löschverhalten:**  
- `booking.inseratId` → `ON DELETE CASCADE` (Buchungen werden mit dem Inserat gelöscht)  
- `booking.vehicleId` → `ON DELETE CASCADE` (fahrzeugbezogene Buchungen werden mit dem Fahrzeug gelöscht)

**Kontext-Entitäten (nicht Teil dieses Schemas, aber angrenzend):**
- `bookingRequest` (Anfragen)  
- `rentRequest` / `confirmedBooking` (Angebots-/Zahlungsfluss)  
> Ein produktiver Flow ist z. B.: Anfrage → Angebot (rentRequest) → Zahlung/Bestätigung → **Booking** erstellen.

---

## Anwendungslogik & Konflikte

### Konflikterkennung (Überlappung)
Zwei Belegungen `A` und `B` überlappen, wenn:  
`A.start < B.end AND A.end > B.start` (unter Annahme end‑exklusiv).

**Mehrfachinserate:** Prüfe Überlappungen **je `vehicleId`**.  
**Einzelfahrzeug-Inserate:** Prüfe Überlappungen **je `inseratId`** (wenn `vehicleId` nicht verwendet wird).

### Availability-Blocks
- `isAvailability = true` kennzeichnet **interne Sperren** (Werkstatt, Reinigung, Urlaub, Übergabe-Puffer).  
- Diese Blöcke sind **gleichrangig** zu Kundenbuchungen bei der Konfliktprüfung.

---

## JSON-Beispiele

### 1) Kundenbuchung (Einzelfahrzeug-Inserat, ohne `vehicleId`)
```json
{
  "id": "b6c9dc3a-6c1d-4b7a-8a1b-2f47c9d9a001",
  "inseratId": "e7a4b7a1-3a62-4a6f-9a3c-1a4c1b8d6f01",
  "userId": "user_123",
  "name": "Max Mustermann",
  "buchungsnummer": "UR-2025-000123",
  "content": "Umzug am Samstag",
  "startDate": "2025-09-06T08:00:00Z",
  "endDate": "2025-09-07T08:00:00Z",
  "startPeriod": "MORNING",
  "endPeriod": "MORNING",
  "isAvailability": false,
  "createdAt": "2025-08-28T10:15:12Z"
}
```

### 2) Verfügbarkeitsblock (Werkstatt) für **ein bestimmtes Fahrzeug** (Mehrfachinserat)
```json
{
  "id": "f2d5f2e3-1a41-4b3b-a0de-7b3d3e0aa111",
  "inseratId": "a1d2e3f4-5678-49ab-9cde-001122334455",
  "vehicleId": "veh_9b0b5c22-0e32-4e4a-8d11-6c9a9e2a77aa",
  "name": "Werkstatttermin",
  "content": "Ölwechsel + Inspektion",
  "startDate": "2025-09-10T06:00:00Z",
  "endDate": "2025-09-10T12:00:00Z",
  "isAvailability": true,
  "createdAt": "2025-08-28T10:20:00Z"
}
```

### 3) Ganztägige Sperre für das gesamte Inserat (Mehrfachinserat, **ohne** `vehicleId` – appseitig als globale Sperre interpretieren)
```json
{
  "id": "0f1e2d3c-4b5a-6978-9a0b-cdef01234567",
  "inseratId": "a1d2e3f4-5678-49ab-9cde-001122334455",
  "name": "Geschlossen (Feiertag)",
  "startDate": "2025-12-25T00:00:00Z",
  "endDate": "2025-12-26T00:00:00Z",
  "isAvailability": true
}
```

> **Interpretation:** Für Mehrfachinserate empfiehlt es sich, **fahrzeuggenaue** Blöcke (`vehicleId`) zu nutzen. Eine Inserat-weite Sperre (ohne `vehicleId`) sollte appseitig als globale Blockierung behandelt werden (z. B. alle Fahrzeuge gesperrt).

---

## Einbettungen (empfohlene API-Repräsentation)

Bei Darstellung eines Bookings können folgende eingebettete Objekte nützlich sein:

- `user` (Basisdaten, falls `userId` gesetzt)  
- `vehicle` (nur bei Mehrfachinseraten relevant)  
- `inserat` (kompakte Grunddaten, z. B. Titel/Kategorie)

**Beispiel (ausführlich):**
```json
{
  "id": "b6c9dc3a-6c1d-4b7a-8a1b-2f47c9d9a001",
  "inseratId": "e7a4b7a1-3a62-4a6f-9a3c-1a4c1b8d6f01",
  "userId": "user_123",
  "vehicleId": null,
  "name": "Max Mustermann",
  "buchungsnummer": "UR-2025-000123",
  "startDate": "2025-09-06T08:00:00Z",
  "endDate": "2025-09-07T08:00:00Z",
  "isAvailability": false,
  "createdAt": "2025-08-28T10:15:12Z",

  "user": {
    "id": "user_123",
    "displayname": "MaxM",
    "email": "max@example.com"
  },
  "inserat": {
    "id": "e7a4b7a1-3a62-4a6f-9a3c-1a4c1b8d6f01",
    "title": "Mercedes Sprinter 316",
    "category": "TRANSPORT"
  }
}
```

---

## Empfohlene Indizes & Constraints (optional)

- Indizes:  
  - `(inseratId, startDate)`  
  - `(vehicleId, startDate)` (für Mehrfachinserate)  
- **Konfliktprävention (PostgreSQL):**  
  - Exclusion Constraint auf `tstzrange(startDate, endDate, '[]')` **per (inseratId oder vehicleId)**, um Überlappungen zu verhindern.  
  - Partiell pro `isAvailability` möglich, falls abweichende Regeln gelten.

---

## Zusammenfassung

- **Booking** deckt sowohl **Kundenbuchungen** als auch **interne Availability‑Blocks** ab (`isAvailability`).  
- **vehicleId** ist **nur** bei Mehrfachinseraten relevant und ermöglicht **fahrzeuggenaue** Belegung.  
- Zeitintervalle werden als ISO‑Zeitstempel gespeichert; empfohlene Auswertung: `start` inkl., `end` exkl.  
- Konfliktlogik erfolgt je nach Szenario **pro Fahrzeug** oder **pro Inserat**.
