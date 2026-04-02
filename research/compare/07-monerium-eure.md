# Križna provjera: Monerium EURe na Gnosis lancu

**Izvještaji:**
- [07a — Claude Opus 4.6](../reports/07a-monerium-eure-claude.md)
- [07b — Perplexity](../reports/07b-monerium-eure-perplexity.md)

**Prompt:** [07-monerium-eure-gnosis.md](../prompts/07-monerium-eure-gnosis.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 92/100**

Izuzetno visoka podudarnost. Oba izvora dolaze do gotovo identičnih tehničkih i regulatornih zaključaka. Razlike su u kvantitativnim detaljima (supply brojevi, volumen) — ali oba koriste iste izvore.

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| Adrese ugovora (V1 + V2) | 100/100 | Identične adrese u oba izvora. |
| ERC-20 kompatibilnost + 18 decimala | 100/100 | Oba potvrđuju. |
| ERC-2612 Permit podrška | 100/100 | Oba navode. |
| Blacklist/freeze mehanizam | 100/100 | Identičan opis: BlacklistValidatorUpgradeable, admin uloga, blokira slanje i primanje. |
| LCX hack kao presedan zamrzavanja | 100/100 | Oba navode isti incident (611.000 EURe, ista adresa). |
| Tri uloge (Owner/Admin/System) | 100/100 | Identičan opis. |
| UUPS proxy nadogradivost | 100/100 | Oba navode. |
| Monerium EMI licenca (Centralna banka Islanda) | 100/100 | Identično. |
| SEPA ↔ EURe tok (Web3 IBAN) | 95/100 | Oba detaljno opisuju. |
| Bez naknada za SEPA konverziju | 95/100 | Oba navode fee schedule = 0 EUR. |
| Safe integracija (Safe App + Core SDK) | 95/100 | Oba opisuju. Claude dodaje Gnosis Pay detalje. |
| SDK (@monerium/sdk) i sandbox | 95/100 | Oba navode npm paket, sandbox na monerium.dev. |
| MiCA članci za EMT | 85/100 | Claude navodi čl. 48-58 (Glava IV). Perplexity navodi čl. 102, 105 (Glava VII — nadzorne ovlasti). Komplementarni, ne proturječni. |
| Supply na Gnosisu | 65/100 | Claude: ~18,3M EURe. Perplexity: ~6M EURe. Različiti datumi ili različita tumačenja V1/V2. |
| Alternative (EURS, agEUR, USDC, DAI) | 85/100 | Oba pokrivaju iste alternative. Perplexity detaljnija tablica. |

---

## 2. Razlika u supply podacima

| Izvor | EURe supply na Gnosisu | Izvor podatka |
|-------|----------------------|---------------|
| Claude | ~18,3M | GnosisScan V2 tracker |
| Perplexity | ~6M | GnosisScan V1 tracker |

**Objašnjenje:** V1 i V2 token trackeri na GnosisScan-u mogu pokazivati različite brojke ovisno o tome kako se V1→V2 migracija reflektira. Claude koristi V2 adresu (noviju), Perplexity V1. **Preporučamo verificirati na V2 adresi** (`0x420CA...`).

---

## 3. Činjenice na kojima se oba izvora slažu (visoko povjerenje)

| # | Činjenica | Povjerenje |
|---|-----------|------------|
| F1 | V1: `0xcB444e90D8198415266c6a2724b7900fb12FC56E`, V2: `0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430` | Vrlo visoko |
| F2 | ERC-20 s 18 decimala + ERC-2612 Permit + ERC-677 transferAndCall | Vrlo visoko |
| F3 | BlacklistValidatorUpgradeable — zajednički za sve Monerium tokene na istom lancu | Vrlo visoko |
| F4 | Admin uloga može blacklistati adrese (blokirano slanje I primanje) | Vrlo visoko |
| F5 | Owner je Gnosis Safe multisig | Vrlo visoko |
| F6 | System uloge za mint/burn | Vrlo visoko |
| F7 | Burn zahtijeva potpis vlasnika tokena (ne može Monerium bez korisnikove suglasnosti) | Visoko |
| F8 | `recover()` funkcija za premještanje s izgubljenog novčanika (uz korisnički potpis) | Visoko |
| F9 | Pausable mehanizam — Owner može globalno pauzirati transfere | Visoko |
| F10 | UUPS proxy (OpenZeppelin), Owner kontrolira nadogradnje | Vrlo visoko |
| F11 | V2 migracija smanjila gas za transfer/approve >60% | Visoko |
| F12 | Monerium EMI ehf. — islandska EMI licenca, passportirana na EEA | Vrlo visoko |
| F13 | LCX hack 2022: 611.000 EURe zamrznuto na Ethereumu | Vrlo visoko |
| F14 | Zamrzavanje u dva slučaja: sigurnosna prijetnja ILI zakonski/sudski nalog | Vrlo visoko |
| F15 | Nema javne liste zamrznutih adresa — treba on-chain analitika BlacklistAdded eventova | Visoko |
| F16 | SEPA ↔ EURe: osobni IBAN → automatski mint na wallet adresi (i obrnuto) | Vrlo visoko |
| F17 | Trenutno bez naknada (SEPA, mint/burn, API, 1 IBAN po korisniku) | Visoko |
| F18 | @monerium/sdk na npm-u, OAuth2, sandbox na api.monerium.dev | Vrlo visoko |
| F19 | Safe App integracija na Gnosisu (chain ID 100) | Vrlo visoko |
| F20 | Gnosis Pay koristi EURe + Safe + Monerium IBAN za Visa karticu | Vrlo visoko |
| F21 | approve() + transferFrom() radi standardno — kompatibilno s razdjelnicima | Vrlo visoko |
| F22 | EURS na Gnosisu — bridgano, mala likvidnost | Visoko |
| F23 | agEUR/EURA — decentraliziran, bez SEPA, ograničena likvidnost na Gnosisu | Visoko |
| F24 | USDC na Gnosisu — visoka likvidnost ali freeze rizik identičan EURe | Visoko |
| F25 | DAI/xDAI — nema centralni freeze, ali nije EUR denominiran | Visoko |

---

## 4. Činjenice koje je pronašao samo jedan izvor

### Samo Claude

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| C1 | Implementacija adresa: `0x60cb9fdd...` ("GnosisControllerToken") | Srednje-visoko | Specifičan detalj za devove. |
| C2 | Aktivni system račun za mint: `0x882145B1...` | Srednje-visoko | Korisno za praćenje mint aktivnosti. |
| C3 | 30.487 holdera na V2 | Srednje | Ovisi o datumu provjere. |
| C4 | sDAI/EURE par na Oku Trade ~805K USD dnevno | Srednje | Specifičan DEX podatak. |
| C5 | Audit od Ackee Blockchain Security | Srednje-visoko | Važno za sigurnosnu procjenu. |
| C6 | Gnosis Pay case study detalji (102% kolateralizacija) | Visoko | Battle-tested use case. |

### Samo Perplexity

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| P1 | MiCA čl. 102 (Precautionary measures) i čl. 105 (Product intervention) | Visoko | Konkretne regulatorne ovlasti za freeze. |
| P2 | Prosječno B2B integracijsko vrijeme ~4 tjedna | Srednje | Iz LinkedIn materijala. |
| P3 | LlamaRisk/Aave analiza: ne preporučuje EURe za Aave Linea zbog likvidnosti | Visoko | Relevantan signal o likvidnosnom riziku. |
| P4 | Detaljna tablica alternativa s freeze sposobnostima | — | Korisna referenca. |
| P5 | cEUR (Celo Euro Wormhole) na Gnosisu — zanemariva količina | Srednje | Kompletira sliku alternativa. |

---

## 5. Zaključci za projekt

### Potvrđeno (možemo se osloniti)

- **EURe je optimalan izbor** za EUR-denominiranu platformu na Gnosisu — jedini s MiCA usklađenošću, SEPA rampom i Safe integracijom.
- **approve() + transferFrom() radi** — 0xSplits V2 PushSplit kompatibilan s EURe.
- **ERC-2612 Permit** — korisnik može odobriti razdjelnik bez zasebne approve transakcije (gasless).
- **Sandbox postoji** (api.monerium.dev) — možemo razvijati bez pravih sredstava.
- **Bez naknada** trenutno — ali Monerium zadržava pravo uvesti ih.
- **Gnosis Pay** je battle-tested dokaz da EURe + Safe + Monerium radi u produkciji.

### Potvrđeni rizici

| Rizik | Ozbiljnost | Ublažavanje |
|-------|-----------|-------------|
| Blacklist zamrzava razdjelnik ili treasury | Visok | Pull model (0xSplits Warehouse fallback), segmentacija adresa, čist KYC |
| Monerium gubi licencu ili gasi uslugu | Srednji | Diverzifikacija: podržati i USDC/DAI kao alternativu |
| Globalni Pause zaustavlja SVE transfere | Srednji | Nema tehničkog ublažavanja — regulatorni rizik |
| Nadogradnja ugovora mijenja ponašanje | Nizak-srednji | Pratiti Monerium governance, alert na proxy upgradove |
| recover() — Monerium može premjestiti sredstva uz korisnički potpis | Nizak | Potpis dan pri onboardingu — ali korisnici moraju biti informirani |

### Utjecaj na arhitekturu

| Nalaz | Utjecaj | Dokument za ažuriranje |
|-------|---------|----------------------|
| EURe + 0xSplits V2 kompatibilni | Potvrđuje ADR-001 odluku i Q6 rješenje | architecture.md — zatvoriti Q6 |
| ERC-2612 Permit | UX poboljšanje: jedna transakcija umjesto approve + split | architecture.md — UX tok |
| Blacklist blokira i from i to | 0xSplits PushSplit Warehouse fallback je OBAVEZAN, ne opcija | architecture.md, model-prijetnji.md T13 |
| Pausable mehanizam | Novi T17? Globalni pause zaustavlja cijelu platformu | model-prijetnji.md |
| recover() funkcija | Novi aspekt za model prijetnji — centralizirana intervencija | model-prijetnji.md |
| Bez naknada za SEPA | Ekonomski model potvrđen: 0% overhead za on/off ramp | ekonomski-model.md |
| LlamaRisk upozorenje o likvidnosti | Za velike volumene, EURe likvidnost na Gnosisu može biti ograničavajuća | ekonomski-model.md — faza 2+ razmatranje |

---

## 6. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Duboko tehničko istraživanje (adrese implementacija, system računi, audit firma) | Regulatorno-tehničko s fokusom na rizike i alternative |
| **Prednost** | Specifičniji za pametni ugovor integraciju | Bolja regulatorna analiza (MiCA članci), detaljnija tablica alternativa |
| **Slabost** | Supply podatak možda zastarjeo/krivo tumačen | Manje implementacijskih detalja |
| **Pouzdanost** | Visoka | Visoka |
