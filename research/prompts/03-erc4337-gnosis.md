# Deep Research: ERC-4337 apstrakcija računa na Gnosis lancu

## Kontekst za istraživača

Gradim aplikaciju na Gnosis lancu gdje novi korisnici trebaju pozvati pametni ugovor (`claimCertiliaSBT()`) ali nemaju xDAI za gorivo jer su tek napravili novčanik. Trebam Paymaster koji će sponzorirati tu prvu transakciju.

Zahtjevi:
- Gnosis lanac (chain ID 100) — NE Gnosis Chiado testnet, NE Ethereum
- Sponzorirano kovanje tokena (jedna transakcija po korisniku)
- Ograničenje: maksimalno 1 sponzorirana transakcija po adresi/identitetu
- Transakcija troši ~2-5M goriva (kriptografska verifikacija)

## Pitanja na koja trebam odgovore

### 1. ERC-4337 infrastruktura na Gnosisu

- Koji EntryPoint ugovor je deployran na Gnosis lancu? Koja verzija (v0.6, v0.7)?
- Adresa EntryPoint ugovora na Gnosisu?
- Koji Bundleri su aktivni na Gnosisu? Za svaki:
  - Naziv i pružatelj
  - RPC URL ili način pristupa
  - Podržavaju li v0.6 ili v0.7?
  - Ograničenja (rate limit, maksimalni gas po UserOp)?

### 2. Paymaster pružatelji na Gnosisu

Za svaki od sljedećih pružatelja, provjeri jesu li aktivni na Gnosis lancu:

**Pimlico:**
- Podržava li Gnosis lanac?
- Koji tipovi Paymastera? (Verifying Paymaster, ERC-20 Paymaster?)
- Cijena / besplatni tier?
- API dokumentacija za Gnosis?

**Biconomy:**
- Podržava li Gnosis lanac?
- Koji tipovi Paymastera?
- Cijena / besplatni tier?
- Sponsorship Paymaster — može li se konfigurirati "samo za ovu funkciju na ovom ugovoru"?

**Gelato Relay:**
- Podržava li Gnosis lanac?
- Je li to ERC-4337 kompatibilno ili vlastiti relay sustav?
- Cijena / besplatni tier?
- Relay vs. Paymaster — koja je razlika u njihovom sustavu?

**Alchemy (Account Kit):**
- Podržava li Gnosis lanac?

**StackUp:**
- Podržava li Gnosis lanac?

**Ostali:**
- Postoje li Paymaster pružatelji specifični za Gnosis lanac?
- Postoji li Gnosis-ov vlastiti Paymaster ili sponzorski program?

### 3. Vlastiti Paymaster

- Ako nijedan komercijalni pružatelj ne podržava Gnosis, koliko je složeno deployati vlastiti Paymaster?
- Postoje li otvorene implementacije Verifying Paymastera?
  - OpenZeppelin ima li Paymaster ugovore?
  - eth-infinitism/account-abstraction — referentna implementacija?
  - Kako se implementira pravilo "samo 1 sponzorirana transakcija po adresi"?
- Koliko xDAI trebam držati u Paymaster ugovoru za N korisnika?

### 4. Smart Account tvornice na Gnosisu

- Safe{Core} Protocol (Safe AA) — je li aktivan na Gnosisu?
- Kernel (ZeroDev) — Gnosis podrška?
- SimpleAccount (referentna implementacija) — deployrana na Gnosisu?
- Trebam li Smart Account ili može obični EOA s Paymaster sponzorstvom?

### 5. Alternativa ERC-4337: Meta-transakcije

- Ako ERC-4337 nije zreo na Gnosisu, postoje li alternative?
- EIP-2771 (meta-transakcije s Forwarder ugovorom) — jednostavnije od ERC-4337?
- OpenGSN (Gas Station Network) — podržava li Gnosis?
- Vlastiti relayer koji samo prosljeđuje potpisane poruke?
- Usporedba složenosti: ERC-4337 vs EIP-2771 vs vlastiti relayer

### 6. Troškovi

- Koliko goriva troši ERC-4337 UserOperation overhead (iznad same transakcije)?
- Ako moja transakcija troši 3M goriva, koliko ukupno troši kao UserOp?
- Koliko to košta u xDAI na Gnosisu?
- Koliko to košta za 1.000 korisnika? 10.000 korisnika?

## Format odgovora

Za svaki Paymaster pružatelj:

| Pružatelj | Gnosis podrška | ERC-4337 verzija | Cijena | Ograničenja | URL dokumentacije |
|-----------|---------------|------------------|--------|-------------|-------------------|

Sav izlaz na **hrvatskom jeziku**.
