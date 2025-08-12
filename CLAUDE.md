# CLAUDE.md  
**Kontext‑Datei für Claude Code (claude.ai/code)** – bitte alle festen Fakten einhalten, keine Halluzinationen.

---

## Projekt‑Überblick
AI Cold Outreach Proposal Dashboard  
Front‑End = **Single Page Application (SPA)** + statische HTML‑Seiten mit Vanilla JS (kein Build‑Step).  
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
| **Unified Settings speichern** | `https://optionai.optionai.at/webhook/save-settings` | POST | `{proposal_code, instantly_api_key?, avg_deal_value?, password?, ...}` | none | dashboard/settings.html |
| **Instantly Analytics laden** | `https://optionai.optionai.at/webhook/load-instantly-kpi` | POST | `{api_key}` | none | dashboard/index.html |
| **Customer Client Data laden** | `https://optionai.optionai.at/webhook/get-client` | GET | `?proposal_code=PROP-…` | none | client-login.html, dashboard/index.html, dashboard/settings.html, dashboard/profile.html |

> **Auth-Hinweis**: Aktuelle Implementation nutzt nur `sessionStorage.adminAuthenticated` - **kein JWT** implementiert.

---

## Vollständige Dateistruktur (Front‑End)

| Datei | Zweck | Zeilen | Features |
|-------|-------|--------|----------|
| **/index.html** | Proposal-Viewer + Code‑Eingabe + Signatur‑System | 2800+ | PDF-Gen, 3 Signatur-Methoden, Status-Gate |
| **/client-login.html** | **NEU:** Kunden-Login + Sign-Up | 717 | Apple Design, Modal Sign-Up, Pre-load client data |
| **/dashboard/index.html** | **SPA Kunden-Dashboard (HAUPTDATEI)** | 3200+ | **Single Page Application**, Analytics + Proposals + Settings, Dark Apple Design, Navigation ohne Reload |
| ~~**/dashboard/settings.html**~~ | ~~**Gelöscht**~~ | ~~870~~ | ~~**ENTFERNT** - Funktionalität komplett in SPA integriert~~ |
| ~~**/dashboard/proposals.html**~~ | ~~**Gelöscht**~~ | ~~812~~ | ~~**ENTFERNT** - Funktionalität in SPA integriert~~ |
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
   `index.html` → Code‑Eingabe + "Dashboard Login" Button → `load-client` →  
   - `status = 'draft'` ⇒ "Proposal wird bearbeitet …"  
   - `status = 'approved'` ⇒ Template‑HTML + Custom‑HTML rendern → **Signatur‑System**

### 2. **Admin (Passwort-geschützt)**  
   `/admin/` → **Redirect** → `/admin/login.html` → Passwort `sand-stone-austria-40` → `/admin/index.html`  
   - **/admin/index.html** → `load-clients` → Client-Tabelle mit Links  
   - **/admin/client.html** → `load-client` →  
     - TinyMCE Editor 1: Template‑HTML → `save-template`  
     - TinyMCE Editor 2: Custom‑HTML → `save-proposal`  
     - Status‑Dropdown → `save-proposal`

### 3. **Kunden-Dashboard (SPA System)**
   `/client-login.html` → Login/Sign-Up + Passwort vergessen → `/dashboard/` →  
   
   **🎯 Single Page Application (SPA) Features:**
   - **Nahtlose Navigation**: Analytics ↔ Proposals ↔ Settings ohne Seitenreload
   - **Konstante UI**: Sidebar + Header bleiben immer gleich, nur Content wechselt
   - **URL History**: Browser Back/Forward funktioniert korrekt
   - **Fade-in Animationen**: 0.3s smooth transitions zwischen Seiten
   
   **📊 Analytics Dashboard** (`/dashboard/`):
   - **Instantly Analytics**: KPI-Grid mit Email-Metriken (Gesendet, Öffnungen, Antworten, Leads, Umsatz)
   - **Apple Dark Design**: Konsistent mit macOS-Ästhetik
   - **Real-time Data**: Live-Updates über Instantly API
   
   **📋 Proposals Section** (`/dashboard/proposals.html` Route):
   - **KPI-Grid Layout**: Gleiche Struktur wie Analytics
   - **Vollständige Proposal-Daten**: Price → Implementierungskosten, Created/Signed dates
   - **Status mit pulsierendem Dot**: Dynamischer Status-Indikator
   - **PDF Download Button**: Nur bei vorhandener signed PDF URL
   
   **⚙️ Settings Section** (`/dashboard/settings.html` Route):
   - **Account Information**: Proposal Code, Full Name, Company, Status
   - **Quick Actions**: Erweiterte Einstellungen, Abmelden
   - **Unified Settings Integration**: Link zur Detail-Settings-Seite
   
   **🎨 UI Konsistenz:**
   - **Kollapsible Sidebar**: 84px ↔ 240px mit perfekt zentrierten SVG Icons
   - **Header mit User-Info**: Name + Proposal Code oben rechts
   - **Active Navigation**: Automatisches Highlighting der aktuellen Seite


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
| `Password` | Text | optional | Kunden-Dashboard Login |
| `Instantly API` | Text | optional | Instantly API Key für Analytics |
| `Avg Deal Value` | Number | optional | Durchschnittsumsatz pro Lead (€) |
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

