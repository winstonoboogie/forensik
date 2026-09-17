# Lathund — Forensisk incidentrapport, från loggar till färdig rapport

En snabbreferens för hela kedjan: insamling → analys → verifiering → rapport.
Sparad tillsammans med skillen **forensic-incident-report** (innehåller
`report_template.js` och `analyze_case.py`).

---

## 0. Helhetsbilden

```
Invoke-M365IR_v2.ps1          →  IR-CASE-<namn>/ (loggmapp)
        │
        ▼
IoC-lista (du skriver)        →  iocs.csv
        │
        ▼
analyze_case.py                →  findings.json + review.md
        │                          (maskinläsbart)      (för dig/Claude att läsa)
        ▼
Du + Claude verifierar review.md mot källdata
        │
        ▼
case_data.json (Claude skriver, du godkänner)
        │
        ▼
report_template.js             →  Forensisk_Incidentrapport_*.docx
```

**Din del i praktiken varje gång:** kör PS-scriptet (steg 1), bygg en
IoC-lista (steg 2), och skicka båda till Claude. Claude sköter resten
(analys, verifiering, rapportskrivning, visuell kontroll) — men du bör
alltid läsa igenom `review.md` och slutrapporten själv innan den går
vidare, särskilt allt märkt "kräver granskning".

---

## 1. Kör insamlingsscriptet

```powershell
.\Invoke-M365IR_v2.ps1 `
  -CaseName "kortnamn" `
  -UserUPN "anvandare@domän.se" `
  -StartTimeUtc "2026-09-11T00:00:00Z" `
  -EndTimeUtc "2026-09-12T00:00:00Z"
```

Ger dig en mapp `IR-CASE-<kortnamn>/` med:

| Mapp | Innehåll |
|---|---|
| `00_METADATA/` | `case_metadata.json`, `coverage_gaps.csv` (⚠️ kolla alltid denna), `manifest.csv`, `execution_log.txt`, `completion.txt` |
| `01_RAW/` | `unified_audit_log.jsonl` (rådata — Claude läser denna vid behov av fält som saknas i den normaliserade CSV:n) |
| `02_NORMALIZED/` | `entra_signins.csv`, `unified_audit_events.csv`, `mailitems.csv`, `message_trace.csv`, `sharepoint_file_operations.csv`, `inbox_rules.csv`, `forwarding.csv`, `mailbox_permissions.csv`, `mailbox_folder_permissions.csv`, `oauth_consent_grants.csv`, `risk_detections.csv` |
| `03_ANALYSIS/` | `timeline.csv` |

**Innan du går vidare:** öppna `coverage_gaps.csv`. Är den inte tom vet du
redan vilka delar av underlaget som kan vara ofullständiga.

Zippa hela `IR-CASE-<kortnamn>/`-mappen och skicka till Claude.

---

## 2. Bygg en IoC-lista

CSV med kolumnerna `type,value,comment`:

```csv
type,value,comment
ip,43.130.111.140,Angriparens infrastruktur
ip,173.252.166.76,Webbläsaråtkomst + e-postregel
device,DESKTOP-7E28C8,Obehörigt registrerad enhet
useragent,python-requests/2.34.2,Automatiserad åtkomst
alias,anvandare@annandomän.se,E-postalias för samma brevlåda
```

| type | Använd för |
|---|---|
| `ip` | Angripar-IP:er (vanligast) |
| `device` | Kända skadliga enhetsnamn |
| `useragent` | Ovanliga/skriptade user-agent-strängar |
| `domain` | Domäner (fritt fält, ingen specifik matchning ännu) |
| **`alias`** | ⚠️ **Viktig om kontot har fler e-postadresser** (proxyadresser) utöver sin UPN. Utan detta missas mail skickat/mottaget under aliaset helt i massutskicks- och konversationskapningsanalysen. |

Enkel textfil (en indikator per rad) funkar också — typ gissas automatiskt.

---

## 3. Analys (Claude kör detta åt dig)

