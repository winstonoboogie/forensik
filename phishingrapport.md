# Phishingrapport

## 1. Beskrivning av e-post och inhämtade artefakter

### E-postartefakter

| Artefakt | Värde |
|---|---|
| Avsändaradress | emailsecalert1@gmail.com |
| Reply-To-adress | emailsecalert1@gmail.com (samma som avsändare) |
| Ämnesrad | "Ditt e-postkonto kommer att låsas! Agera NU!" |
| Mottagare | john.smith@dicksonunited.co.uk, alice.cooper@dicksonunited.co.uk, jacon.long@dicksonunited.co.uk, fred.johnson@dicksonunited.co.uk, pickle.rick@dicksonunited.co.uk (5 mottagare) |
| Datum och tid | 2020-06-01, 15:21 |
| Avsändande server-IP | 209.85.222.173 |
| Reverse DNS | mail-qk1-f173.google.com (Gmail-infrastruktur) |
| SPF/DKIM/DMARC | Ej specificerat i underlaget – bör inhämtas från fullständiga headers om möjligt |

### Webbartefakter

| Artefakt | Värde |
|---|---|
| Fullständig URL (sanerad) | hxxps://outlook-security.emailsecalerts[.]net/index/2020/OWA.php |
| Rotdomän | emailsecalerts[.]net |
| Domänålder | ~25 dagar vid analystillfället |
| Webbplatsens syfte | Falsk Outlook Web Access-inloggningssida som samlar in e-postadress och lösenord |

### Filartefakter

Inga bifogade filer har identifierats i det aktuella meddelandet.

### Sammanfattning

Meddelandet utger sig för att vara en automatiserad säkerhetsvarning från Microsoft Outlook och skapar tidspress ("Agera NU!") för att få mottagaren att klicka på en länk och verifiera sitt konto. Avsändaren är i själva verket ett vanligt Gmail-konto, inte Microsoft. Länken leder till en typosquattad domän som imiterar en e-postsäkerhetstjänst och som är utformad för att stjäla Office 365/Outlook-inloggningsuppgifter. Detta klassas som ett **phishingangrepp (credential harvesting)** riktat mot flera användare inom organisationen dicksonunited.co.uk.

---

## 2. Analys av artefakter

| Verktyg | Analyserad artefakt | Resultat | Bedömning |
|---|---|---|---|
| Analys av e-posthuvuden | Avsändare, Reply-To, ämnesrad | Avsändaradress och Reply-To är identiska, ett privat Gmail-konto som utger sig för att representera Microsoft/Outlook | Tydligt tecken på spoofing/social engineering |
| Reverse DNS-kontroll | Server-IP 209.85.222.173 | mail-qk1-f173.google.com – bekräftar att mejlet skickats via Gmail, inte via Microsofts infrastruktur | Stärker misstanken om imitationsförsök |
| URL2PNG | hxxps://outlook-security.emailsecalerts[.]net/index/2020/OWA.php | Visuell rendering visar en falsk Outlook Web Access-inloggningssida som begär e-postadress och lösenord | Sidan är utformad för uppgiftsstöld (credential harvesting) |
| VirusTotal | Domänen emailsecalerts[.]net | Domänen är redan flaggad av flera säkerhetsleverantörer som skadlig/phishing | Bekräftar att domänen är känd skadlig |
| WHOIS-sökning | Rotdomän | Domänen registrerad ca 25 dagar tidigare, namn valt för att likna legitima e-postsäkerhetstjänster (typosquatting) | Typiskt mönster för nyregistrerad phishinginfrastruktur |
| SIEM-loggar | Nätverkstrafik från mottagarna | Ingen användare har etablerat kommunikation med den skadliga domänen | Ingen indikation på att någon klickat på länken |
| EDR-loggar | Endpoint-aktivitet hos mottagarna | Ingen avvikande aktivitet kopplad till incidenten | Stärker bilden av att inget klick skett |
| E-postgatewayloggar | Svar/vidarebefordran av meddelandet | Ingen mottagare har svarat på meddelandet, inga ytterligare mottagare identifierade utöver de fem kända | Spridningen är begränsad till de fem ursprungliga mottagarna |

### Sammanfattande bedömning

Meddelandet klassas som **Phishing**, mer specifikt ett försök till stöld av inloggningsuppgifter (credential harvesting) via en falsk Outlook-inloggningssida. Ingen indikation finns på att någon mottagare interagerat med länken, vilket tyder på att incidenten fångats upp innan skada uppstått.

---

## 3. Rekommenderade skyddsåtgärder

### E-postrelaterade åtgärder
- Blockera avsändaradress emailsecalert1@gmail.com i e-postgatewayen
- Blockera Reply-To-adressen (samma adress i detta fall)
- Skapa gatewayregel som filtrerar på ämnesraden/liknande mönster ("konto kommer att låsas", brådskande verifieringskrav)
- Rensa kvarvarande kopior av meddelandet från samtliga fem mottagares postlådor

### Webbrelaterade åtgärder
- Blockera domänen emailsecalerts[.]net samt rotdomänen i webbproxy/DNS-filter
- Blockera den specifika URL:en i URL-filtreringen
- Blockera avsändande IP 209.85.222.173 endast om det bedöms lämpligt (observera att detta är en delad Gmail-serveradress – blockering här kan påverka legitim Gmail-trafik, se konsekvensresonemang nedan)

### Filrelaterade åtgärder
Ej tillämpligt – inga filbilagor identifierade i detta ärende.

### Användarrelaterade åtgärder
- Informera de fem berörda mottagarna om det aktuella phishingförsöket
- Genomför riktad säkerhetsutbildning/awareness-insats med fokus på att känna igen falska säkerhetsvarningar och brådskande uppmaningar
- Kontrollera MFA-status för de berörda kontona, och aktivera om det saknas
- Granska inloggningshistorik för de fem kontona för säkerhets skull, trots att loggar inte visar klick

### Konsekvenser av blockering att beakta
- Att blockera hela IP-adressen 209.85.222.173 är inte lämpligt eftersom den tillhör delad Gmail-infrastruktur och skulle kunna blockera legitim e-post från andra Gmail-avsändare. Blockering bör istället ske på specifik avsändaradress och domän.
- Blockering av domänen och URL:en bedöms inte medföra några negativa konsekvenser eftersom domänen är nyregistrerad och saknar legitimt verksamhetssyfte för organisationen.

### Sammanfattning

- **Utförda åtgärder:** Analys av headers, reverse DNS, URL-rendering, VirusTotal-kontroll, WHOIS, genomgång av SIEM/EDR/gatewayloggar
- **Rekommenderade åtgärder:** Blockering av avsändaradress, domän och URL; användarinformation och awareness-utbildning; MFA-kontroll
- **Risknivå:** Låg–medel (inget bekräftat klick, men riktat mot flera användare och tekniskt trovärdigt utformat)
- **Slutlig bedömning:** Phishing / Credential harvesting-försök riktat mot organisationen dicksonunited.co.uk