### Dashboard Dark Theme (Apple-Style)
- **Background**: `#0d1117` (GitHub Dark)
- **Sidebar**: `#161b22` mit `84px ↔ 240px` Hover-Expansion
- **Cards**: `#161b22` mit `border-radius: 24px` und Padding `60px 80px`
- **Border**: `1px solid rgba(48, 54, 61, 0.5)` mit Box-Shadow
- **Text Primary**: `#e6edf3` (Hellgrau)
- **Text Secondary**: `#7d8590` (Grau)

### Public Proposal (Light Theme)
- **Primary**: `#66b3ff` (Gradient-Blau)
- **Header**: `linear-gradient(135deg, #000000 0%, #2a2a2a 100%)`
- **Background**: `#f0f2f5` (Light Gray)
- **Cards**: `#ffffff` mit `border-radius: 16px`

### Typography
- **Font-Stack**: `'SF Pro Display'/'SF Pro Text', -apple-system, BlinkMacSystemFont, sans-serif`
- **Font-Files**: SF-Pro-{Display-Bold, Text-Regular, Text-Medium, Text-Bold}.otf
- **Font-Weights**: 400 (Regular), 500 (Medium), 600 (Bold), 700 (Display)

### Icon System
- **SVG Icons**: 24px × 24px, `fill="currentColor"`
- **Sidebar Icons**: Perfekt zentriert bei 84px Breite, stationär bei Hover
- **Status Dots**: Pulsierende Animation für Proposal-Status

---

## Aktuelle Tasks-Status (August 2025)

### ✅ **Abgeschlossen (Stand August 2025)**
1. **admin/login.html** – Passwort-System implementiert
2. **admin/index.html** – Client-Tabelle mit Status-Badges
3. **admin/client.html** – TinyMCE-Editoren + Webhook-Integration  
4. **index.html** – Vollständiges Signatur-System + PDF-Generation
5. **Dual-Auth** – sessionStorage-basierte Admin-Protection
6. **client-login.html** – Apple-Style Kunden-Login + Sign-Up
7. **Single Page Application (SPA)** – Vollständig implementiert in `/dashboard/index.html`
8. **Dark Apple Design** – Konsistente macOS-Ästhetik mit dunklem Theme
9. **Kollapsible Sidebar** – 84px ↔ 240px mit perfekt zentrierten SVG Icons
10. **Multi-Page Navigation** – Analytics ↔ Proposals ↔ Settings ohne Reload
11. **URL History Management** – Browser Back/Forward Support mit pushState
12. **Proposal KPI Dashboard** – Vollständige Proposal-Daten im KPI-Grid Format
13. **Status mit Pulsing Animation** – Dynamischer Status-Dot für Proposals
14. **PDF Download Integration** – Context-aware PDF Download Button
15. **Instantly Analytics** – Live-Statistiken im Dashboard (emails_sent, reply_count, total_opportunities, potenzieller Umsatz)
16. **Unified Settings System** – Ein Webhook für alle Kunden-Einstellungen (`/webhook/save-settings`)
17. **API Key Toggle** – Sicheres Anzeigen/Verbergen mit 👁️ Button
18. **Analytics Protection** – Webhook-Calls nur bei vorhandenem API Key
19. **User-Info im Header** – Name + Proposal Code oben rechts statt in Sidebar
20. **SPA Konsolidierung** – Alle Funktionen von proposals.html und settings.html in index.html integriert
21. **Kritische Fehlerbehebung** – 101+ HTML/JavaScript Fehler behoben (fehlende Funktionen, DOM-Referenzen, Syntax)
22. **Icon Positioning Fix** – Sidebar Icons stationär während Hover-Transitions (absolute positioning)
23. **Complete Settings Integration** – Passwort-Änderung, API-Konfiguration, Support-Kontakt vollständig integriert
24. **Large Proposal Cards** – 4-spaltige KPI-Grid Redesign mit Apple-Style großen Cards
25. **File Cleanup** – Unnötige proposals.html und settings.html gelöscht nach vollständiger SPA-Integration
26. **Analytics Card-Header Redesign** – Icon links, Status rechts, Querstrich unter Header für Konsistenz
27. **Proposals Card Complete Redesign** – Exakt gleiche Struktur wie Analytics mit KPI-Grid, nicht klickbar
28. **Status Data Source Fix** – Nutzt korrekt `Status Proposal` Feld statt `status`
29. **PDF URL Parsing** – Korrekte Extraktion aus Array-Struktur `Signed PDF[0].url`
30. **Download Button Design** – Status-Badge Design für Download Button
31. **Settings Complete Redesign** – Apple-Style mit KPI-Grid, klickbares Passwort-Card, funktionale Formulare
32. **Unified Design Language** – Alle 3 Views (Analytics/Proposals/Settings) nutzen identische Card-Struktur

