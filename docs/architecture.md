# Zapis arhitektonskih odluka

**Projekt:** javni-novci / svima-a-ne-samo-njima
**Verzija:** v0.4-nacrt
**Status:** Iteracija — poziv na tehničku recenziju

---

## Sadržaj

1. [Pregled sustava](#1-pregled-sustava)
2. [ADR-001: Arhitektura trezora i tok sredstava](#2-adr-001-arhitektura-trezora-i-tok-sredstava)
3. [ADR-002: Verifikacija identiteta i OAuth2 tok](#3-adr-002-verifikacija-identiteta-i-oauth2-tok)
4. [ADR-003: Izbor mreže i korisničko iskustvo s gorivom](#4-adr-003-izbor-mreže-i-korisničko-iskustvo-s-gorivom)
5. [ADR-004: Privatnost lančanih podataka](#5-adr-004-privatnost-lančanih-podataka)
6. [Model prijetnji](#6-model-prijetnji)
7. [Arhitektura pametnih ugovora](#7-arhitektura-pametnih-ugovora)
8. [Centralni registar otvorenih pitanja](#8-centralni-registar-otvorenih-pitanja)

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
| Posrednik | Gelato / Biconomy (ERC-4337) | Kovanje SBT-a bez goriva za nove korisnike (vidi Q8) |

### Temeljno načelo

**Niti jedna komponenta ne smije imati dovoljno moći da samostalno kompromitira sustav.**

- Poslužitelj ne drži Web3 ključeve — ne može trošiti sredstva.
- Poslužitelj ne može falsificirati Certilia potpis — ne može kovati lažne identitete.
- Klijent ne može zaobići poslužitelj — ne može sam dobiti JWT bez `client_secret` stražnjeg kanala.
- Pametni ugovor ne vjeruje nikome — sam verificira JWT kriptografski.

### Ključna ograničenja privatnosti

**UPOZORENJE:** Ovaj sustav još NE jamči potpunu privatnost na lancu. Detaljan pregled problema i rješenja: [privatnost.md](privatnost.md). Sažetak:

- **Faza 1:** Selektivno prosljeđivanje JWT-a minimizira izloženost, ali `identityHash` u pohrani ostaje pseudonimni identifikator (GDPR rizik — vidi [pravna-analiza.md](pravna-analiza.md)).
- **Faza 2+:** Nullifier pristup eliminira vezu između identiteta i lančanih podataka.

```
+------------------+     OIDC      +--------------------+     S2S      +----------------+
|                  | -- nonce -->  |                    | ----------> |                |
|  Klijent         |  auth code    |  Node.js           |   JWT       |  Certilia AKD  |
|  (Novčanik pod   | <-- JWT ---  |  Poslužitelj        | <---------- |  (eIDAS OIDC)  |
|   vlastitom      |               |  (Poštar)          |             |                |
|   skrbi)         |               |                    |             |                |
+--------+---------+               +--------------------+             +----------------+
         |
         |  claimCertiliaSBT(jwtProof)
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

### Kvantitativna matrica

| Metrika (težina) | A: Monolitni Safe | B: Razdjelnik plaćanja | C: Zodiac modul dopuštenja |
|-------------------|--------------------|------------------------|---------------------------|
| Sigurnost i odsutnost povjerenja (40%) | 30 — sva sredstva na jednom mjestu, kvorum potpisnika je SPOF | 85 — kreatorova sredstva nikad u našem trezoru, atomičnost | 50 — sredstva pomiješana, modul dodaje napadačku površinu |
| Korisničko iskustvo (20%) | 20 — svaka isplata čeka M-od-N potpis | 80 — automatska isplata, bez čekanja | 70 — automatske male isplate, ali sa ograničenjima |
| Gorivo i mrežna učinkovitost (20%) | 30 — Safe transakcije troše višestruko više goriva | 75 — jedan split = jedan poziv ugovora | 60 — Safe overhead + modul overhead |
| Skalabilnost i nadogradivost (20%) | 20 — svaka mikrotransakcija = potpisivanje | 70 — linearna konfiguracija po kreatoru | 55 — limiti zahtijevaju stalno kalibriranje |

| Opcija | Ponderirani prosjek |
|--------|---------------------|
| A: Monolitni Safe | **27** |
| B: Razdjelnik plaćanja | **80** |
| C: Zodiac modul dopuštenja | **56** |

### Odluka

**Opcija B — Razdjelnik plaćanja.**

### Sigurnosne implikacije

- Kreator mora sam čuvati novčanik (odgovornost vlastite skrbi).
- Adrese primatelja moraju biti nepromjenjive u razdjelniku ILI promjenjive samo od strane Safe administratora.
- **Zaokruživanje:** Cjelobrojno dijeljenje u Solidity-u zaokružuje prema dolje. Za iznose koji nisu djeljivi bez ostatka, prašina (ostatak od dijeljenja) ostaje u razdjelniku. Trebamo `sweep()` funkciju kojom Safe periodički povlači akumuliranu prašinu, ili koristiti bazne točke (bips) za preciznost.

### Neriješeno

- Q6: 0xSplits v2 (revidirano, dokazano) ili vlastiti razdjelnik (potpuna kontrola)? Zahtijeva ADR.

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

### Kvantitativna matrica

| Metrika (težina) | A: Svemoćni pozadinski sustav | B: OIDC stražnji kanal + samokovanje |
|-------------------|-------------------------------|--------------------------------------|
| Sigurnost i odsutnost povjerenja (40%) | 10 — katastrofalna jedna točka kvara, server = bog | 85 — server kompromis = samo uskrata usluge |
| Korisničko iskustvo (20%) | 90 — Web2 prijava, korisnik ne vidi blockchain | 45 — treba novčanik, razumijevanje potpisivanja |
| Gorivo i mrežna učinkovitost (20%) | 85 — obično kovanje, jeftino | 40 — RSA verifikacija ~2M+ goriva (ali Gnosis!) |
| Skalabilnost i nadogradivost (20%) | 30 — server je usko grlo i regulatorni rizik (skrbnik) | 80 — svaki korisnik sam kuje, nema uskog grla |

| Opcija | Ponderirani prosjek |
|--------|---------------------|
| A: Svemoćni pozadinski sustav | **39** |
| B: OIDC stražnji kanal + samokovanje | **68** |

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
   7.  |  GET /auth/jwt/{sessionId}         |                               |
       |  <-- JWT (jednokratno) ----------- |                               |
       |                                    |                               |
   8.  |  claimCertiliaSBT(jwtProof)        |                               |
       |  --> Gnosis lanac pametni ugovor   |                               |
       |      Verificira JWT potpis         |                               |
       |      Provjerava nonce == msg.sender|                               |
       |      Kuje SBT na 0xABC...         |                               |
       |                                    |                               |
```

### Korak 7 — prijenos JWT-a klijentu (POJAŠNJENO)

Standardni OIDC stražnji kanal vraća JWT isključivo poslužitelju. Klijent ga treba za lančanu verifikaciju. Mehanizam prijenosa:

1. Poslužitelj sprema JWT u privremenu pohranu s ključem `sessionId` i rokom trajanja (5 min, kao i sam JWT).
2. Klijent dohvaća JWT putem `GET /auth/jwt/{sessionId}` — jednokratna krajnja točka koja briše JWT nakon prvog čitanja.
3. Prijenos zaštićen: HTTPS + CORS ograničenja + jednokratno čitanje + automatski istek.

**Sigurnosna analiza ovog koraka:**
- MITM: HTTPS eliminira presretanje u prijenosu.
- XSS: Ako napadač ima XSS na našoj domeni, može dohvatiti JWT — ali JWT nonce je vezan za korisnikovu adresu, pa napadač ne može skovati SBT na svoju adresu (ugovor provjerava `nonce == msg.sender`).
- Istek: JWT + pohrana na poslužitelju istječu paralelno. Nema trajnog skladištenja.

### Zašto svaki korak postoji

| Korak | Svrha | Bez njega? |
|-------|-------|------------|
| 1 | Korisnik kontrolira ključeve | Poslužitelj bi bio skrbnik |
| 2 | Šalje adresu kao nonce | Ne bi bilo vezanja identitet↔novčanik |
| 3 | Poslužitelj čuva `client_secret` | Klijent ne smije imati pristup OIDC tajni |
| 4 | Klijent verificira nonce u URL-u | Zlonamjerni poslužitelj mogao bi podmetnuti tuđu adresu |
| 5 | Biometrička autentikacija | Nema dokaza da je to pravi korisnik |
| 6 | Autorizacijski kôd ide na poslužitelj | Klijent ne može sam zamijeniti kôd za JWT |
| 7 | Poslužitelj jednokratno izlaže JWT klijentu | Klijent ne bi mogao sam skovati SBT |
| 8 | Lančana verifikacija + kovanje | Nema dokaza identiteta bez povjerenja |

### Sigurnosne implikacije

- Korak 7 uvodi novu krajnju točku na poslužitelju — napadačka površina. Ublažavanje: jednokratno čitanje, automatski istek, CORS.
- JWT u prijenosu je izložen XSS-u — ali nonce vezanje čini presretnuti JWT beskorisnim za napadača.

### Neriješeno

- Q3: Podržava li Certilia prilagođeni `nonce`? Bez toga, cijeli tok propada.
- Q9: Treba li korak 7 koristiti WebSocket umjesto pollinga za bolje korisničko iskustvo?

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
- Ograničenje: 1 kovanje bez goriva po Certilia identitetu (ne po adresi — adrese se mogu generirati besplatno, ali Certilia identiteti ne).
- Dnevni proračunski limit platitelja.
- Proračun platitelja je operativni trošak platforme — financira se iz platformskog Safe-a.

### Neriješeno

- Q8: ERC-4337 nasuprot Gelato posredniku? Zahtijeva ADR s matricom bodovanja.

---

## 5. ADR-004: Privatnost lančanih podataka

Potpuna analiza izdvojena u: **[privatnost.md](privatnost.md)**

Sažetak odluke: **Opcija C (hibridni pristup)**
- Faza 1: Selektivno prosljeđivanje (Opcija A) — na lanac idu samo potpis + sažetak + nonce + hash(sub). Osobni podaci NE idu u calldata.
- Faza 2+: Nullifier pristup (Opcija B) — potpuna privatnost putem ZK dokaza.

**Kritično za arhitekturu:** Ova odluka zahtijeva da CertiliaSBT ugovor bude dizajniran s **zamjenjivom verifikacijskom logikom** od prvog dana. Vidi odjeljak 7.1.

---

## 6. Model prijetnji

Potpuna analiza izdvojena u: **[model-prijetnji.md](model-prijetnji.md)**

Sadrži:
- 15 identificiranih napada (T1-T15) klasificiranih po ozbiljnosti
- Stabla napada za kritične scenarije (krađa identiteta, krađa sredstava)
- Matricu rizika (vjerojatnost × utjecaj)
- Prioritetni popis ublažavanja

---

## 7. Arhitektura pametnih ugovora

### 7.1. CertiliaSBT.sol — Specifikacija

```
CertiliaSBT (ERC-721 + ERC-5192 neprenosivo sučelje)
|
|-- Pohrana
|   |-- certiliaPublicKeys: mapping(bytes32 => bool)  // VIŠE KLJUČEVA (rotacija!)
|   |-- admin: address                                 // Safe višepotpisna adresa
|   |-- identityClaimed: mapping(bytes32 => address)   // hash(sub) => SBT adresa
|   |-- usedJWTs: mapping(bytes32 => bool)
|   |-- revokedTokens: mapping(uint256 => bool)
|   |-- verifierVersion: uint8                         // za migraciju A→B
|
|-- Funkcije
|   |-- claimCertiliaSBT(bytes jwtProof) external
|   |   |-- Dekodira dokaz (format ovisi o verifierVersion)
|   |   |-- Verificira potpis koristeći certiliaPublicKeys
|   |   |-- Provjerava: jwt.nonce == msg.sender
|   |   |-- Provjerava: jwt.exp > block.timestamp
|   |   |-- Provjerava: !usedJWTs[hash(jwt)]
|   |   |-- Provjerava: !identityClaimed[hash(jwt.sub)]
|   |   |-- Kuje SBT na msg.sender
|   |
|   |-- resetIdentityClaim(bytes32 identityHash) external onlyAdmin
|   |   |-- Briše identityClaimed[identityHash]
|   |   |-- Dozvoljava korisniku ponovno kovanje na novoj adresi
|   |   |-- MORA se pozvati NAKON revokeSBT() na staroj adresi
|   |
|   |-- revokeSBT(uint256 tokenId) external onlyAdmin
|   |-- addPublicKey(bytes32 keyHash) external onlyAdmin
|   |-- removePublicKey(bytes32 keyHash) external onlyAdmin
|   |-- isVerified(address wallet) external view returns (bool)
|   |-- locked(uint256 tokenId) external view returns (bool)  // ERC-5192
|   |
|   |-- transferFrom() --> POGREŠKA (neprenosiv — ERC-5192)
|   |-- approve()      --> POGREŠKA (neprenosiv — ERC-5192)
```

**Promjene u odnosu na v0.3:**

1. **Višestruki ključevi:** `certiliaPublicKeys` je sada mapa umjesto jednog bajt-niza. Podržava preklapanje pri rotaciji — oba ključa (stari i novi) mogu biti aktivna istovremeno. Safe administrator dodaje novi ključ PRIJE nego AKD povuče stari.

2. **`resetIdentityClaim()`:** Rješava problem "trajne zabrane". Kada korisnik izgubi novčanik:
   - Safe opoziva SBT na staroj adresi (`revokeSBT`).
   - Safe briše identitetnu tvrdnju (`resetIdentityClaim`).
   - Korisnik se ponovno autenticira s Certilijom i kuje novi SBT na novoj adresi.
   - Atomičnost: u budućnosti oba koraka mogu ići u jednu Safe transakciju.

3. **`verifierVersion`:** Priprema za migraciju s Opcije A (selektivno prosljeđivanje) na Opciju B (nullifier). Verzija 1 = JWT verifikacija. Verzija 2 = ZK verifikator. Logika `claimCertiliaSBT` čita verziju i poziva odgovarajući interni verifikator.

4. **ERC-5192:** Umjesto prilagođenog `transferFrom() → revert`, koristimo standardizirano ERC-5192 sučelje. Kompatibilno s alatima koji prepoznaju neprenosive tokene (OpenSea, tržišta, indexeri).

### 7.2. Safe integracija

```
Safe trezor (Platforma)
|
|-- Zodiac modul uloga (FAZA 2 — ne za dokaz koncepta)
|   |-- Uloga: VERIFICIRANI_KREATOR
|   |-- Uvjet: CertiliaSBT.isVerified(pozivatelj) == true
|   |-- Dozvole: Može inicirati isplatu do X EURe
|
|-- Zodiac stražar (opcija, FAZA 2)
|   |-- Kuka prije transakcije
|   |-- Provjerava da primatelj isplate ima validan SBT
```

**Pojašnjenje:** Za fazu 1 (dokaz koncepta), Safe funkcionira kao obični višepotpisni novčanik (2/3) za operativne troškove platforme. Zodiac moduli su odgođeni za fazu 2 kada DAO upravljanje postaje relevantno.

### 7.3. Razdjelnik plaćanja

```
RazdjelnikPlacanja (po kreatoru ili 0xSplits v2)
|
|-- Konfiguracija
|   |-- primatelji: [adresaKreatora, platformskiSafe]
|   |-- udjeli: [9000, 1000]  // bazne točke (ukupno 10000)
|
|-- Funkcije
|   |-- split(uint256 iznos) external
|   |   |-- uKreatoru = iznos * 9000 / 10000
|   |   |-- uPlatformu = iznos - uKreatoru   // ostatak ide platformi (nema gubitka prašine)
|   |   |-- EURe.transferFrom(msg.sender, kreator, uKreatoru)
|   |   |-- EURe.transferFrom(msg.sender, platformskiSafe, uPlatformu)
```

**Promjena:** Zaokruživanje riješeno oduzimanjem — `uPlatformu = iznos - uKreatoru`. Kreator uvijek dobiva zaokruženo prema dolje, a sva prašina ide platformi. Nijedan wei se ne gubi.

---

## 8. Centralni registar otvorenih pitanja

Sva otvorena pitanja iz svih dokumenata, mapirana na faze i ovisnosti.

### Kritična (blokiraju fazu 1)

| # | Pitanje | Izvor | Ovisnosti |
|---|---------|-------|-----------|
| Q1 | Koji algoritam koristi Certilia za JWT potpis? | architecture.md | Određuje verifikacijsku biblioteku |
| Q2 | Što je `sub` tvrdnja u Certilia JWT-u? Je li deterministički jedinstvena po građaninu? | architecture.md | Sybil zaštita, privatnost.md hash problem |
| Q3 | Podržava li Certilia prilagođeni `nonce` parametar? | architecture.md | Cijeli OIDC tok |
| Q4 | Postoji li Certilia testno okruženje? | architecture.md | Razvoj bez pravih identiteta |
| P4 | Je li hash(`sub`) "osobni podatak" po GDPR-u? Trebamo li DPIA? | privatnost.md, pravna-analiza.md | Faza 1 regulatorni rizik |
| L1 | Zabranjuju li Certilia uvjeti korištenja prosljeđivanje JWT-a pametnim ugovorima? | pravna-analiza.md | Legalnost cijelog projekta |

### Važna (utječu na dizajn faze 1)

| # | Pitanje | Izvor | Ovisnosti |
|---|---------|-------|-----------|
| Q5 | Najjeftiniji način za lančanu RSA verifikaciju na Gnosisu? | architecture.md | Ovisi o Q1 |
| Q6 | 0xSplits v2 ili vlastiti razdjelnik? | architecture.md | Zahtijeva ADR |
| Q8 | ERC-4337 nasuprot Gelato posredniku? | architecture.md | Zahtijeva ADR |
| Q9 | Korak 7: Polling ili WebSocket za JWT dohvat? | architecture.md | UX |
| L5 | Trebamo li CASP licencu? | pravna-analiza.md | HANFA upit |
| E1 | Fiksni (90/10) ili DAO-upravljani postotak raspodjele? | ekonomski-model.md | Za fazu 1: fiksni. DAO tek u fazi 2. |

### Blokiraju fazu 2

| # | Pitanje | Izvor | Ovisnosti |
|---|---------|-------|-----------|
| P1 | Koji ZK sustav? Groth16, PLONK, Halo2? | privatnost.md | Nullifier implementacija |
| P2 | Može li se RSA/P-256 učinkovito implementirati u ZK krugu? | privatnost.md | Ovisi o Q1 |
| P3 | Kako riješiti opozivost u nullifier modelu? | privatnost.md | ZK oporavak novčanika |

### Istraživačka (faza 3+)

| # | Pitanje | Izvor |
|---|---------|-------|
| Q10 | Višedržavna eIDAS podrška | architecture.md |
| Q11 | SBT s istekom nasuprot trajnosti | architecture.md |
| L4 | eIDAS 2.0 / EUDI utjecaj na arhitekturu | pravna-analiza.md |

---

## Dodatak: Sučelje za identitetnog pružatelja (apstrakcijski sloj)

Za kompatibilnost s budućim EUDI novčanikom (eIDAS 2.0), sustav mora od prvog dana koristiti apstrakcijski sloj za identitetnog pružatelja:

```
Sučelje: IIdentityVerifier
|
|-- verifyProof(bytes proof, address claimer) returns (bool valid, bytes32 identityHash)
|
|-- Implementacija v1: CertiliaJWTVerifier
|   |-- Parsira JWT, verificira RSA/P-256 potpis, provjerava nonce
|
|-- Implementacija v2: CertiliaNullifierVerifier (faza 2)
|   |-- Verificira ZK dokaz, provjerava nullifier jedinstvenost
|
|-- Implementacija v3: EUDIVerifier (faza 3)
|   |-- Verificira W3C Verifiable Credential iz EUDI novčanika
```

CertiliaSBT.sol poziva `IIdentityVerifier(verifierAddress).verifyProof(...)` umjesto hardkodirane JWT logike. Safe administrator može postaviti novu implementaciju verifikatora bez zamjene SBT ugovora.

**Ovo rješava:**
- Migraciju Opcija A → Opcija B bez novog ugovora.
- Buduću podršku za EUDI bez novog ugovora.
- Višedržavnu eIDAS podršku (različiti verifikatori po državi).

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
