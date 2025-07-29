# CLAUDE.md  
**Kontext‑Datei für Claude Code (claude.ai/code)** – bitte alle festen Fakten einhalten, keine Halluzinationen.

---

## Projekt‑Überblick
AI Cold Outreach Proposal Dashboard  
Front‑End = reine **statische HTML‑Seiten** mit Vanilla JS (kein Build‑Step).  
Backend = **n8n Webhooks + Airtable + Netlify Identity (JWT)** – steht bereits.

---

## Fixe Backend‑Endpunkte (alle aktiv)

| Zweck | URL | Methode | Body / Query | Auth |
|-------|-----|---------|--------------|------|
| **Clients‑Liste** | `https://optionai.optionai.at/webhook/admin/load-clients` | GET | – | Bearer JWT (Rolle admin) |
| **Einzel‑Client + Template** | `/admin/load-client` | GET `?code=PROP‑…` | – | JWT admin |
| **Template speichern** | `/admin/save-template` | POST | `{template_id, html}` | JWT admin |
| **Proposal speichern** | `/admin/save-proposal` | POST | `{proposal_code, custom_html?, status?}` | JWT admin |

> JWT wird im Front‑End via `netlifyIdentity.currentUser().token.access_token` gesetzt.

---

## Aktuelle Dateistruktur (Front‑End)

| Datei | Zweck |
|-------|-------|
| **/index.html** | Kunden‑Dashboard + Code‑Eingabe (liefert Proposal) |
| **/login.html** | Admin‑Login (Netlify Identity Modal) |
| **/admin/index.html** | Admin‑Übersicht (Tabelle aller Clients) |
| **/admin/client.html** | Admin‑Detail: Template‑Editor + Custom‑Editor + Status |

*(Alle Files sind eigenständige Single‑File‑Dokus; CSS & JS inline.)*

---

## UX‑Flows

1. **Kunde**  
   Code‑Eingabe → `load-client` →  
   - `status != approved` ⇒ Nachricht “Proposal in Bearbeitung”  
   - `status = approved` ⇒ Template‑HTML (+ Custom‑HTML) rendern

2. **Admin**  
   `/login.html` → Netlify‑Modal → Rolle admin → `/admin/`  
   - **/admin/index.html** → `load-clients` → Tabelle & Links  
   - **/admin/client.html** → `load-client` →  
     - Editor 1: Template‑HTML → `save-template`  
     - Editor 2: Custom‑HTML → `save-proposal`  
     - Status‑Dropdown (draft / pending / approved / signed) → `save-proposal`

---

## Airtable‑Schema (= Quelle aller Daten)

### Templates (Table)
| Feld | Typ | Pflicht |
|------|-----|---------|
| `Name` | Text | ✔ |
| `HTML` | Long text (Rich off) | ✔ |
| `template_id` | Autonumber / Record‑ID | ✔ |

### Clients (Table)
| Feld | Typ | Pflicht |
|------|-----|---------|
| `Proposal Code` | Text (PK) | ✔ |
| `Full Name` | Formula | ✔ |
| `Company`, `Price`, … | diverse | ✔ |
| `Template` | Link → Templates | ✔ |
| `status` | Single select (`draft` / `pending` / `approved` / `signed`) | ✔ |
| `custom_html` | Long text (Rich off) | optional |

---

## Front‑End‑Leitsätze

- **Keine Build‑Tools** → direktes HTML/JS/CSS.
- **Fetch API** mit async/await & try/catch.
- **Tailwind CDN** darf für Styles genutzt werden (optional).
- **Ace Editor CDN** oder **CodeMirror CDN** für Admin‑Editoren.
- **Keine neuen Seiten** ohne Absprache.

---

## Offene Tasks (Stand Juli 2025)

1. **login.html** – Grundgerüst steht, evtl. Style.
2. **admin/index.html** – Client‑Tabelle rendern (Code, Name, Status, Link).
3. **admin/client.html** –  
   - Zwei Editoren & Status‑Select  
   - Buttons “Speichern” → passende Webhooks
4. **index.html** – Render‑Logik + Platzhalter‑Replace + Status‑Gate.
5. **(später)** eIDAS‑Signatur via DocuSign o. ä. – noch nicht implementiert.

---

## Konventions‑Reminder für Claude

- **Halte dich strikt an die URLs & Feldnamen** in diesem Dokument.  
- **Keinen neuen Build‑Stack** einführen.  
- **Ändere nur die Datei, die im Prompt genannt ist.**  
- **Wenn unsicher → erst Strukturvorschlag als Kommentar posten.**

---
