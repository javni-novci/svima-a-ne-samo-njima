# Architecture Decision Record & Threat Model

**Projekt:** javni-novci / svima-a-ne-samo-njima
**Verzija:** v0.2-draft
**Status:** Iteracija — poziv na tehnicku recenziju

---

## Sadrzaj

1. [Pregled sustava](#1-pregled-sustava)
2. [ADR-001: Treasury arhitektura i tok sredstava](#2-adr-001-treasury-arhitektura-i-tok-sredstava)
3. [ADR-002: Identity verifikacija i OAuth2 tok](#3-adr-002-identity-verifikacija-i-oauth2-tok)
4. [ADR-003: Izbor mreze i Gas UX](#4-adr-003-izbor-mreze-i-gas-ux)
5. [Analiza prijetnji (Threat Model)](#5-analiza-prijetnji-threat-model)
6. [Arhitektura pametnih ugovora](#6-arhitektura-pametnih-ugovora)
7. [Nerijesena pitanja i sljedeci koraci](#7-nerijesena-pitanja-i-sljedeci-koraci)

---

## 1. Pregled sustava

### Stack

| Sloj | Tehnologija | Uloga |
|------|-------------|-------|
| Identitet | Certilia Mobile.id (AKD) | eIDAS OIDC provider — pravni identitet gradana |
| Blockchain | Gnosis Chain | Izvrsavanje pametnih ugovora, jeftin gas |
| Valuta | EURe (Monerium) | MiCA uskladeni EUR stablecoin |
| Trezor | Safe (ex-Gnosis Safe) | Multisig upravljanje sredstvima |
| Identitetni token | Soulbound Token (SBT) | Neprenosiva on-chain znacka verificiranog gradanina |
| Backend | Node.js | OIDC postar — nema Web3 kljuceve |
| Relayer | Gelato / Biconomy (ERC-4337) | Gasless SBT mintanje za nove korisnike |

### Temeljno nacelo

**Niti jedna komponenta ne smije imati dovoljno moci da samostalno kompromitira sustav.**

- Server ne drzi Web3 kljuceve — ne moze trositi sredstva.
- Server ne moze falsificirati Certilia potpis — ne moze mintati lazne identitete.
- Klijent ne moze zaobici server — ne moze sam dobiti JWT bez back-channel `client_secret`.
- Pametni ugovor ne vjeruje nikome — sam verificira JWT kriptografski.

```
+------------------+     OIDC      +------------------+     S2S      +----------------+
|                  | -- nonce -->  |                  | ----------> |                |
|  Klijent         |  auth code    |  Node.js Server  |   JWT       |  Certilia AKD  |
|  (Self-custody   | <-- JWT ---  |  (Postar)        | <---------- |  (eIDAS OIDC)  |
|   Wallet)        |               |                  |             |                |
+--------+---------+               +------------------+             +----------------+
         |
         |  claimCertiliaSBT(jwt)
         v
+---------------------------+          +---------------------------+
|  CertiliaSBT.sol          |          |  Safe Treasury            |
|  Gnosis Chain             |  ------> |  + Zodiac Role Module     |
|  On-chain JWT verifikacija|          |  + Payment Splitter       |
+---------------------------+          +---------------------------+
```

---

## 2. ADR-001: Treasury arhitektura i tok sredstava

### Kontekst

Podcast marketplace prima uplate od korisnika u EURe. Sredstva se moraju podijeliti izmedu kreatora (90%) i platforme (10%). Trebamo odluciti kako strukturirati tok sredstava.

### Razmatrane opcije

#### Opcija A: Monolitni Safe trezor (ODBACENO)

Sva sredstva (uplate korisnika, zarada kreatora, platformske naknade) ulaze u jedan Safe multisig. Signeri odobravaju svaku isplatu.

**Prednosti:**
- Jednostavna implementacija — jedan ugovor, jedan trezor.
- Potpuna kontrola signera nad svim sredstvima.

**Nedostaci:**
- **Bottleneck:** Svaka mikrotransakcija (npr. 2 EURe za epizodu podcasta) zahtijeva M-od-N potpis. Neskalirano.
- **Honeypot:** Sva sredstva na jednoj adresi. Ako kompromitiramo kvoruma signera, gubimo SVE.
- **Racunovodstvena nocna mora:** Nemoguce razlikovati korisnicke depozite od platformske zarade bez off-chain bookkeepinga.
- **Gas trosak:** Svaka Safe transakcija trosi visestruko vise gasa od obicnog transfera.

#### Opcija B: Payment Splitter / 0xSplits (ODABRANO)

Svaka uplata prolazi kroz automatski splitter ugovor koji u istoj transakciji rutira sredstva na predodredene adrese.

**Prednosti:**
- **Atomicnost:** Uplata se dijeli u jednom koraku. Nema stanja "sredstva zapela u tranzitu".
- **Nema manualne intervencije:** Kreator prima svoj udio automatski. Nema cekanja na signer approval.
- **Izolacija rizika:** Platformski Safe drzi samo 10% prihoda. Kreatorova sredstva nikada nisu u nasem trezoru.
- **Auditabilnost:** Svaki split je on-chain vidljiv. Bilo tko moze verificirati da kreator prima tocno 90%.

**Nedostaci:**
- Kompleksnija inicijalna konfiguracija (deploy splitter-a po kreatoru ili koristenje 0xSplits v2 s pull-based distribucijom).
- Kreator mora sam curati svoj novcanik (self-custody odgovornost).

#### Opcija C: Zodiac Allowance Module (ALTERNATIVA)

Safe s Zodiac modulom koji dozvoljava automatske male isplate do odredenog limita bez signera.

**Prednosti:**
- Sva sredstva i dalje prolaze kroz Safe — jedinstven audit trail.
- Automatske isplate do X EURe dnevno bez signer quoruma.

**Nedostaci:**
- Sredstva i dalje kominglovana u jednom trezoru (honeypot rizik ostaje).
- Zodiac modul dodaje napadacku povrsinu na Safe.
- Limiti se moraju pazljivo kalibrirati — previse restriktivno = bottleneck, previse liberalno = rizik drainanja.

### Odluka

**Opcija B — Payment Splitter** za korisnicke uplate. Platformski Safe prima samo svoj udio (10%). Za buducnost, Zodiac Allowance Module moze sluziti za operativne troskove platforme iz Safe-a.

### Tok sredstava (PoC)

```
Korisnik
  |
  |  Placa 10 EURe za podcast epizodu
  v
+---------------------------+
|  Payment Splitter         |
|  (0xSplits ili custom)    |
+---------------------------+
  |                    |
  |  9 EURe            |  1 EURe
  v                    v
+---------------+    +--------------------+
|  Kreator      |    |  Platform Safe     |
|  (Self-custody|    |  (Multisig 2/3)    |
|   wallet)     |    |  Operativni fond   |
+---------------+    +--------------------+
```

---

## 3. ADR-002: Identity verifikacija i OAuth2 tok

### Kontekst

Moramo povezati hrvatski pravni identitet (Certilia/eIDAS) s Ethereum adresom. Kljucni problem: **Certilia koristi RSA/X.509 potpise (eIDAS standard), a EVM koristi secp256k1.** Safe ne moze nativno citati Certilia potpise.

### Razmatrane opcije

#### Opcija A: God-Mode Backend (ODBACENO)

Node.js server drzi master Web3 privatni kljuc. Kada korisnik prode Certilia autentikaciju, server potpisuje on-chain atestaciju i minta SBT u ime korisnika.

**Prednosti:**
- Trivijalna implementacija — standardni Web2 auth + server-side Web3 transakcija.
- Korisnik ne treba razumjeti blockchain.

**Nedostaci:**
- **Katastrofalni Single Point of Failure.** Kompromitiran server = napadac moze:
  - Mintati neogranicen broj laznih SBT-ova (lazni identiteti).
  - Trositi sredstva iz trezora ako je server kljuc signer na Safe-u.
  - Opozvati legitimne SBT-ove.
- Proturecje cijeloj "trustless" filozofiji projekta.
- Regulatorni rizik — server koji drzi kljuceve je de facto custodian.

#### Opcija B: OIDC Back-channel + Self-Minting SBT (ODABRANO)

Server je iskljucivo OIDC postar. Klijent sam minta svoj SBT prosljedujuci JWT pametnom ugovoru koji ga kriptografski verificira.

**Prednosti:**
- **Server kompromis ne ugrozava sredstva niti identitete.** Server nema Web3 kljuceve i ne moze falsificirati AKD potpis.
- **Korisnik zadrzava suverenitet.** Privatni kljuc nikada ne napusta preglednik/wallet.
- **Matematicki dokaz:** Pametni ugovor sam verificira — nema "vjeruj serveru" koraka.
- **Auditabilnost:** Bilo tko moze reproducirati verifikaciju prosljedujuci isti JWT ugovoru.

**Nedostaci:**
- **On-chain JWT verifikacija je skupa.** RSA-2048 verifikacija trosi ~2M+ gasa na EVM. Na Ethereum mainnet ovo je neprihvatljivo (ali na Gnosisu je jeftino — vidi ADR-003).
- **Kompleksnost pametnog ugovora.** Solidity nema nativne RSA/P-256 biblioteke — trebamo ih implementirati ili koristiti precompiles.
- **JWT expiry.** Token ima kratki TTL (obicno 5min). Korisnik mora mintati SBT unutar tog prozora.
- **Korisnicko iskustvo.** Korisnik mora imati wallet — veca barijera od Web2 login-a.

### Odluka

**Opcija B — OIDC Back-channel + Self-Minting SBT.**

### Detaljan tok (8 koraka)

```
Klijent (Preglednik)                  Server (Node.js)                 Certilia AKD
       |                                    |                               |
   1.  |  Generira self-custody wallet      |                               |
       |  (ili koristi postojeci)           |                               |
       |                                    |                               |
   2.  |  GET /auth/certilia                |                               |
       |  { walletAddress: 0xABC... }       |                               |
       | ---------------------------------> |                               |
       |                                    |                               |
   3.  |                                    |  Generira OIDC auth URL       |
       |                                    |  nonce = 0xABC...             |
       |                                    |  state = CSRF token           |
       |  <-- redirect URL ---------------  |                               |
       |                                    |                               |
   4.  |  *** KLIJENT VERIFICIRA ***        |                               |
       |  Parsira URL, provjerava da je     |                               |
       |  nonce == vlastita adresa.         |                               |
       |  Tek tada redirecta korisnika.     |                               |
       |                                    |                               |
   5.  |  Korisnik autenticira se           |                               |
       |  biometrijom na mobitelu   ------- | ----------------------------> |
       |                                    |                               |
   6.  |  Auth Code (via callback)          |                               |
       | ---------------------------------> |                               |
       |                                    |  POST /token (S2S)            |
       |                                    |  { code, client_secret }      |
       |                                    | ----------------------------> |
       |                                    |                               |
       |                                    |  <-- Signed JWT ------------- |
       |                                    |                               |
   7.  |  <-- raw JWT --------------------- |                               |
       |                                    |                               |
   8.  |  claimCertiliaSBT(jwt)             |                               |
       |  --> Gnosis Chain pametni ugovor   |                               |
       |      Verificira JWT potpis         |                               |
       |      Provjerava nonce == msg.sender|                               |
       |      Minta SBT na 0xABC...        |                               |
       |                                    |                               |
```

### Zasto svaki korak postoji

| Korak | Svrha | Bez njega? |
|-------|-------|------------|
| 1 | Korisnik kontrolira kljuceve | Server bi bio custodian |
| 2 | Salje adresu kao nonce | Ne bi bilo binding-a identitet<->wallet |
| 3 | Server cuva `client_secret` | Klijent ne smije imati pristup OIDC secret-u |
| 4 | Klijent verificira nonce u URL-u | Zlonamjerni server mogao bi podmetnuti tudu adresu |
| 5 | Biometricka autentikacija | Nema dokaza da je to pravi korisnik |
| 6 | Auth code ide na server | Klijent ne moze sam zamijeniti code za JWT |
| 7 | Server prosljeduje JWT klijentu | Klijent ne bi mogao sam mintati SBT |
| 8 | On-chain verifikacija + mint | Nema trustless dokaza identiteta |

---

## 4. ADR-003: Izbor mreze i Gas UX

### Kontekst

On-chain JWT verifikacija zahtijeva RSA ili P-256 kriptografiju unutar EVM-a. Ovo je racunski skupo. Takoder, novi korisnici nemaju xDAI/ETH za gas.

### Zasto Gnosis Chain

| Kriterij | Ethereum Mainnet | Gnosis Chain |
|----------|------------------|--------------|
| Gas cijena (RSA-2048 verifikacija, ~2M gas) | ~$50-200 USD | ~$0.01-0.05 USD |
| Native Safe podrska | Da | Da (domovina Safe-a) |
| EURe dostupnost | Da | Da (Monerium nativno podrzava) |
| Finality | ~12min | ~5s |
| Decentralizacija | Najvisa | Umjerena (dovoljno za PoC) |
| EIP-4337 ekosustav | Zreo | Rastuci |

**Odluka:** Gnosis Chain za PoC. Dovoljno decentraliziran, drasticno jeftiniji gas, nativna podrska za Safe i EURe.

**Buducnost:** Ako Ethereum implementira precompile za RSA/P-256 (EIP-7212 za P-256 vec postoji), migracija na L2 ili mainnet postaje opcija.

### Gasless onboarding (ERC-4337 Paymaster)

Problem: Korisnik upravo napravio wallet. Nema xDAI. Ne moze pozvati `claimCertiliaSBT()`.

```
Korisnik                  Bundler (Gelato/Biconomy)        Gnosis Chain
   |                              |                            |
   |  UserOperation               |                            |
   |  { calldata: claimSBT(jwt) } |                            |
   | ----------------------------> |                            |
   |                              |  Paymaster placa gas        |
   |                              | --------------------------> |
   |                              |                            |  SBT mintan
   |                              |  <-- tx receipt ---------- |
   |  <-- potvrda -------------- |                            |
```

**Ogranicenja Paymastera:**
- Paymaster mora imati whitelist ili rate-limit per adresa (1 gasless SBT mint po adresi).
- Bez limita, napadac moze slati jeftine UserOps i drainati Paymaster fond.
- Budzet Paymastera je operativni trosak platforme — financira se iz Platform Safe-a.

---

## 5. Analiza prijetnji (Threat Model)

### 5.1. Matrica napada

| # | Napad | Vektor | Mitigacija | Rezidualni rizik |
|---|-------|--------|------------|------------------|
| T1 | Kompromitiran Node.js server | Napadac preuzme server | Server nema Web3 kljuceve, ne moze falsificirati AKD potpis. Moze jedino prestati prosljedivati JWT-ove (DoS). | Nizak. DoS na auth, ali sredstva i identiteti sigurni. |
| T2 | Nonce swap (Server podmece tudu adresu) | Zlonamjerni ili hakiran server | Klijent u koraku 4 parsira auth URL i verificira da je nonce == vlastita adresa. Odbija redirect ako se ne poklapa. | Nizak. Ovisi o ispravnoj implementaciji klijenta. |
| T3 | JWT replay (isti JWT koristen dvaput) | Napadac presretne JWT | Pametni ugovor vodi mapu `usedJWTs[hash]`. Drugi poziv s istim JWT-om revertira. Dodatno: JWT ima `exp` claim. | Vrlo nizak. |
| T4 | Sybil napad (jedna osoba, vise SBT-ova) | Korisnik koristi vise wallet-a | Pametni ugovor mora iz JWT-a izvuci jedinstveni identifikator (sub claim ili hash OIB-a) i voditi mapu `identityUsed[hash]`. Drugi pokusaj revertira. | Nizak. Ovisi o jedinstvenosti identifikatora u JWT-u. |
| T5 | Certilia key compromise | AKD privatni kljuc procurjeli | `updateCertiliaPublicKey()` — admin (Safe multisig) rotira kljuc. Stari SBT-ovi se mogu bulk-revokirati. | Srednji. Prozor izlozenosti do detekcije. |
| T6 | Paymaster draining | Spam UserOperations | Rate limit: max 1 gasless mint po adresi. Paymaster budget cap. | Nizak. Napadac moze potrositi budzet, ali ne moze ukrasti sredstva. |
| T7 | Smart contract bug | Greska u JWT parsiranju / SBT logici | Formalna verifikacija kriticnih dijelova. Profesionalni audit prije produkcije. | Srednji. Ovisi o kvaliteti audita. |
| T8 | Izgubljen mobitel / istekla iskaznica | Korisnik gubi pristup Certilia identitetu | `revokeSBT(address)` — poziva Safe multisig. Korisnik moze re-mintati s novim wallet-om nakon ponovne Certilia verifikacije. | Nizak. |
| T9 | Frontend supply-chain napad | Zlonamjerni npm paket, CDN hijack | Korisnik verificira nonce (korak 4). Cak i zlonamjerni frontend ne moze ukrasti sredstva bez privatnog kljuca. SRI hashevi na CDN resursima. | Srednji. Frontend moze prikazati lazne podatke ali ne moze izvuci kljuceve iz hardware wallet-a. |

### 5.2. Detaljne analiza kriticnih napada

#### T2: Nonce Swap — detaljno

**Scenarij:** Napadac kontrolira server. Kada korisnik 0xAlice zatrazi auth, server generira URL s `nonce=0xEve` umjesto `nonce=0xAlice`. Ako Alice ne provjeri, Certilia ce izdati JWT koji vezuje Alicin identitet za Evin wallet.

**Mitigacija (korak 4 iz ADR-002):**

```
// Frontend pseudokod
const authUrl = await fetch('/auth/certilia', { body: { walletAddress: myAddress } })
const parsedUrl = new URL(authUrl)
const nonceInUrl = parsedUrl.searchParams.get('nonce')

if (nonceInUrl.toLowerCase() !== myAddress.toLowerCase()) {
  throw new Error('SECURITY: Server returned wrong nonce. Aborting.')
}

// Tek sada redirectaj korisnika
window.location.href = authUrl
```

**Preostali rizik:** Ova obrana ovisi o integritetu frontenda. Ako je frontend kompromitiran (T9), napadac moze preskoeciti provjeru. Zato je hardware wallet + korisnicka edukacija zadnja linija obrane.

#### T4: Sybil napad — detaljno

**Scenarij:** Osoba s jednim OIB-om generira 10 wallet-ova i pokusava mintati 10 SBT-ova za 10 glasova.

**Mitigacija:**

```solidity
// Pseudokod pametnog ugovora
mapping(bytes32 => bool) public identityClaimed;

function claimCertiliaSBT(bytes calldata jwt) external {
    // ... JWT verifikacija ...

    bytes32 identityHash = keccak256(abi.encodePacked(jwt.sub));

    require(!identityClaimed[identityHash], "Identity already claimed");
    identityClaimed[identityHash] = true;

    _mint(msg.sender, tokenId);
}
```

**Otvoreno pitanje:** Sto je `sub` claim u Certilia JWT-u? Ako je to OIB hash, dobro. Ako je session ID, ne radi. Trebamo Certilia dokumentaciju.

#### T5: Certilia Key Rotation — detaljno

**Scenarij:** AKD objavi da rotira svoj OIDC signing key (standardna praksa ili post-breach).

**Tok:**

```
AKD objavljuje novi JWKS
        |
        v
Safe multisig (admin) primijeti
        |
        v
Safe pokrece tx: CertiliaSBT.updatePublicKey(newKey)
        |
        v
Pametni ugovor od sada verificira JWT-ove s novim kljucem.
Stari SBT-ovi ostaju vazeci (vec su verificirani).
```

**Kriticno:** `updatePublicKey` MORA biti `onlyAdmin` gdje je admin = Safe adresa. Nikakav EOA, nikakav pojedinacni kljuc.

---

## 6. Arhitektura pametnih ugovora

### 6.1. CertiliaSBT.sol — Specifikacija

```
CertiliaSBT (ERC-721 + Soulbound ekstenzija)
|
|-- Storage
|   |-- certiliaPublicKey: bytes        // AKD OIDC javni kljuc
|   |-- admin: address                  // Safe multisig adresa
|   |-- identityClaimed: mapping(bytes32 => bool)
|   |-- usedJWTs: mapping(bytes32 => bool)
|   |-- revokedTokens: mapping(uint256 => bool)
|
|-- Funkcije
|   |-- claimCertiliaSBT(bytes jwt) external
|   |   |-- Dekodira JWT header + payload + signature
|   |   |-- Verificira potpis koristeci certiliaPublicKey
|   |   |-- Provjerava: jwt.nonce == msg.sender
|   |   |-- Provjerava: jwt.exp > block.timestamp
|   |   |-- Provjerava: !usedJWTs[hash(jwt)]
|   |   |-- Provjerava: !identityClaimed[hash(jwt.sub)]
|   |   |-- Minta SBT na msg.sender
|   |
|   |-- revokeSBT(uint256 tokenId) external onlyAdmin
|   |-- updatePublicKey(bytes newKey) external onlyAdmin
|   |-- isVerified(address wallet) external view returns (bool)
|   |
|   |-- transferFrom() --> REVERT (Soulbound - neprenosiv)
|   |-- approve()      --> REVERT (Soulbound - neprenosiv)
```

### 6.2. Safe integracija

```
Safe Treasury (Platform)
|
|-- Zodiac Role Module
|   |-- Role: VERIFIED_CREATOR
|   |-- Uvjet: CertiliaSBT.isVerified(caller) == true
|   |-- Dozvole: Moze inicirati isplatu do X EURe
|
|-- Zodiac Guard (opcija)
|   |-- Pre-transaction hook
|   |-- Provjerava da primatelj isplate ima validan SBT
```

### 6.3. Payment Splitter

```
PaymentSplitter (per kreator ili 0xSplits v2)
|
|-- Konfiguracija
|   |-- recipients: [creatorAddress, platformSafe]
|   |-- shares: [90, 10]  // bips: [9000, 1000]
|
|-- Funkcije
|   |-- split(uint256 amount) external
|   |   |-- EURe.transferFrom(msg.sender, creator, amount * 90 / 100)
|   |   |-- EURe.transferFrom(msg.sender, platformSafe, amount * 10 / 100)
```

---

## 7. Nerijesena pitanja i sljedeci koraci

### Kriticna (blokiraju implementaciju)

| # | Pitanje | Kontekst |
|---|---------|----------|
| Q1 | Koji algoritam koristi Certilia za JWT potpis? RSA-2048, RSA-4096, P-256, ili nesto drugo? | Odreduje koji precompile/biblioteku trebamo na Gnosisu. EIP-7212 pokriva P-256 ali ne RSA. Za RSA trebamo custom implementaciju ili cekati precompile. |
| Q2 | Sto je `sub` claim u Certilia JWT-u? | Mora biti deterministicki jedinstven po gradaninu (hash OIB-a ili ekvivalent). Ako je session-based, Sybil zastita ne radi. |
| Q3 | Podrzava li Certilia custom `nonce` parametar u OIDC zahtjevu? | Cijela arhitektura ovisi o mogucnosti slanja wallet adrese kao nonce. Ako ne podrzava, trebamo alternativni binding mehanizam. |
| Q4 | Postoji li Certilia sandbox/test okruzenje? | Bez test okruzenja ne mozemo razvijati bez koristenja pravih identiteta. |

### Vazna (utjecu na dizajn)

| # | Pitanje | Kontekst |
|---|---------|----------|
| Q5 | Koji je najjeftiniji nacin za on-chain RSA verifikaciju na Gnosisu? | Opcije: (a) custom Solidity RSA lib (npr. adria0/SolRSA), (b) cekati native precompile, (c) ZK proof off-chain + on-chain verificer. |
| Q6 | 0xSplits v2 ili custom splitter? | 0xSplits je auditan i dokazan, ali dodaje ovisnost. Custom daje potpunu kontrolu. |
| Q7 | Kako rijesiti JWT expiry UX problem? | JWT zivi ~5min. Korisnik mora imati wallet spreman i dovoljno brzo potpisati tx. Trebamo UX tok koji minimizira frikciju. |
| Q8 | ERC-4337 vs Gelato Relay za gasless? | ERC-4337 je standard ali kompleksniji. Gelato je jednostavniji ali centraliziraniji. |

### Istraživacka (dugorocno)

| # | Pitanje | Kontekst |
|---|---------|----------|
| Q9 | ZK-SNARK za privatnost glasanja? | Trenutno SBT je javno vidljiv. Za glasanje mozda trebamo ZK dokaz da osoba IMA SBT bez otkrivanja KOJI SBT/wallet. |
| Q10 | Multi-country eIDAS podrska? | Certilia je HR-only. eIDAS standard pokriva cijelu EU. Mozemo li generalizirati na druge drzave? |
| Q11 | SBT obnova vs trajnost? | Treba li SBT imati expiry (npr. poklapa se s rokom osobne iskaznice) ili je trajan dok se ne opozove? |

---

## Dodatak: Usporedba arhitektura

```
                  God-Mode Backend          OIDC + Self-Mint SBT
                  (ODBACENO)                (ODABRANO)

Server hack =     Totalni kompromis         DoS na auth (sredstva sigurna)
Fake identitet =  Server moze mintati       Nemoguce bez AKD potpisa
User custody =    Server drzi kljuceve      Korisnik drzi kljuceve
Auditabilnost =   "Vjeruj serveru"          Matematicki dokaz on-chain
Gas trosak =      Nizak (obican mint)       Visok (JWT verif.) — ali Gnosis!
UX =              Jednostavno (Web2)        Kompleksnije (treba wallet)
Regulatorni =     Custodian (MiCA risk)     Non-custodial
```
