# Križna provjera: eIDAS 2.0 i EUDI novčanik

**Izvještaji:**
- [05a — Claude Opus 4.6](../reports/05a-eidas2-eudi-claude.md)
- [05b — Perplexity](../reports/05b-eidas2-eudi-perplexity.md)

**Prompt:** [05-eidas2-eudi.md](../prompts/05-eidas2-eudi.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 85/100**

Visoka podudarnost na svim ključnim pitanjima. Oba izvora detaljni i kvalitetni. Razlike su u dubini pojedinih aspekata i jednom verzijskom proturječju (ARF verzija).

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| Regulatorni vremenski okvir | 95/100 | Identični datumi: 20.5.2024 stupanje na snagu, kraj 2026 novčanik, kraj 2027 obvezno prihvaćanje. |
| Formati akreditiva (SD-JWT-VC + mdoc) | 95/100 | Potpuno se slažu. Oba navode dualni format. |
| Kriptografski algoritmi (P-256/ES256 primarni) | 90/100 | Oba navode P-256 kao primarni. Perplexity dodaje RSA-PSS i EdDSA kao moguće. Claude specifičniji o WSCD. |
| Selektivno otkrivanje | 95/100 | Identičan opis mehanizma za oba formata. |
| ZK dokazi — planirani ali ne za lansiranje | 90/100 | Oba se slažu. Claude dodaje BBS+/BBS# detalje. |
| Hrvatska — status implementacije | 80/100 | Oba: kasni ali napreduje, AKD razvija, Q4 2026. Claude ima Ljubi citat. Perplexity ima Identyum u EWC. |
| Certilia koegzistencija s EUDI | 95/100 | Oba: koegzistiraju 3-5 godina, OIDC→OpenID4VP tranzicija. |
| Blockchain/DLT u ARF-u | 95/100 | Oba: nije u ARF-u. DC4EU eksperimentira s EBSI-jem. |
| Nullifier / globalni identifikator | 90/100 | Oba: ne postoji u EUDI. Perplexity detaljniji o per-RP pseudonimima. |
| ARF verzija | 50/100 | **Proturječje.** Claude: v2.8.0 (velj. 2026). Perplexity: v1.2.0 (stud. 2023). |
| W3C VCDM status | 90/100 | Oba: nije mandatirano. Claude eksplicitniji (ARF kaže "rizici za interoperabilnost"). |
| Pilot projekti (LSP) | 75/100 | Perplexity detaljniji (EWC, POTENTIAL, NOBID, DC4EU s članicama). Claude površniji. |

---

## 2. Proturječje: ARF verzija

| Izvor | ARF verzija | Datum |
|-------|-------------|-------|
| Claude | v2.8.0 | 2. veljače 2026. |
| Perplexity | v1.2.0 | studeni 2023. |

**Analiza:** Perplexity citira v1.2.0 kao "ključnu referentnu točku Toolboxa" i napominje da postoje radne verzije (1.3.x, 1.4.x). Claude tvrdi da postoji v2.8.0 — ovo je moguće ako se verzioniranje promijenilo ili ako je Claude fabricirao verziju. **Potrebna verifikacija na eudi.dev ili eu-digital-identity-wallet GitHub repozitoriju.**

---

## 3. Činjenice na kojima se oba izvora slažu (visoko povjerenje)

| # | Činjenica | Povjerenje |
|---|-----------|------------|
| F1 | Uredba (EU) 2024/1183 stupila na snagu 20. svibnja 2024. | Vrlo visoko |
| F2 | Rok za EUDI novčanik: ~kraj 2026. (24 mjeseca od provedbenih akata) | Vrlo visoko |
| F3 | Obvezno prihvaćanje javnog sektora: ~kraj 2027. (36 mjeseci) | Vrlo visoko |
| F4 | Dva formata: SD-JWT-VC (online) + ISO/IEC 18013-5 mdoc (blizinsko) | Vrlo visoko |
| F5 | Primarni algoritam: ECDSA P-256 (ES256) | Vrlo visoko |
| F6 | Selektivno otkrivanje je obvezno za PID | Vrlo visoko |
| F7 | ZK dokazi predviđeni regulativom (Recital 14/29) ali ne za lansiranje | Visoko |
| F8 | W3C VCDM NIJE mandatiran | Visoko |
| F9 | OpenID4VP/SIOPv2 je prezentacijski protokol (zamjenjuje klasični OIDC) | Vrlo visoko |
| F10 | OpenID4VCI je izdavački protokol | Vrlo visoko |
| F11 | ARF ne spominje blockchain/DLT | Vrlo visoko |
| F12 | Nema globalnog jedinstvenog identifikatora / nullifiera u EUDI | Vrlo visoko |
| F13 | Certilia i EUDI koegzistiraju — postojeće eID sheme nisu ukinute | Vrlo visoko |
| F14 | Hrvatska kasni ali planira Q4 2026. za certificirani novčanik | Visoko |
| F15 | AKD razvija EUDI novčanik (na temelju Certilije) | Visoko |
| F16 | Ministarstvo digitalne transformacije je nadležno tijelo | Visoko |
| F17 | PID atributi: family_name, given_name, birth_date (obvezni) | Vrlo visoko |
| F18 | Privatni ključevi moraju biti u WSCD-u (hardverski sigurni element) | Visoko |
| F19 | DC4EU eksperimentira s EBSI blockchain-om | Visoko |
| F20 | 5 provedbenih uredbi usvojeno 28.11.2024. | Visoko |

---

## 4. Činjenice koje je pronašao samo jedan izvor

### Samo Claude

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| C1 | ARF v2.8.0 (veljača 2026.) | Nisko-srednje | Nije potvrđeno od Perplexity. Moguće fabricirano. |
| C2 | BBS+ / BBS# multi-message potpisi kao ZK pristup u ARF diskusiji | Srednje | Specifičan detalj, vjerojatno iz Discussion Topica. |
| C3 | 350.000+ Certilia korisnika, 100.000 dnevnih interakcija | Srednje-visoko | Vjerojatno iz AKD prezentacije. |
| C4 | Igor Ljubi citat o zabrinutosti oko certifikacije | Srednje | Specifičan navod s ENISA foruma. |
| C5 | ~330K gas za P-256 on-chain verifikaciju (daimo-eth/p256-verifier) | Visoko | Konzistentno s istraživanjem 02. |
| C6 | Batch issuance za mitigaciju linkabilnosti | Srednje-visoko | Specifičan ARF mehanizam. |
| C7 | EU DTC pilot s Finskom na Zračnoj luci Zagreb | Srednje | Specifičan detalj. |

### Samo Perplexity

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| P1 | Identyum je jedini hrvatski partner u EWC konzorciju | Srednje-visoko | Specifičan nalaz iz EWC dokumentacije. |
| P2 | POTENTIAL konzorcij: 19 država — Hrvatska NIJE uključena | Visoko | Važno za razumijevanje HR pozicije. |
| P3 | Monerium fund recovery u EUDI kontekstu | Srednje | Zanimljiva paralela, ali tangencijalna. |
| P4 | Per-RP pseudonimi kao zamjena za globalni ID | Visoko | Ključno za Sybil zaštitu dizajn. |
| P5 | ARF 1.2.0 uvodi trust model poglavlje i PID/mDL Rulebook anekse | Visoko | Strukturalni detalj ARF-a. |
| P6 | Detaljne preporuke za dizajn sustava (8 odjeljaka) | — | Strateški, ne činjenični. |

---

## 5. Zaključci za projekt

### Potvrđeno (možemo se osloniti)

- **P-256 (ES256) je primarni algoritam EUDI-ja** — naš `IIdentityVerifier` apstrakcijski sloj je ispravno dizajniran za ovo.
- **OpenID4VP zamjenjuje klasični OIDC** — tok se značajno mijenja, ali apstrakcijski sloj to pokriva.
- **Nema EUDI nullifiera** — moramo sami dizajnirati Sybil zaštitu (potvrduje privatnost.md pristup).
- **Certilia i EUDI koegzistiraju** — dual-path dizajn iz architecture.md Dodatka je ispravan.
- **Rok kraj 2026.** — imamo ~8 mjeseci prije nego EUDI postane relevantan.
- **SD-JWT-VC** je format za online prezentaciju — naš budući EUDIVerifier mora parsirati SD-JWT, ne klasični JWT.

### Zahtijeva verifikaciju

| Prioritet | Pitanje | Kako verificirati |
|-----------|---------|-----------------|
| SREDNJE | ARF verzija — v2.8.0 ili v1.2.0? | Provjeriti eu-digital-identity-wallet GitHub i eudi.dev |
| NISKO | Je li Identyum jedini HR partner u EWC? | Provjeriti EWC službenu listu partnera |

### Utjecaj na arhitekturu

| Nalaz | Utjecaj | Dokument za ažuriranje |
|-------|---------|----------------------|
| P-256 je EUDI primarni algoritam | IIdentityVerifier v3 (EUDI) će koristiti P-256 — isti kao Certilia (ako je ES256). Konzistentno. | architecture.md — potvrda dizajna |
| SD-JWT-VC format (ne klasični JWT) | EUDIVerifier mora parsirati SD-JWT s disclosures, ne standardni JWT. Različit parser. | architecture.md — razlika v1→v3 |
| Nema EUDI nullifiera | Moramo sami implementirati per-dApp pseudonime za Sybil zaštitu | privatnost.md — potvrda Opcije B |
| OpenID4VP umjesto OIDC | Korak 2-7 u ADR-002 se mijenja za EUDI. Ali apstrakcijski sloj to pokriva. | architecture.md — potvrda IIdentityVerifier |
| Per-RP pseudonimi (Perplexity) | `hash(PID, dapp_salt)` pristup iz privatnost.md je usklađen s EUDI filozofijom | privatnost.md — validacija |
| Certilia 350K+ korisnika | Dovoljna korisnička baza za PoC | putokaz.md — pozitivna indikacija |
| Hrvatska kasni ali razvija | Rizik: EUDI možda neće biti spreman za Q4 2026. Certilia ostaje fallback. | putokaz.md — faza 3 rizik |

---

## 6. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Sintetički — integrira tehničke detalje s praktičnim implikacijama | Dokumentarni — citira sve izvore, detaljne tablice |
| **Prednost** | Specifičniji tehniški detalji (BBS+, batch issuance, gas procjene), kontekst za blockchain | Potpuniji pregled pilot projekata, bolja pokrivenost LSP-ova, detaljnije preporuke za dizajn |
| **Slabost** | ARF verzija možda fabricirana (v2.8.0 nepotvrđen) | Manje tehničkih detalja o blockchain integraciji |
| **Pouzdanost** | Visoka za tehničke zaključke, srednja za verzije/datume | Visoka za regulatorne činjenice i izvore |

**Oba izvještaja su izuzetno kvalitetni za ovu temu.** Perplexity je bolji za regulatorni kontekst, Claude za tehničke implikacije.
