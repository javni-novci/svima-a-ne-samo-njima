# Deep Research: Lančana RSA/P-256 verifikacija na Gnosis lancu

## Kontekst za istraživača

Gradim pametni ugovor na Gnosis lancu (bivši xDAI) koji mora **verificirati RSA ili P-256 digitalni potpis unutar EVM-a**. Konkretno, trebam verificirati JWT potpis koji dolazi od hrvatskog eIDAS pružatelja (Certilia). JWT je potpisan RSA-256 ili P-256 algoritmom (još ne znamo točno koji).

Ovo je računski skupa operacija — RSA-2048 verifikacija troši ~2-3M goriva na EVM-u. Na Ethereum glavnoj mreži to košta $50-200, ali na Gnosisu bi trebalo biti $0.01-0.05. Trebam provjeriti stvarno stanje.

## Pitanja na koja trebam odgovore

### 1. RSA verifikacija u Solidity-u

- Koje Solidity biblioteke za RSA verifikaciju trenutno postoje? Za svaku navedi:
  - GitHub URL
  - Je li revidirana (auditirana)? Tko ju je revidirao?
  - Koji RSA algoritmi su podržani? (PKCS#1 v1.5, PSS, RSA-SHA256?)
  - Koliko goriva troši RSA-2048 verifikacija? RSA-4096?
  - Je li deployrana na Gnosis lancu?
  - Zadnji commit / aktivnost projekta?
- Konkretno provjeri ove projekte:
  - `adria0/SolRSA`
  - `solidity-rsa` ili slični
  - Google-ova `android-key-attestation` Solidity implementacija
  - Bilo što iz Ethereum Attestation Service (EAS) ekosustava

### 2. P-256 (secp256r1) verifikacija u Solidity-u

- EIP-7212: Koji je status? Je li implementiran na Gnosisu?
- Daisy Chain ili drugi P-256 precompile-ovi — postoje li na Gnosisu?
- Koje Solidity biblioteke za P-256 postoje bez precompile-a?
  - `dcposch/p256-verifier`
  - `freshcryptolib` (FCL)
  - `openzeppelin` — imaju li P-256 podršku?
  - Troškovi goriva za svaku?
- Usporedba: P-256 softverska verifikacija vs. RSA softverska verifikacija — što je jeftinije na EVM-u?

### 3. JWT parsiranje u Solidity-u

- Postoje li Solidity biblioteke za parsiranje JWT-a (Base64 dekodiranje, JSON parsiranje)?
- Kako se JWT struktura (header.payload.signature) obrađuje u Solidity-u?
- Koliko goriva troši samo Base64 dekodiranje i JSON parsiranje (bez kriptografske verifikacije)?
- Postoji li alternativa: verificirati potpis nad sirovim base64-kodiranim podacima bez dekodiranja sadržaja?

### 4. Gnosis lanac specifičnosti

- Koja je trenutna cijena goriva na Gnosis lancu? (gwei)
- Koliko košta transakcija od 2M goriva na Gnosisu u USD?
- Koliko košta transakcija od 5M goriva na Gnosisu u USD?
- Ima li Gnosis lanac neke EVM precompile-ove koje Ethereum nema?
- Koji je maksimalni gas limit po bloku na Gnosisu?
- Postoji li rizik da transakcija od 2-5M goriva bude prevelika za jedan blok?

### 5. ZK alternativa

- Ako je direktna lančana verifikacija preskupa, postoji li izvediv ZK pristup?
- Može li se RSA-2048 verifikacija staviti u ZK krug (circom, halo2)?
  - Ako da, koliko constraintova ima takav krug?
  - Koliko vremena traje generiranje dokaza na mobitelu?
- Može li se P-256 verifikacija staviti u ZK krug?
  - `circom-ecdsa` — podržava li P-256 (secp256r1) ili samo secp256k1?
  - Troškovi goriva za on-chain verifikaciju Groth16 dokaza na Gnosisu?
- Semaphore na Gnosisu — je li deployran? Koja verzija?

### 6. Postojeći projekti koji rade sličnu stvar

- Postoje li projekti koji već rade lančanu JWT verifikaciju? Na kojem lancu?
- Postoje li projekti koji verificiraju eIDAS potpise na blockchainu?
- zkEmail — koristi li RSA verifikaciju? Na koji način?
- Automata Network DCAP attestation — relevantno?

## Format odgovora

Za svaku biblioteku/alat navedi:

| Biblioteka | Algoritam | Gorivo (gas) | Revidirana? | Gnosis deployment | Aktivna? | URL |
|------------|-----------|-------------|-------------|-------------------|----------|-----|

Sav izlaz na **hrvatskom jeziku**.