### 🔄 **Nicht mehr verwendet (gelöst/ersetzt)**
- **login.html** (Netlify Identity) – Datei entfernt, da überflüssig
- **dashboard/profile.html** – Entfernt, Funktionalität in SPA Settings integriert
- **dashboard/proposals.html** – Gelöscht, Funktionalität komplett in SPA index.html integriert
- **dashboard/settings.html** – Gelöscht, Funktionalität komplett in SPA index.html integriert
- **Separate Proposal/Settings Pages** – Ersetzt durch SPA Router System in einer Datei
- **Multi-Page Architecture** – Komplett ersetzt durch Single Page Application
- **JWT-Implementation** – Dokumentiert aber nicht implementiert

### 🎯 **Zukünftige Erweiterungen**
- eIDAS-konforme Signatur via DocuSign
- JWT-Migration für echte Admin-Security  
- Proposal-Templates mit Platzhalter-System

---

## Unified Settings System (Neu August 2025)

### **Webhook:** `/webhook/save-settings`
**Flexibles Payload-System** für alle Kunden-Einstellungen:

```json
{
  "proposal_code": "PROP-CAME-RUDM",
  "instantly_api_key": "ODYzZGViNWYtNWNkZS0...",    // optional
  "avg_deal_value": 2500.00,                        // optional  
  "password": "newPassword123",                     // optional
  // weitere Settings automatisch erweiterbar
}
```

### **Analytics Protection**
- **Dashboard Analytics** startet nur bei vorhandenem API Key
- **Verhindert** unnötige n8n Webhook-Calls
- **Smart UX**: "API Key nicht konfiguriert" Hinweis

### **API Key Management**
- **👁️ Toggle-Button**: Sicher anzeigen/verbergen
- **Persistent Loading**: Vorhandene Keys beim Laden anzeigen
- **Masked Display**: Initial als Password-Feld

## Konventions‑Reminder für Claude

### **SPA Dashboard System**
- **Navigation**: Nutze SPA Router für seamless page transitions
- **Content Loading**: Hide/Show spa-sections, nicht innerHTML überschreiben
- **CSS Konsistenz**: Alle Sections nutzen `automation-master-card` + `kpi-grid` Layout
- **Icon Positioning**: Sidebar Icons bei 22px margin-left (oben) / 30px (unten) für 84px Sidebar
- **Animations**: Fade-in für Content-Switches, Pulsing für Status-Dots

### **Data Mapping (Proposals)**
- **Price** → **Implementierungskosten** (€ Format)
- **Created** → **Erstellt am** (deutsches Datum)
- **Signed Date** → **Unterschrieben am** (deutsches Datum)
- **Status Proposal** → Pulsierender Dot + deutscher Text
- **Signed PDF** → PDF Download Button (nur wenn URL vorhanden)

### **Bestehende Konventionen**
- **Unified Settings**: Nutze `/webhook/save-settings` für alle Kunden-Einstellungen
- **Analytics Protection**: Prüfe API Key vor Webhook-Calls
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