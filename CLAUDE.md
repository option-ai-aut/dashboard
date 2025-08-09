# CLAUDE.md  
**Kontext‑Datei für Claude Code (claude.ai/code)** – bitte alle festen Fakten einhalten, keine Halluzinationen.

---

## Projekt‑Überblick
AI Cold Outreach Proposal Dashboard  
Front‑End = reine **statische HTML‑Seiten** mit Vanilla JS (kein Build‑Step).  
Backend = **n8n Webhooks + Airtable + Dual Auth-System** – steht bereits.

---

## Komplette Backend‑Endpunkte (alle aktiv)

| Zweck | URL | Methode | Body / Query | Auth | Verwendet von |
|-------|-----|---------|--------------|------|---------------|
| **Clients‑Liste** | `https://optionai.optionai.at/webhook/admin/load-clients` | GET | – | sessionStorage | admin/index.html |
| **Einzel‑Client + Template** | `https://optionai.optionai.at/webhook/admin/load-client` | GET | `?proposal_code=PROP‑…` | sessionStorage/none | admin/client.html, index.html |
| **Template speichern** | `https://optionai.optionai.at/webhook/admin/save-template` | POST | `{template_id, html}` | sessionStorage | admin/client.html |
| **Proposal speichern** | `https://optionai.optionai.at/webhook/admin/save-proposal` | POST | `{proposal_code, custom_html?, status?}` | sessionStorage | admin/client.html |
| **Signatur speichern** | `https://optionai.optionai.at/webhook/save-sign` | POST | `{Full Name, Email Address, record_id, proposal_code, auth_method, signed_pdf}` | none | index.html |
| **Proposal View Tracking** | `https://optionai.optionai.at/webhook/track-proposal` | POST | `{proposal_code, action, timestamp, user_agent, client_name}` | none | index.html |
| **Kunden Passwort setzen** | `https://optionai.optionai.at/webhook/set-password` | POST | `{proposal_code, password}` | none | client-login.html |
| **Kunden Login validieren** | `https://optionai.optionai.at/webhook/validate-password` | POST | `{Proposal Code, password}` | none | client-login.html, dashboard/settings.html |

> **Auth-Hinweis**: Aktuelle Implementation nutzt nur `sessionStorage.adminAuthenticated` - **kein JWT** implementiert.

---

## Vollständige Dateistruktur (Front‑End)

| Datei | Zweck | Zeilen | Features |
|-------|-------|--------|----------|
| **/index.html** | Proposal-Viewer + Code‑Eingabe + Signatur‑System | 2800+ | PDF-Gen, 3 Signatur-Methoden, Status-Gate |
| **/login.html** | Admin‑Login (Netlify Identity Modal) | 395 | JWT-basiert, Rollen-Routing |
| **/client-login.html** | **NEU:** Kunden-Login + Sign-Up | 420 | Apple Design, Modal Sign-Up |
| **/dashboard/index.html** | **NEU:** Kunden-Dashboard | 450 | Apple Design, Statistiken, Aktionen |
| **/dashboard/profile.html** | **NEU:** Kunden-Profil bearbeiten | 400 | Formular, Daten-Updates |
| **/dashboard/settings.html** | **NEU:** Kunden-Einstellungen | 380 | Passwort ändern, Account-Info |
| **/admin/login.html** | Passwort‑Login für Admin‑Bereich | 420 | Hardcoded PW: `sand-stone-austria-40` |
| **/admin/index.html** | Admin‑Übersicht (Tabelle aller Clients) | 460 | sessionStorage-Auth, Client-Liste |
| **/admin/client.html** | Admin‑Detail: Template‑Editor + Custom‑Editor + Status | 440 | TinyMCE, Status-Management |

*(Alle Files sind eigenständige Single‑File‑Dokus; CSS & JS inline.)*

---

## Dual Authentication System

### 1. Netlify Identity (login.html) - **DERZEIT NICHT GENUTZT**
```javascript
// JWT-basiert, Rollen: 'admin', 'customer'
netlifyIdentity.currentUser().token.access_token
// API-URL: https://option-ai-at.netlify.app/.netlify/identity
```

