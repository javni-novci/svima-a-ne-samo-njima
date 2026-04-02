# Zapis arhitektonskih odluka i model prijetnji

**Projekt:** javni-novci / svima-a-ne-samo-njima
**Verzija:** v0.2-nacrt
**Status:** Iteracija — poziv na tehničku recenziju

---

## Sadržaj

1. [Pregled sustava](#1-pregled-sustava)
2. [ADR-001: Arhitektura trezora i tok sredstava](#2-adr-001-arhitektura-trezora-i-tok-sredstava)
3. [ADR-002: Verifikacija identiteta i OAuth2 tok](#3-adr-002-verifikacija-identiteta-i-oauth2-tok)
4. [ADR-003: Izbor mreže i korisničko iskustvo s gorivom](#4-adr-003-izbor-mreže-i-korisničko-iskustvo-s-gorivom)
5. [Analiza prijetnji (model prijetnji)](#5-analiza-prijetnji-model-prijetnji)
6. [Arhitektura pametnih ugovora](#6-arhitektura-pametnih-ugovora)
7. [Neriješena pitanja i sljedeći koraci](#7-neriješena-pitanja-i-sljedeći-koraci)

---

## 1. Pregled sustava

### Tehnološki stog

| Sloj | Tehnologija | Uloga |
|------|-------------|-------|
| Identitet | Certilia Mobile.id (AKD) | eIDAS OIDC pružatelj — pravni identitet građana |
| Blockchain | Gnosis lanac | Izvršavanje pametnih ugovora, jeftino gorivo |
| Valuta | EURe (Monerium) | MiCA usklađeni EUR stabilni token |
| Trezor | Safe (bivši Gnosis Safe) | Višepotpisno upravljanje sredstvima |
| Identitetni token | Neprenosivi token identiteta (SBT) | Neprenosiva lančana značka verificiranog građanina |
| Pozadinski sustav | Node.js | OIDC poštar — nema Web3 ključeve |
| Posrednik | Gelato / Biconomy (ERC-4337) | Kovanje SBT-a bez goriva za nove korisnike |

### Temeljno načelo

**Niti jedna komponenta ne smije imati dovoljno moći da samostalno kompromitira sustav.**

- Poslužitelj ne drži Web3 ključeve — ne može trošiti sredstva.
- Poslužitelj ne može falsificirati Certilia potpis — ne može kovati lažne identitete.
- Klijent ne može zaobići poslužitelj — ne može sam dobiti JWT bez `client_secret` stražnjeg kanala.
- Pametni ugovor ne vjeruje nikome — sam verificira JWT kriptografski.

```
+------------------+     OIDC      +--------------------+     S2S      +----------------+
|                  | -- nonce -->  |                    | ----------> |                |
|  Klijent         |  auth code    |  Node.js           |   JWT       |  Certilia AKD  |
|  (Novčanik pod   | <-- JWT ---  |  Poslužitelj        | <---------- |  (eIDAS OIDC)  |
|   vlastitom      |               |  (Poštar)          |             |                |
|   skrbi)         |               |                    |             |                |
+--------+---------+               +--------------------+             +----------------+
         |
         |  claimCertiliaSBT(jwt)
         v
+---------------------------+          +---------------------------+
|  CertiliaSBT.sol          |          |  Safe trezor              |
|  Gnosis lanac             |  ------> |  + Zodiac modul uloga     |
|  Lančana JWT verifikacija |          |  + Razdjelnik plaćanja    |
+---------------------------+          +---------------------------+
```

---

## 2. ADR-001: Arhitektura trezora i tok sredstava

### Kontekst

Podcast tržište prima uplate od korisnika u EURe. Sredstva se moraju podijeliti između kreatora (90%) i platforme (10%). Trebamo odlučiti kako strukturirati tok sredstava.

### Razmatrane opcije

#### Opcija A: Monolitni Safe trezor (ODBAČENO)

Sva sredstva (uplate korisnika, zarada kreatora, platformske naknade) ulaze u jedan Safe višepotpisni novčanik. Potpisnici odobravaju svaku isplatu.

**Prednosti:**
- Jednostavna implementacija — jedan ugovor, jedan trezor.
- Potpuna kontrola potpisnika nad svim sredstvima.

**Nedostaci:**
- **Usko grlo:** Svaka mikrotransakcija (npr. 2 EURe za epizodu podcasta) zahtijeva M-od-N potpis. Neskalabilno.
- **Meta za napadače:** Sva sredstva na jednoj adresi. Ako kompromitiramo kvorum potpisnika, gubimo SVE.
- **Računovodstvena noćna mora:** Nemoguće razlikovati korisničke depozite od platformske zarade bez izvanlanačnog knjigovodstva.
- **Trošak goriva:** Svaka Safe transakcija troši višestruko više goriva od običnog prijenosa.

#### Opcija B: Razdjelnik plaćanja / 0xSplits (ODABRANO)

Svaka uplata prolazi kroz automatski razdjelni ugovor koji u istoj transakciji usmjerava sredstva na predodređene adrese.

**Prednosti:**
- **Atomičnost:** Uplata se dijeli u jednom koraku. Nema stanja "sredstva zapela u prijenosu".
- **Nema ručne intervencije:** Kreator prima svoj udio automatski. Nema čekanja na odobrenje potpisnika.
- **Izolacija rizika:** Platformski Safe drži samo 10% prihoda. Kreatorova sredstva nikada nisu u našem trezoru.
- **Provjerljivost:** Svaka raspodjela je lančano vidljiva. Bilo tko može verificirati da kreator prima točno 90%.

**Nedostaci:**
- Kompleksnija inicijalna konfiguracija (postavljanje razdjelnika po kreatoru ili korištenje 0xSplits v2 s distribucijom na zahtjev).
- Kreator mora sam čuvati svoj novčanik (odgovornost vlastite skrbi).

#### Opcija C: Zodiac modul dopuštenja (ALTERNATIVA)

Safe sa Zodiac modulom koji dozvoljava automatske male isplate do određenog limita bez potpisnika.

**Prednosti:**
- Sva sredstva i dalje prolaze kroz Safe — jedinstven revizijski trag.
- Automatske isplate do X EURe dnevno bez kvoruma potpisnika.

**Nedostaci:**
- Sredstva i dalje pomiješana u jednom trezoru (rizik mete za napadače ostaje).
- Zodiac modul dodaje napadačku površinu na Safe.
- Limiti se moraju pažljivo kalibrirati — previše restriktivno = usko grlo, previše liberalno = rizik iscrpljivanja.

### Odluka

**Opcija B — Razdjelnik plaćanja** za korisničke uplate. Platformski Safe prima samo svoj udio (10%). Za budućnost, Zodiac modul dopuštenja može služiti za operativne troškove platforme iz Safe-a.

### Tok sredstava (dokaz koncepta)

```
Korisnik
  |
  |  Plaća 10 EURe za podcast epizodu
  v
+---------------------------+
|  Razdjelnik plaćanja      |
|  (0xSplits ili vlastiti)  |
+---------------------------+
  |                    |
  |  9 EURe            |  1 EURe
  v                    v
+---------------+    +--------------------+
|  Kreator      |    |  Platformski Safe  |
|  (Novčanik pod|    |  (Višepotpisni 2/3)|
|   vlastitom   |    |  Operativni fond   |
|   skrbi)      |    |                    |
+---------------+    +--------------------+
```

---

## 3. ADR-002: Verifikacija identiteta i OAuth2 tok

### Kontekst

Moramo povezati hrvatski pravni identitet (Certilia/eIDAS) s Ethereum adresom. Ključni problem: **Certilia koristi RSA/X.509 potpise (eIDAS standard), a EVM koristi secp256k1.** Safe ne može nativno čitati Certilia potpise.

### Razmatrane opcije

#### Opcija A: Svemoćni pozadinski sustav (ODBAČENO)

Node.js poslužitelj drži glavni Web3 privatni ključ. Kada korisnik prođe Certilia autentikaciju, poslužitelj potpisuje lančanu atestaciju i kuje SBT u ime korisnika.

**Prednosti:**
- Trivijalna implementacija — standardni Web2 auth + transakcija na strani poslužitelja.
- Korisnik ne treba razumjeti blockchain.

**Nedostaci:**
- **Katastrofalna jedna točka kvara.** Kompromitiran poslužitelj = napadač može:
  - Kovati neograničen broj lažnih SBT-ova (lažni identiteti).
  - Trošiti sredstva iz trezora ako je ključ poslužitelja potpisnik na Safe-u.
  - Opozvati legitimne SBT-ove.
- Proturječje cijeloj filozofiji sustava bez povjerenja.
- Regulatorni rizik — poslužitelj koji drži ključeve je de facto skrbnik.

#### Opcija B: OIDC stražnji kanal + samokovanje SBT-a (ODABRANO)

Poslužitelj je isključivo OIDC poštar. Klijent sam kuje svoj SBT prosljeđujući JWT pametnom ugovoru koji ga kriptografski verificira.

**Prednosti:**
- **Kompromis poslužitelja ne ugrožava sredstva niti identitete.** Poslužitelj nema Web3 ključeve i ne može falsificirati AKD potpis.
- **Korisnik zadržava suverenitet.** Privatni ključ nikada ne napušta preglednik/novčanik.
- **Matematički dokaz:** Pametni ugovor sam verificira — nema koraka "vjeruj poslužitelju".
- **Provjerljivost:** Bilo tko može reproducirati verifikaciju prosljeđujući isti JWT ugovoru.

**Nedostaci:**
- **Lančana JWT verifikacija je skupa.** RSA-2048 verifikacija troši ~2M+ goriva na EVM-u. Na Ethereum glavnoj mreži ovo je neprihvatljivo (ali na Gnosisu je jeftino — vidi ADR-003).
- **Kompleksnost pametnog ugovora.** Solidity nema nativne RSA/P-256 biblioteke — trebamo ih implementirati ili koristiti prekompajlirane ugovore.
- **JWT istek.** Token ima kratki životni vijek (obično 5 min). Korisnik mora skovati SBT unutar tog prozora.
- **Korisničko iskustvo.** Korisnik mora imati novčanik — veća barijera od Web2 prijave.

### Odluka

**Opcija B — OIDC stražnji kanal + samokovanje SBT-a.**

### Detaljan tok (8 koraka)

```
Klijent (Preglednik)                  Poslužitelj (Node.js)            Certilia AKD
       |                                    |                               |
   1.  |  Generira novčanik pod             |                               |
       |  vlastitom skrbi                   |                               |
       |  (ili koristi postojeći)           |                               |
       |                                    |                               |
   2.  |  GET /auth/certilia                |                               |
       |  { walletAddress: 0xABC... }       |                               |
       | ---------------------------------> |                               |
       |                                    |                               |
   3.  |                                    |  Generira OIDC auth URL       |
       |                                    |  nonce = 0xABC...             |
       |                                    |  state = CSRF token           |
       |  <-- preusmjereni URL ------------ |                               |
       |                                    |                               |
   4.  |  *** KLIJENT VERIFICIRA ***        |                               |
       |  Parsira URL, provjerava da je     |                               |
       |  nonce == vlastita adresa.         |                               |
       |  Tek tada preusmjerava korisnika.  |                               |
       |                                    |                               |
   5.  |  Korisnik se autenticira           |                               |
       |  biometrijom na mobitelu   ------- | ----------------------------> |
       |                                    |                               |
   6.  |  Autorizacijski kôd (povratni URL) |                               |
       | ---------------------------------> |                               |
       |                                    |  POST /token (S2S)            |
       |                                    |  { code, client_secret }      |
       |                                    | ----------------------------> |
       |                                    |                               |
       |                                    |  <-- Potpisani JWT ---------- |
       |                                    |                               |
   7.  |  <-- neobrađeni JWT -------------- |                               |
       |                                    |                               |
   8.  |  claimCertiliaSBT(jwt)             |                               |
       |  --> Gnosis lanac pametni ugovor   |                               |
       |      Verificira JWT potpis         |                               |
       |      Provjerava nonce == msg.sender|                               |
       |      Kuje SBT na 0xABC...         |                               |
       |                                    |                               |
```

### Zašto svaki korak postoji

| Korak | Svrha | Bez njega? |
|-------|-------|------------|
| 1 | Korisnik kontrolira ključeve | Poslužitelj bi bio skrbnik |
| 2 | Šalje adresu kao nonce | Ne bi bilo vezanja identitet↔novčanik |
| 3 | Poslužitelj čuva `client_secret` | Klijent ne smije imati pristup OIDC tajni |
| 4 | Klijent verificira nonce u URL-u | Zlonamjerni poslužitelj mogao bi podmetnuti tuđu adresu |
| 5 | Biometrička autentikacija | Nema dokaza da je to pravi korisnik |
| 6 | Autorizacijski kôd ide na poslužitelj | Klijent ne može sam zamijeniti kôd za JWT |
| 7 | Poslužitelj prosljeđuje JWT klijentu | Klijent ne bi mogao sam skovati SBT |
| 8 | Lančana verifikacija + kovanje | Nema dokaza identiteta bez povjerenja |

---

## 4. ADR-003: Izbor mreže i korisničko iskustvo s gorivom

### Kontekst

Lančana JWT verifikacija zahtijeva RSA ili P-256 kriptografiju unutar EVM-a. Ovo je računski skupo. Također, novi korisnici nemaju xDAI/ETH za gorivo.

### Zašto Gnosis lanac

| Kriterij | Ethereum glavna mreža | Gnosis lanac |
|----------|----------------------|--------------|
| Cijena goriva (RSA-2048 verif., ~2M goriva) | ~$50-200 USD | ~$0,01-0,05 USD |
| Izvorna Safe podrška | Da | Da (domovina Safe-a) |
| EURe dostupnost | Da | Da (Monerium izvorno podržava) |
| Konačnost | ~12 min | ~5 s |
| Decentralizacija | Najviša | Umjerena (dovoljno za dokaz koncepta) |
| EIP-4337 ekosustav | Zreo | Rastući |

**Odluka:** Gnosis lanac za dokaz koncepta. Dovoljno decentraliziran, drastično jeftinije gorivo, izvorna podrška za Safe i EURe.

**Budućnost:** Ako Ethereum implementira prekompajlirani ugovor za RSA/P-256 (EIP-7212 za P-256 već postoji), migracija na L2 ili glavnu mrežu postaje opcija.

### Uključivanje bez goriva (ERC-4337 platitelj)

Problem: Korisnik upravo napravio novčanik. Nema xDAI. Ne može pozvati `claimCertiliaSBT()`.

```
Korisnik                  Skupljač (Gelato/Biconomy)       Gnosis lanac
   |                              |                            |
   |  Korisnička operacija        |                            |
   |  { calldata: claimSBT(jwt) } |                            |
   | ----------------------------> |                            |
   |                              |  Platitelj plaća gorivo     |
   |                              | --------------------------> |
   |                              |                            |  SBT iskovan
   |                              |  <-- potvrda transakcije - |
   |  <-- potvrda -------------- |                            |
```

**Ograničenja platitelja:**
- Platitelj mora imati popis dopuštenih ili ograničenje po adresi (1 kovanje SBT-a bez goriva po adresi).
- Bez ograničenja, napadač može slati jeftine korisničke operacije i iscrpiti fond platitelja.
- Proračun platitelja je operativni trošak platforme — financira se iz platformskog Safe-a.

---

## 5. Model prijetnji

Potpuna analiza prijetnji izdvojena je u zasebni dokument: **[model-prijetnji.md](model-prijetnji.md)**

Sadrži:
- 15 identificiranih napada (T1-T15) klasificiranih po ozbiljnosti
- Stabla napada za kritične scenarije (krađa identiteta, krađa sredstava)
- Matricu rizika (vjerojatnost × utjecaj)
- Prioritetni popis ublažavanja

---

## 6. Arhitektura pametnih ugovora

### 6.1. CertiliaSBT.sol — Specifikacija

```
CertiliaSBT (ERC-721 + neprenosivo proširenje)
|
|-- Pohrana
|   |-- certiliaPublicKey: bytes        // AKD OIDC javni ključ
|   |-- admin: address                  // Safe višepotpisna adresa
|   |-- identityClaimed: mapping(bytes32 => bool)
|   |-- usedJWTs: mapping(bytes32 => bool)
|   |-- revokedTokens: mapping(uint256 => bool)
|
|-- Funkcije
|   |-- claimCertiliaSBT(bytes jwt) external
|   |   |-- Dekodira JWT zaglavlje + sadržaj + potpis
|   |   |-- Verificira potpis koristeći certiliaPublicKey
|   |   |-- Provjerava: jwt.nonce == msg.sender
|   |   |-- Provjerava: jwt.exp > block.timestamp
|   |   |-- Provjerava: !usedJWTs[hash(jwt)]
|   |   |-- Provjerava: !identityClaimed[hash(jwt.sub)]
|   |   |-- Kuje SBT na msg.sender
|   |
|   |-- revokeSBT(uint256 tokenId) external onlyAdmin
|   |-- updatePublicKey(bytes newKey) external onlyAdmin
|   |-- isVerified(address wallet) external view returns (bool)
|   |
|   |-- transferFrom() --> POGREŠKA (neprenosiv)
|   |-- approve()      --> POGREŠKA (neprenosiv)
```

### 6.2. Safe integracija

```
Safe trezor (Platforma)
|
|-- Zodiac modul uloga
|   |-- Uloga: VERIFICIRANI_KREATOR
|   |-- Uvjet: CertiliaSBT.isVerified(pozivatelj) == true
|   |-- Dozvole: Može inicirati isplatu do X EURe
|
|-- Zodiac stražar (opcija)
|   |-- Kuka prije transakcije
|   |-- Provjerava da primatelj isplate ima validan SBT
```

### 6.3. Razdjelnik plaćanja

```
RazdjelnikPlacanja (po kreatoru ili 0xSplits v2)
|
|-- Konfiguracija
|   |-- primatelji: [adresaKreatora, platformskiSafe]
|   |-- udjeli: [90, 10]  // bazne točke: [9000, 1000]
|
|-- Funkcije
|   |-- split(uint256 iznos) external
|   |   |-- EURe.transferFrom(msg.sender, kreator, iznos * 90 / 100)
|   |   |-- EURe.transferFrom(msg.sender, platformskiSafe, iznos * 10 / 100)
```

---

## 7. Neriješena pitanja i sljedeći koraci

### Kritična (blokiraju implementaciju)

| # | Pitanje | Kontekst |
|---|---------|----------|
| Q1 | Koji algoritam koristi Certilia za JWT potpis? RSA-2048, RSA-4096, P-256, ili nešto drugo? | Određuje koji prekompajlirani ugovor/biblioteku trebamo na Gnosisu. EIP-7212 pokriva P-256 ali ne RSA. Za RSA trebamo vlastitu implementaciju ili čekati prekompajlirani ugovor. |
| Q2 | Što je `sub` tvrdnja u Certilia JWT-u? | Mora biti deterministički jedinstvena po građaninu (sažetak OIB-a ili ekvivalent). Ako se temelji na sesiji, Sybil zaštita ne radi. |
| Q3 | Podržava li Certilia prilagođeni `nonce` parametar u OIDC zahtjevu? | Cijela arhitektura ovisi o mogućnosti slanja adrese novčanika kao nonce. Ako ne podržava, trebamo alternativni mehanizam vezanja. |
| Q4 | Postoji li Certilia testno okruženje? | Bez testnog okruženja ne možemo razvijati bez korištenja pravih identiteta. |

### Važna (utječu na dizajn)

| # | Pitanje | Kontekst |
|---|---------|----------|
| Q5 | Koji je najjeftiniji način za lančanu RSA verifikaciju na Gnosisu? | Opcije: (a) vlastita Solidity RSA biblioteka (npr. adria0/SolRSA), (b) čekati izvorni prekompajlirani ugovor, (c) ZK dokaz izvan lanca + lančani verifikator. |
| Q6 | 0xSplits v2 ili vlastiti razdjelnik? | 0xSplits je revidiran i dokazan, ali dodaje ovisnost. Vlastiti daje potpunu kontrolu. |
| Q7 | Kako riješiti problem korisničkog iskustva s istekom JWT-a? | JWT živi ~5 min. Korisnik mora imati novčanik spreman i dovoljno brzo potpisati transakciju. Trebamo tok koji minimizira trenje. |
| Q8 | ERC-4337 nasuprot Gelato posredniku za iskustvo bez goriva? | ERC-4337 je standard ali kompleksniji. Gelato je jednostavniji ali centraliziraniji. |

### Istraživačka (dugoročno)

| # | Pitanje | Kontekst |
|---|---------|----------|
| Q9 | ZK-SNARK za privatnost glasanja? | Trenutno je SBT javno vidljiv. Za glasanje možda trebamo ZK dokaz da osoba IMA SBT bez otkrivanja KOJI SBT/novčanik. |
| Q10 | Višedržavna eIDAS podrška? | Certilia je samo za Hrvatsku. eIDAS standard pokriva cijelu EU. Možemo li generalizirati na druge države? |
| Q11 | Obnova SBT-a nasuprot trajnosti? | Treba li SBT imati istek (npr. poklapa se s rokom osobne iskaznice) ili je trajan dok se ne opozove? |

---

## Dodatak: Usporedba arhitektura

```
                  Svemoćni pozadinski       OIDC + samokovanje SBT-a
                  sustav (ODBAČENO)         (ODABRANO)

Hakiran poslužit. = Potpuni kompromis       Uskrata usluge (sredstva sigurna)
Lažni identitet  = Poslužitelj može kovati  Nemoguće bez AKD potpisa
Skrb korisnika   = Poslužitelj drži ključ.  Korisnik drži ključeve
Provjerljivost   = "Vjeruj poslužitelju"    Matematički dokaz na lancu
Trošak goriva    = Nizak (obično kovanje)   Visok (JWT verif.) — ali Gnosis!
Kor. iskustvo    = Jednostavno (Web2)       Kompleksnije (treba novčanik)
Regulatorno      = Skrbnik (MiCA rizik)     Bez skrbništva
```