```bash
python3 analyze_case.py \
  --case-dir "IR-CASE-<kortnamn>" \
  --iocs iocs.csv \
  --out findings.json --review review.md
```

`review.md` ger dig:
- Coverage gaps
- Tidslinje, endast IoC-träffar, med källhänvisning (fil + radnummer)
- Ett **heuristiskt** klassificeringsförslag (aldrig en slutgiltig bedömning)
- Massutskick / konversationskapning (från `message_trace.csv`)
- Inkorgsregel-**historik** ur råloggen (fångar regler som skapats *och
  tagits bort* — syns inte i `inbox_rules.csv`, som bara visar nuvarande
  regler)
- Filtäckning (vilka källfiler som faktiskt hittades)

**Regel:** inget i `review.md` skrivs rakt in i rapporten. Allt verifieras
mot källfilen/radnumret som anges, innan det blir en formulering i
slutrapporten.

---

## 4. Rapportbygget (`case_data.json`)

Claude skriver denna åt dig utifrån de verifierade fynden. Formen:

```json
{
  "meta": { "title": "...", "user": "...", "org": "...", "..." },
  "content": [ { "t": "h1", "text": "1. Sammanfattning" }, "..." ]
}
```

### Blocktyper (för referens — du behöver sällan skriva dessa själv)

| Block | Används för |
|---|---|
| `h1` / `h2` / `h3` | Rubriker (driver innehållsförteckningen) |
| `p` | Brödtext |
| `rich` | Blandad formatering i ett stycke (t.ex. teknisk term i monospace) |
| `mono` | Tekniska rader (IP, felkoder, enhetsnamn) |
| `bullets` | Punktlista |
| `source` | Grå källhänvisning under ett avsnitt |
| `verdict` | Grön/röd/gul bock-rad (`level`: `good`/`bad`/`warn`) |
| `table` | Tabell, med `badge` (`confirmed`/`notverified`/`notfound`) per cell |
| `timelineHeader` | Mörkblått tidsstämpel-band |
| `box` | Markerad ruta (slutlig klassificering) |

---

## 5. Rendera och leverera

```bash
node report_template.js utfil.docx case_data.json
```

Claude konverterar sedan till PDF och granskar varje sida som bild innan
leverans — kontrollerar platshållartext, badge-färger, kronologi och att
källhänvisningar pekar på riktiga filnamn.

---

## 6. Kända begränsningar att ha koll på

- **`unified_audit_events.csv` kan sakna fält** (t.ex. `Parameters` för
  inkorgsregler) på grund av hur `Export-Csv` hanterar blandade
  objekttyper i PS-scriptet. Om något ser ofullständigt ut, be Claude
  kolla `01_RAW/unified_audit_log.jsonl` direkt. **Inte fixat i scriptet
  ännu** — flaggat som rekommenderad åtgärd i rapporterna tills vidare.
- **`mailitems.csv`** använder `ClientIPAddress`, inte `ClientIP` som
  övriga filer — redan hanterat i `analyze_case.py`, men värt att minnas
  om du någon gång läser filen manuellt.
- **Message trace + alias** — se `type: alias` i IoC-listan ovan.
- **`Get-MessageTraceV2`** har begränsad retention (grovt ~10 dagar) —
  för äldre incidenter kan `message_trace.csv` vara tom även om scriptet
  fungerar perfekt.
- **Enhetsregistreringar** som identifieras manuellt i Entra-portalen
  (t.ex. vid sanering) syns inte alltid i granskningsloggen — nämn det
  explicit till Claude om ni hittat enheter den vägen, så tas det med
  korrekt källhänvisat i rapporten istället för att antas komma från
  loggen.

---

## 7. Så här ber du Claude om nästa rapport

Enklast möjliga formulering, när du har allt underlag:

> "Här är loggmappen och IoC-listan för [ärende]. Kör analysen och ta
> fram rapporten enligt vår mall."

Bifoga zip-filen med `IR-CASE-...`-mappen och IoC-filen. Claude känner
igen skillen automatiskt och kör hela kedjan (steg 3–5 ovan) självmant.
