# Križna provjera: Certilia Mobile.id

**Izvještaji:**
- [01a — Claude Opus 4.6](../reports/01a-certilia-mobileid-claude.md)
- [01b — Perplexity](../reports/01b-certilia-mobileid-perplexity.md)

**Prompt:** [01-certilia-mobileid.md](../prompts/01-certilia-mobileid.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 82/100**

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| Osnovni identitet pružatelja (AKD, pravni status) | 95/100 | Potpuno se slažu. Claude dodaje vlasničku strukturu i ISO certifikate. |
| OIDC / OAuth2 potvrda | 90/100 | Oba potvrđuju OAuth2/OIDC. Claude identificira WSO2 IS i konkretne URL-ove. |
| JWT sadržaj (ime, prezime, OIB) | 95/100 | Identični nalazi iz istog izvora (certilia.com/identity-api/). |
| eIDAS razina i notifikacija | 100/100 | Potpuno podudarno: visoka razina, nije notificirano prekogranično. |
| Cijena i registracija | 100/100 | Identične brojke: 0 EUR Start (100K), 66,36 EUR Business. |
| Nonce parametar podrška | 60/100 | **Ključna razlika.** Claude: "potvrđeno" (WSO2 IS default). Perplexity: "razumno očekivano, ali neslužbeno potvrđeno". |
| JWT algoritam potpisa | 45/100 | **Značajna razlika.** Claude: RS256/RSA-2048 (WSO2 IS default). Perplexity: "nepoznato iz javnih izvora". |
| Sandbox okruženje | 40/100 | **Značajna razlika.** Claude pronašao developer.test.certilia.com. Perplexity ne spominje. |
| Sub claim definicija | 70/100 | Oba kažu: nepoznato bez pristupa developer portalu. Claude spekulira malo više. |
| NIAS / dual-protocol arhitektura | 55/100 | Claude detaljno opisuje SAML 2.0 za NIAS vs OIDC za treće strane. Perplexity spominje NIAS ali bez tehničke dubine. |

---

## 2. Činjenice na kojima se oba izvora slažu (visoko povjerenje)

| # | Činjenica | Claude | Perplexity | Povjerenje |
|---|-----------|--------|------------|------------|
| F1 | AKD d.o.o. je pravni subjekt iza Certilije | Da | Da | Vrlo visoko |
| F2 | Identity API koristi OAuth2/OpenID Connect | Da | Da | Vrlo visoko |
| F3 | JWT sadrži: ime, prezime, OIB | Da | Da | Vrlo visoko |
| F4 | Developer portal: developer.certilia.com | Da | Da | Vrlo visoko |
| F5 | Cijena: Start 0 EUR (100K/mj), Business 66,36 EUR/mj | Da | Da | Vrlo visoko |
| F6 | eIDAS visoka razina za mobile.ID | Da | Da | Vrlo visoko |
| F7 | NIJE notificirano za prekograničnu eIDAS upotrebu | Da | Da | Vrlo visoko |
| F8 | Besplatna samoposlužna registracija aplikacije | Da | Da | Vrlo visoko |
| F9 | Korisnik daje GDPR privolu pri prvoj prijavi | Da | Da | Vrlo visoko |
| F10 | Jedina HR notificirana eID shema je eOI (osobna iskaznica) | Da | Da | Vrlo visoko |

---

## 3. Činjenice koje je pronašao samo jedan izvor

### Samo Claude (ne potvrđeno od Perplexity)

| # | Činjenica | Razina povjerenja | Razlog |
|---|-----------|-------------------|--------|
| C1 | Platforma je WSO2 Identity Server | Srednje-visoko | Temelji se na analizi URL parametara (tenantDomain=carbon.super, commonAuthCallerPath). Uvjerljivo, ali ne 100% sigurno. |
| C2 | IDP domena: idp.certilia.com | Srednje-visoko | Pronađeno iz Google-indeksirane stranice. Vjerodostojno. |
| C3 | Authorization endpoint: https://idp.certilia.com/oauth2/authorize | Srednje-visoko | Izvedeno iz C2. |
| C4 | Podržani scopeovi: openid, profile, offline_access, internal_login | Srednje-visoko | Pronađeno u indeksiranom URL-u. |
| C5 | Sandbox: developer.test.certilia.com | Srednje | URL pronađen, ali nije potvrđeno da je funkcionalan. |
| C6 | JWT potpis: RS256 / RSA-2048 | Srednje | Izvedeno iz WSO2 IS default konfiguracije. AKD je mogao promijeniti. |
| C7 | Nonce parametar: podržan | Srednje | WSO2 IS ga podržava, ali AKD mogao isključiti. |
| C8 | NIAS koristi SAML 2.0, Identity API koristi OIDC (dual-protocol) | Srednje-visoko | Konzistentno s poznatom NIAS arhitekturom. |
| C9 | AKD 100% u vlasništvu Vlade RH, ISO 27001, Tier III DC | Visoko | Javno dostupni podaci o AKD-u. |
| C10 | Rotacija ključeva: ručna (WSO2 IS default) | Nisko | Čista spekulacija iz WSO2 IS defaulta. |

### Samo Perplexity (ne potvrđeno od Claude)

| # | Činjenica | Razina povjerenja | Razlog |
|---|-----------|-------------------|--------|
| P1 | LinkedIn objava (Felix Magedanz) potvrđuje javni OIDC sustav | Srednje | Neslužbeni izvor, ali stručna osoba. |
| P2 | Trusted List uključenost (EU i nacionalna) | Visoko | Javno provjerljivo na EU Trusted List portalu. |
| P3 | gov.hr integracija za mobile.ID autentikaciju | Visoko | Javno dostupna informacija na gov.hr. |
| P4 | Detaljne reference na OIDC Core 1.0 specifikaciju | — | Generičko znanje, ne Certilia-specifično. |

---

## 4. Proturječja i razlike u procjeni

| # | Tema | Claude procjena | Perplexity procjena | Tko je vjerojatno u pravu? |
|---|------|-----------------|---------------------|---------------------------|
| D1 | Nonce podrška | "DA, podržan" (izvedeno iz WSO2 IS) | "Razumno očekivano, ali neslužbeno nepotvrđeno" | **Perplexity je korektniji metodološki.** Claude izvodi zaključak iz identificirane platforme, što je legitimno ali ne potvrđeno. Istina je vjerojatno da je podržan, ali moramo verificirati. |
| D2 | JWT algoritam | "RS256 / RSA-2048" (WSO2 IS default) | "Nepoznato iz javnih izvora" | **Perplexity je pošteniji.** Claude nagađa iz platforme. Točan odgovor: ne znamo sigurno bez pristupa JWKS endpointu. |
| D3 | Sandbox | "Postoji: developer.test.certilia.com" | Ne spominje | **Claude je konkretniji.** URL postoji, ali nije testirano je li funkcionalan. |
| D4 | Razina detalja | Puno specifičniji (URL-ovi, parametri) | Oprezniji, više referenci na izvore | **Komplementarni pristupi.** Claude je korisniji za inženjere, Perplexity je pouzdaniji za činjenične tvrdnje. |

---

## 5. Zaključci za projekt

### Potvrđeno (možemo se osloniti)

- Identity API postoji, koristi OIDC, besplatan je do 100K/mj.
- JWT sadrži OIB → **potvrđuje calldata privatnost problem** iz privatnost.md.
- eIDAS visoka razina, ali NIJE prekogranično → ograničava fazu 3 viziju (višedržavnost).
- Developer portal postoji za samoposlužnu registraciju.

### Zahtijeva neposrednu verifikaciju

| Prioritet | Pitanje | Kako verificirati |
|-----------|---------|-------------------|
| KRITIČNO | Podržava li Certilia custom nonce parametar? | Registrirati se na developer.certilia.com i testirati OIDC tok |
| KRITIČNO | Koji je JWT algoritam potpisa? (RS256 vs ES256) | Dohvatiti JWKS endpoint ili pročitati discovery dokument |
| KRITIČNO | Što je točno `sub` claim? | Pogledati primjer JWT-a iz sandbox okruženja |
| VISOKO | Je li developer.test.certilia.com funkcionalan sandbox? | Pokušati se registrirati |
| VISOKO | Uvjeti korištenja — zabranjuju li blockchain upotrebu JWT-a? | Pročitati uvjete na developer portalu |

### Utjecaj na arhitekturu

| Nalaz | Utjecaj | Dokument za ažuriranje |
|-------|---------|----------------------|
| OIB u JWT-u | Potvrđuje da calldata curenje otkriva OIB. Selektivno prosljeđivanje (privatnost.md Opcija A) je **obavezno** od prvog dana. | privatnost.md, architecture.md |
| RSA-2048 vjerojatno | Fokusirati istraživanje 02 na RSA biblioteke za Solidity, ne P-256. Ali ne eliminirati P-256 dok ne potvrdimo. | Prompt 02 i dalje relevantan za obje opcije. |
| Nije prekogranično | Faza 3 (višedržavna podrška) ne može se osloniti na Certiliju. EUDI postaje još važniji. | putokaz.md, pravna-analiza.md |
| WSO2 IS (ako potvrđeno) | Poznat softver → poznate ranjivosti, poznata konfiguracija. Dobar za nas. | model-prijetnji.md (dodati WSO2 specifične prijetnje?) |

---

## 6. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Istraživačko-deduktivni: pronašao indeksirane URL-ove i izveo zaključke iz WSO2 IS platforme | Dokumentarno-referentni: citira službene stranice i standarde |
| **Prednost** | Konkretniji, akcijski korisni podaci (URL-ovi, endpointi, algoritmi) | Pouzdaniji — svaka tvrdnja ima izvor, manje spekulacije |
| **Slabost** | Sklon proglašavanju "potvrđenim" onoga što je zapravo izvedeno/vjerojatno | Oprezniji do točke da propušta korisne informacije |
| **Idealan za** | Tehničke specifikacije, inženjerske odluke | Činjeničnu osnovu, pravne i regulatorne tvrdnje |

**Preporuka za buduća istraživanja:** Koristiti oba alata komplementarno. Perplexity za činjeničnu bazu, Claude za tehničku dedukciju. Križna provjera eliminira lažno samopouzdanje obaju izvora.