### 2. Passwort-System (admin/login.html) - **AKTIV**
```javascript
// Hardcoded Password: 'sand-stone-austria-40'
// Session: sessionStorage.setItem('adminAuthenticated', 'true')
// Auth-Check in admin/index.html & admin/client.html
```

---

## UX‑Flows (Aktuell)

### 1. **Kunde (Öffentlich)**  
   `index.html` → Code‑Eingabe → `load-client` →  
   - `status = 'draft'` ⇒ "Proposal wird bearbeitet …"  
   - `status = 'approved'` ⇒ Template‑HTML + Custom‑HTML rendern → **Signatur‑System**

### 2. **Admin (Passwort-geschützt)**  
   `/admin/` → **Redirect** → `/admin/login.html` → Passwort `sand-stone-austria-40` → `/admin/index.html`  
   - **/admin/index.html** → `load-clients` → Client-Tabelle mit Links  
   - **/admin/client.html** → `load-client` →  
     - TinyMCE Editor 1: Template‑HTML → `save-template`  
     - TinyMCE Editor 2: Custom‑HTML → `save-proposal`  
     - Status‑Dropdown → `save-proposal`

### 3. **Kunden-Dashboard (Passwort-basiert)**
   `/client-login.html` → Login/Sign-Up Modal → `/dashboard/` → Apple-Style Dashboard

### 4. **Netlify Identity (Inaktiv)**
   `/login.html` → Netlify‑Modal → **NICHT VERWENDET** in aktueller Implementation

---

## Signatur-System (Komplett implementiert)

### Signatur-Methoden
| Methode | Implementation | Output |
|---------|---------------|--------|
| **Zeichnen** | SignaturePad Canvas | PNG DataURL |
| **Upload** | File Input + Preview | Image DataURL |
| **ID Austria** | Mock-Button | Placeholder PNG |

### PDF-Workflow
1. **Signatur zu DOM** → `addSignatureToDocument(signatureDataUrl)`
2. **HTML zu PDF** → html2pdf.js mit A4-Settings
3. **PDF zu Base64** → FileReader conversion
4. **Webhook-Call** → `save-sign` mit komplettem PDF

### Webhook-Payload (save-sign)
```json
{
  "Full Name": "Max Mustermann",
  "Email Address": "max@example.com", 
  "record_id": "recXXXXXXXXXXXX",
  "proposal_code": "PROP-2024-001",
  "auth_method": "draw|upload|idAT",
  "signed_pdf": "base64EncodedPDFString..."
}
```

---

## Airtable‑Schema (= Quelle aller Daten)

### Templates (Table)
| Feld | Typ | Pflicht | Beschreibung |
|------|-----|---------|--------------|
| `Name` | Text | ✔ | Template-Name |
| `HTML` | Long text (Rich off) | ✔ | HTML-Content für Proposal |
| `template_id` | Autonumber / Record‑ID | ✔ | Primärschlüssel |

### Clients (Table)
| Feld | Typ | Pflicht | Beschreibung |
|------|-----|---------|--------------|
| `Proposal Code` | Text (PK) | ✔ | Format: PROP-YYYY-XXX |
| `Full Name` | Formula | ✔ | Kombiniert First + Last Name |
| `Email Address` | Email | ✔ | Für Signatur-Workflow |
| `Company` | Text | ✔ | Firmenname |
| `Price` | Number | ✔ | Proposal-Preis |
| `Template` | Link → Templates | ✔ | Verknüpfung zu Template |
| `Status Proposal` | Single select | ✔ | `draft`/`pending`/`approved`/`signed` |
| `Custom HTML` | Long text (Rich off) | optional | Client-spezifische Anpassungen |
| `record_id` | Auto | ✔ | Airtable Record-ID für Updates |

---

## Externe Dependencies & CDNs

| Datei | Bibliothek | Version | Zweck |
|-------|------------|---------|-------|
| **index.html** | `signature_pad` | 4.1.7 | Canvas-Signatur |
| **index.html** | `html2pdf.js` | 0.10.1 | PDF-Generierung |
| **admin/client.html** | `tinymce` | 5.10.7 | WYSIWYG HTML-Editor |
| **login.html** | `netlify-identity-widget.js` | v1 | Netlify Auth (inaktiv) |

