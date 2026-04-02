# Deep Research: eIDAS 2.0 i EUDI novčanik — vremenski okvir i tehničke specifikacije

## Kontekst za istraživača

Gradim sustav koji danas koristi Certilia Mobile.id (hrvatski eIDAS 1.0 OIDC pružatelj) za verifikaciju identiteta. Europska komisija uvodi eIDAS 2.0 regulativu s EUDI novčanikom (European Digital Identity Wallet) koji bi mogao zamijeniti ili nadopuniti Certiliju. Trebam razumjeti vremenski okvir i tehničke specifikacije kako bih dizajnirao sustav koji je kompatibilan s budućnošću.

## Pitanja na koja trebam odgovore

### 1. Regulatorni vremenski okvir

- Kada je eIDAS 2.0 regulativa (Regulation (EU) 2024/1183) stupila na snagu?
- Koji su ključni rokovi za implementaciju?
- Do kada svaka država članica mora ponuditi EUDI novčanik?
- Gdje je Hrvatska u procesu implementacije?
- Tko je u Hrvatskoj zadužen za implementaciju EUDI novčanika? (FINA? AKD? Ministarstvo?)

### 2. EUDI novčanik — tehničke specifikacije

- Koji je format tvrdnji (credentials) u EUDI novčaniku?
  - W3C Verifiable Credentials? Ako da, koji profil (JWT-VC, JSON-LD, SD-JWT)?
  - ISO/IEC 18013-5 (mDL)?
  - Ili nešto treće?
- Koji kriptografski algoritmi su specificirani? (RSA, P-256, Ed25519?)
- Podržava li EUDI novčanik selektivno otkrivanje (selective disclosure)?
- Podržava li ZK dokaze?
- Koji je protokol za prezentaciju tvrdnji? (OpenID4VP? ISO 18013-7?)
- Postoji li specifikacija za lančanu (blockchain) verifikaciju EUDI tvrdnji?

### 3. Architecture Reference Framework (ARF)

- Koji je najnoviji ARF dokument? (verzija, datum)
- Što ARF kaže o:
  - Formatu identifikacijskih atributa (PID — Person Identification Data)?
  - Jedinstvenom identifikatoru osobe u EUDI sustavu?
  - Mogućnosti korištenja EUDI novčanika za potpisivanje transakcija?
  - Interakciji s blockchainom ili DLT-om?

### 4. Pilot projekti

- Koji su EU Large Scale Pilots (LSP) za EUDI? (POTENTIAL, EWC, NOBID, DC4EU)
- Koji od njih uključuju Hrvatsku?
- Postoje li pilot projekti koji kombiniraju EUDI s blockchainom ili Web3?
- Postoji li javno dostupan kod ili SDK za EUDI integraciju?

### 5. Utjecaj na Certiliju

- Hoće li Certilia biti zamijenjena EUDI novčanikom ili će koegzistirati?
- Hoće li EUDI novčanik koristiti isti OIDC protokol kao Certilia ili nešto drugačije (OpenID4VP)?
- Ako EUDI koristi OpenID4VP umjesto klasičnog OIDC, koliko se naš tok mora promijeniti?

### 6. Kompatibilnost s našim sustavom

- Može li se EUDI Verifiable Credential verificirati na blockchainu? Ako da, koji algoritam potpisa se koristi?
- SD-JWT (Selective Disclosure JWT) — može li se koristiti za slanje minimalnih tvrdnji pametnom ugovoru?
- Postoji li standardizirani "nullifier" ili "unique identifier" u EUDI PID-u koji bi služio kao Sybil zaštita?

## Format odgovora

1. **Vremenski okvir** — tablica s ključnim datumima
2. **Tehničke specifikacije** — tablica protokola i formata
3. **Utjecaj na naš projekt** — konkretne preporuke
4. **Izvori** — službeni EU dokumenti, ARF verzije, LSP stranice

Sav izlaz na **hrvatskom jeziku**.