---

## Request-Handling & Performance

### Timeout & Error-Handling
- **API-Timeout**: 15 Sekunden (AbortController)
- **Debouncing**: 300ms für User-Input
- **Request-Deduplication**: Flag verhindert simultane Calls
- **Status-Codes**: 404 → "Code ungültig", 500 → "Serverfehler"

### Status-Badge Mapping
```javascript
const statusMap = {
    'draft': { class: 'status-badge status-draft', text: 'Draft' },      // Grau
    'pending': { class: 'status-badge status-pending', text: 'Pending' }, // Gelb
    'approved': { class: 'status-badge status-approved', text: 'Approved' }, // Grün
    'signed': { class: 'status-badge status-signed', text: 'Signed' }     // Grün
};
```

---

## CSS-Design-System

### Farb-Palette
- **Primary**: `#66b3ff` (Gradient-Blau)
- **Header**: `linear-gradient(135deg, #000000 0%, #2a2a2a 100%)`
- **Background**: `#f0f2f5` (Light Gray)
- **Cards**: `#ffffff` mit `border-radius: 16px`

### Typography
- **Font-Stack**: `'SF Pro', -apple-system, BlinkMacSystemFont, sans-serif`
- **Font-Files**: SF-Pro-Text-{Regular,Medium,Bold,Thin}.otf

### Button-System
- `.btn-primary`: Blue Gradient mit Hover-Animation
- `.btn-secondary`: Transparent mit Border
- **Hover-Effekt**: `translateY(-2px)` + Shadow

---

## Aktuelle Tasks-Status (August 2025)

### ✅ **Abgeschlossen**
1. **admin/login.html** – Passwort-System implementiert
2. **admin/index.html** – Client-Tabelle mit Status-Badges
3. **admin/client.html** – TinyMCE-Editoren + Webhook-Integration  
4. **index.html** – Vollständiges Signatur-System + PDF-Generation
5. **Dual-Auth** – sessionStorage-basierte Admin-Protection

### 🔄 **In Verwendung - Aber veraltet**
- **login.html** (Netlify Identity) – Nicht verwendet in aktueller Architektur
- **JWT-Implementation** – Dokumentiert aber nicht implementiert

### 🎯 **Zukünftige Erweiterungen**
- eIDAS-konforme Signatur via DocuSign
- JWT-Migration für echte Admin-Security  
- Proposal-Templates mit Platzhalter-System

---

## Konventions‑Reminder für Claude

- **Halte dich strikt an die Webhook-URLs & Payload-Strukturen** in diesem Dokument
- **Admin-Auth**: Nutze `sessionStorage.adminAuthenticated`, **nicht** JWT
- **Passwort**: `sand-stone-austria-40` für `/admin/login.html`
- **Keine Build‑Tools**: Alles bleibt Single-File HTML/JS/CSS
- **Status-Values**: Nutze exakt `draft`/`pending`/`approved`/`signed`
- **Signatur-PDF**: Komplettes PDF als Base64 an `save-sign` Webhook
- **Error-Handling**: 15s Timeout + Status-Code-spezifische Meldungen

---

## API-Testing Checkliste

### Frontend → Backend Flows testen:
1. **Client-Login**: `index.html` → `load-client` → Status-Gate + Rendering  
2. **Admin-Login**: `admin/login.html` → Passwort → sessionStorage → Dashboard
3. **Client-Liste**: `admin/index.html` → `load-clients` → Tabelle rendern
4. **Client-Edit**: `admin/client.html` → `load-client` → TinyMCE → `save-template`/`save-proposal`
5. **Signatur-Flow**: `index.html` → Signieren → PDF-Gen → `save-sign`

### Webhook-Parameter validieren:
- Alle `proposal_code` als Query-Parameter URL-encoded
- Alle POST-Bodies als `application/json`
- `signed_pdf` als kompletter Base64-String ohne Prefix
- `record_id` von Airtable für Updates verwenden