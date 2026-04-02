# Lančana RSA/P-256 verifikacija — Perplexity izvještaj

**Prompt:** [02-lancana-kriptografija-gnosis.md](../prompts/02-lancana-kriptografija-gnosis.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

Na Gnosis lancu je potpuno izvedivo on‑chain verificirati JWT potpisan RSA‑2048 ili P‑256, ali softverski RSA‑2048 tipično troši oko 1.5–2.5M gas po verifikaciji, dok dobro optimizirana P‑256 verifikacija pada na 70–300k gas, pa je P‑256 osjetno jeftiniji. Na tipičnim cijenama plina na Gnosisu to znači red veličine milicenta po P‑256 verifikaciji i nekoliko milicentâ po RSA‑2048 verifikaciji, pa je glavni trade‑off više kompleksnost implementacije nego apsolutna novčana cijena.[1][2][3][4][5]

U nastavku ti dajem pregled konkretnih biblioteka, gas profila, ZK alternative i situacije na Gnosis lancu.

***

## 1. RSA verifikacija u Solidity‑ju

### 1.1 Ključne biblioteke

#### adria0/SolRsaVerify

SolRsaVerify je izvorna, vrlo korištena Solidity implementacija RSA‑SHA256 PKCS#1 v1.5 verifikacije (signatura + SHA‑256 + PKCS#1 v1.5 padding). OpenZeppelin 5.x RSA library eksplicitno navodi da je “inspirirana radom Adrià Massaneta”, tj. SolRsaVerify, ali s dodatnim sigurnosnim provjerama (min. 2048‑bit ključevi, provjere paddinga).[6][7]

- Nije javno objavljen formalni audit samog SolRsaVerify repozitorija; autor sada i u README‑u preporučuje korištenje OpenZeppelin RSA implementacije jer je dio šire, dobro revidirane codebase‑a.[7][6]
- Repozitorij je arhiviran u srpnju 2025. i više nije aktivno održavan.[8][9]

Nisam našao direktan, službeno objavljen gas benchmark za SolRsaVerify, ali imamo relativno svjež “state of the art” benchmark za RSA‑2048 verifikaciju preko modexp precompile‑a, koji je računskom jezgrom praktički isti problem.

#### Ostale RSA biblioteke i uzorci

- **axic/ethereum‑rsa** – rani eksperimentalni RSA verifikator, spominje se u EIP‑74 thread‑u kao uzorak implementacije; koristi vlastitu BigInt biblioteku preko modexp precompile‑a.[10][11]
- **OpenZeppelin Contracts 5.x – RSA** – generička RSA PKCS#1 v1.5 verifikacija (RSA.pkcs1Sha256), dizajnirana kao sigurniji nasljednik SolRsaVerify s eksplicitnim ograničenjem na ≥ 2048‑bitne ključeve.[7]
- **solidity‑google‑auth (SocialLock)** – kompletan primjer Google JWT verifikacije na lancu; koristi RSA‑256 (RSASSA‑PKCS1v1.5 SHA‑256) nad Google JWKS ključevima dohvaćenima Chainlink oracleom.[12][13]

### 1.2 Gas profil RSA‑2048 / RSA‑4096

RareSkills je mjerio gas trošak RSA verifikacije koristeći EIP‑198 modexp precompile (adresa 0x05) na Ethereum mainnetu, što se izravno prenosi i na Gnosis jer koristi isti precompile i istu tarifu po EIP‑198/EIP‑2565.[2][14]

- **RSA‑2048 verifikacija**:
  - Prosječan trošak verifikacije: oko 1.49M gas po potpisu.[2]
  - Najgori slučaj (ovisno o duljini eksponenta i konfiguraciji): do ~2.42M gas.[2]
- **RSA‑4096 verifikacija**:
  - Trošak raste otprilike kvadratno u duljini modula; u istom benchmarku RSA‑4096 verifikacije su prelazile 10M gas.[2]
- Ti brojevi su za “golu” modexp‑baziranu verifikaciju; integracija u JWT/attestation tok dodaje još malo overheada, ali red veličine je isti.

SocialLock, koji radi kompletnu Google JWT verifikaciju (parsiranje JWT‑a, provjera audience/nonce te RSA‑256 signatura nad SHA‑256 hešom), prijavljuje ukupnu potrošnju oko **1,78M gas** za jednu `withdraw` transakciju na Ethereum testnetu, što je u skladu s RareSkills mjerenjem.[12]

### 1.3 Tablica – RSA biblioteke / rješenja

| Biblioteka / projekt | Algoritam | Gorivo (gas) | Revidirana? | Gnosis deployment | Aktivna? | URL |
|----------------------|-----------|--------------|-------------|-------------------|----------|-----|
| **adria0/SolRsaVerify** | RSA‑SHA256, PKCS#1 v1.5 (verifikacija potpisa nad SHA‑256 digestom) [6] | Nema objavljenih službenih brojki; očekuj ~1.5–2.5M gas za RSA‑2048 preko modexp precompile‑a, više za 4096‑bit [2][14] | Nema javnog audita; koristi se kao referenca, ali autor upućuje na OZ RSA [6][7] | Nije poznat standardni deployment; tipično se ugrađuje u vlastite ugovore | Arhivirano u 07/2025, praktički neaktivno [8][9] | https://github.com/adria0/SolRsaVerify |
| **OpenZeppelin RSA (Contracts 5.x)** | RSA PKCS#1 v1.5, SHA‑256 (RSA.pkcs1Sha256 nad digestom ili nad raw data) [7] | OZ ne navodi egzaktne brojke; gas je istog reda kao i SolRsaVerify (modexp‑bazirano, ~1.5–2.5M za RSA‑2048) [2][14] | Dio OpenZeppelin Contracts 5.x, višestruko auditiran framework (iako RSA modul nema poseban izdvojeni audit) [7] | Radi na svim EVM chainovima s modexp precompile‑om (uklj. Gnosis); deployment je na tebi | Aktivno održavano (5.x linija) [7] | https://docs.openzeppelin.com/contracts/5.x/api/utils/cryptography#RSA |
| **axic/ethereum‑rsa** | RSA‑SHA256, PKCS#1 v1.5, custom BigInt + modexp [10][11] | Stara implementacija, bez novijih gas mjerenja; očekuj sličan ili lošiji gas od novijih libova (~2M+ gas za RSA‑2048) [2] | Nema poznatog audita; više kao referentni proof‑of‑concept [10] | Nije poznato da je standardno deployan na Gnosisu | Repozitorij star, bez skorašnje aktivnosti | https://github.com/axic/ethereum-rsa |
| **SocialLock (solidity‑google‑auth)** | Google JWT RSA‑256 (RSA‑SHA256 PKCS#1 v1.5) + provjera audience/nonce [12][13] | Cijela `withdraw` transakcija (parsiranje JWT‑a + RSA verifikacija) ≈ 1,780,178 gas [12] | Projekt je POC, nema formalni audit | Deployan kao demo na Sepolia; nema javnog deploymenta na Gnosisu [13] | Repozitorij aktivan kao referenca, ali ne kao proizvodni servis [13] | https://github.com/lucashenning/solidity-google-auth |

***

## 2. P‑256 (secp256r1) verifikacija u Solidity‑ju

### 2.1 Status EIP‑7212 i Gnosis

EIP‑7212 predlaže novi precompile za secp256r1 (P‑256) verifikaciju s fiksnim troškom oko **3,450 gas** po verifikaciji, što je red veličine efikasnije od softverskih implementacija. Neke mreže (npr. Polygon) već su integrirale ovaj precompile ili ekvivalentnu RIP‑7212 varijantu, a OpenZeppelin P256 library ima `verify()` koje pokušava koristiti precompile ako je dostupan, s fallbackom na Solidity implementaciju.[15][16][1][7]

- Dokumentacija za Safe passkey module spominje da se mogu osloniti na EIP‑7212 precompile “na podržanim mrežama” ili na čisti Solidity verifikator kao fallback, ali Gnosis nije naveden kao mreža koja trenutno ima taj precompile.[17]
- Pixel (ERC‑4337 wallet za P‑256 passkeye) navodi da na Goerli, Gnosis Chain, Scroll i Polygon zkEVM koristi **čistu solidity P‑256 verifikaciju** koja troši oko **500k–1M gas** po verifikaciji, upravo zato što ti chainovi nemaju nativni P‑256 precompile.[18]

Na temelju toga, za travanj 2026. razumno je pretpostaviti da Gnosis još nema implementiran EIP‑7212/RIP‑7212 precompile, pa se na njemu koristi softverska P‑256 verifikacija (FCL/Obvious/OZ P256) ili ZK pristupi.[17][18]

### 2.2 Softverske P‑256 biblioteke i gas

HackMD “Current State of Verifying P256 Curve” sažima gas profile za nekoliko aktivnih implementacija P‑256 u Solidity‑ju, uključujući FCL (Ledger), Obvious i Alembic.[1]

- **Obvious Wallet (itsobvioustech – Secp256r1.sol)**  
  - P‑256 WebAuthn verifikacija: **~330k gas** po transakciji; deployment oko 590k gas.[1]
- **Alembic P256 Solidity verifier**  
  - Verifikacija WebAuthn potpisa: **~375k gas**; deployment oko 2M gas.[1]
- **FreshCryptoLib (FCL) – Ledger P‑256, bez prekomputacije**  
  - WebAuthn verifikacija: **~205k gas**; deployment oko 1M gas.[1]
- **FreshCryptoLib (FCL) – s prekomputacijom (16 kB storage)**  
  - Deployment: ~3.2M gas (zbog tablica u storageu).  
  - Verifikacija: **~69k gas** po potpisu.[1]
- **OpenZeppelin P256 library (5.x)**  
  - `P256.verify(h, r, s, qx, qy)` koristi precompile ako postoji, inače Solidity fallback; dokumentacija ne navodi egzaktne gas brojke, ali budući da je “heavily inspired” FCL/Obvious implementacijama, očekuje se isti red veličine (≈70–300k gas ovisno o konfiguraciji).[7][1]
- **dcposch/p256‑verifier (Daimo / base)**  
  - HackMD članci ga navode među ranim implementacijama, ali tablica optimizacija pokazuje da FCL s prekomputacijom trenutno drži najbolji trade‑off gas vs. deployment.[1]

### 2.3 ZK‑bazirani P‑256 pristupi

Isti HackMD dokument daje brojke za ZK‑bazirane verifikatore WebAuthn P‑256 potpisa:[1]

- **Risc0 Bonsai (STARK→SNARK)**  
  - On‑chain verificiranje Groth16 dokaza: **~220–280k gas**.[1]
- **Halo2 (Know Nothing Labs, WebAuthn wallet)**  
  - On‑chain verifikacija: **~500–520k gas**.  
  - Proving vrijeme: oko **4–20 s** na M1/M1 Pro ili čak mobitelu.[1]
- **Circom (zkwebauthn‑circom)**  
  - Fiksni dio: ~230k gas.  
  - Dodatno ~100k gas po potpisu (≈330k ukupno) za WebAuthn verifikaciju.[1]

### 2.4 Usporedba: P‑256 vs. RSA u EVM‑u

Na osnovi spomenutih mjerenja:

- **RSA‑2048 softverska verifikacija**: oko **1.5–2.5M gas**.[2]
- **P‑256 softverska verifikacija**: ovisno o biblioteci, **~70k–375k gas**.[1]
- **P‑256 preko EIP‑7212 precompile‑a**: **~3,450 gas**.[1]

Dakle, i bez precompile‑a, **P‑256 verifikacija je tipično 5–20× jeftinija u gasu od RSA‑2048** na EVM‑u. Ako bi Gnosis jednog dana implementirao EIP‑7212 ili EIP‑7951‑ekvivalent, ta bi se razlika povećala na gotovo **tri reda veličine**.[19][2][1]

### 2.5 Tablica – P‑256 biblioteke / pristupi

| Biblioteka / alat | Algoritam | Gorivo (gas) | Revidirana? | Gnosis deployment | Aktivna? | URL |
|-------------------|-----------|--------------|-------------|-------------------|----------|-----|
| **FCL P256 bez prekomputacije** | P‑256 (secp256r1) ECDSA / WebAuthn [1] | ~205k gas po potpisu; ~1M gas deployment [1] | FCL je istraživački projekt Ledger tima; nema javnog “big‑four” audita, ali se često koristi u AA walletima | Može se deployati na bilo koji EVM (uklj. Gnosis); nema “canonical” instancu | Aktivno održavano [1] | https://github.com/rdubois-crypto/FreshCryptoLib |
| **FCL P256 s prekomputacijom** | Isto, uz 16 kB prekomputiranih tablica [1] | ~69k gas po verifikaciji; ~3.2M gas deployment [1] | Isto kao gore | Nema javnog Gnosis deploya, ali nema tehničke prepreke | Aktivno [1] | https://github.com/rdubois-crypto/FreshCryptoLib |
| **Obvious Secp256r1.sol** | P‑256 WebAuthn verifikacija za AA wallet [1] | ~330k gas po verifikaciji; ~590k gas deployment [1] | Nema javnog audita; korišten u produkcijskim AA rješenjima Obvious Walleta | Pixel navodi da radi na Gnosisu putem istog patterna (čisti Solidity) [18] | Aktivno [1][18] | https://github.com/itsobvioustech/aa-passkeys-wallet |
| **Alembic P256 verifier** | P‑256 WebAuthn [1] | ~375k gas verifikacija; ~2M gas deployment [1] | Nema javnog audita; demo NFT mint na Mumbai [1] | Nije posebno naveden za Gnosis, ali je prenosiv | Aktivno [1] | https://github.com/alembic-tech/P256-verify-signature |
| **OpenZeppelin P256** | P‑256 ECDSA verifikacija (precompile + Solidity fallback) [7] | Bez precompile‑a očekuj ~70–300k gas (u rangu FCL/Obvious); s precompile‑om ~3,450 gas [1][7] | Dio OZ 5.x ekosustava (interno auditiran) [7] | Radi na Gnosisu; gdje nema RIP‑7212 precompile‑a koristi se Solidity fallback [7][18] | Aktivno [7] | https://docs.openzeppelin.com/contracts/5.x/api/utils/cryptography#P256 |
| **dcposch/p256‑verifier** | P‑256 WebAuthn (Daimo, Base/zk sync ekosustav) [1] | HackMD implicira viši gas od FCL‑a; bez konkretnih brojki [1] | Nema poznatog audita; istraživačko/prototipno | Prijenosno na Gnosis, ali nema javnog deploymenta | Repozitorij aktivan, ali ne “standard de facto” | (repo link u HackMD) [1] |

***

## 3. JWT parsiranje u Solidity‑ju

### 3.1 Postojeće biblioteke i patterni

Ne postoji “standardna” proizvodna JWT biblioteka za Solidity koja u jednom potezu radi Base64URL dekodiranje, JSON parsiranje i kripto verifikaciju, ali postoje sastavne kockice i referentni projekti:

- **solidity‑base64 (hir0min)** – Base64/Base64URL encoder/decoder za Solidity 0.8.x, korišten u SocialLock projektu za dekodiranje Google JWKS ključeva.[20][12]
- **OpenZeppelin Base64** – Base64 encoder (ne URL varijanta), korišten u mnogim projektima; nema gotov JWT helper, ali implementacija je vrlo slična。[21]
- **solidity‑jwt (spalladino/OZ, više nije službeno održavano)** – korišten kao inspiracija u SocialLocku; razbija JWT na header/payload/signature i pomaže oko Base64/JSON obrade.[13][12]
- **jsmnSol** – port C biblioteke jsmn u Solidity, za parsiranje *malih* JSON struktura on‑chain. Autor eksplicitno naglašava da je zbog cijene string manipulacije pogodna samo za male JSON‑ove.[22]
- **Solidity‑CBOR** – CBOR parser; nije izravno JWT‑related, ali se koristi za sličan “parsiraj strukturirani dokaz u ugovoru” use‑case.[23]

Najkonkretniji real‑world primjer on‑chain JWT verifikacije je već spomenuti **SocialLock**:

- Frontend šalje **već dekodirane JSON stringove** za header i payload, te raw bytes signature; ugovor zatim:
  - encode‑a header i payload nazad u Base64URL stringove u skladu s JWT standardom,
  - konkatenira `headerBase64 + "." + payloadBase64`,
  - uzima RSA modulus iz JWKS Oracle ugovora (dohvaćen preko Chainlinka),
  - poziva `pkcs1Sha256VerifyStr` (SolRsaVerify/OZ RSA) da provjeri RSA‑256 potpis nad SHA‑256 hešom te poruke.[12]
- Testovi u repozitoriju pokazuju da kompletna `withdraw` operacija (uključujući JWT validaciju) troši **~1,78M gas**.[12]

### 3.2 Gas za Base64 i JSON parsiranje

Direktne, izolirane gas brojke za samo Base64URL decode + JSON parse je teško naći, ali iz SocialLock testova možeš dobiti osjećaj:

- Cijeli tok:
  - provjera RSA‑256 potpisa (≈1.5–2M gas prema RareSkillsu),[2]
  - Base64URL operacije, parsiranje JSON‑a (header/payload),
  - provjere audience i nonce.  
  → **Ukupno ~1.78M gas** po `withdraw` transakciji.[12]
- Iz toga slijedi da Base64 + JSON + kontrolna logika dodaju **relativno mali overhead** preko čistog RSA modexp troška (red veličine stotine tisuća gasa, ne milijune).[12][2]

Za razliku od RSA, P‑256 tokovi (npr. FCL/WebAuthn) uglavnom nemaju kompleksan JSON on‑chain; većina klijenata radi WebAuthn parsing off‑chain, a on‑chain se provjerava samo “čist” P‑256 potpis nad canonicaliziranim hešom challenge‑a, što drži gas trošak bliže 70–300k gas.[1]

### 3.3 Može li se izbjeći dekodiranje JWT sadržaja?

Da. Standardni JWS oblik je `BASE64URL(header) || "." || BASE64URL(payload) || "." || BASE64URL(signature)`. Sam potpis je nad:

$$
\text{SignatureInput} = \text{BASE64URL(header)} || "." || \text{BASE64URL(payload)}
$$

Zato ugovor, u principu, ne mora uopće dekodirati JSON:

- može primiti već formatirani `headerBase64` i `payloadBase64`,
- lokalno izračunati `keccak256` ili SHA‑256 (ovisno o algoritmu),
- verificirati potpis nad tim raw bytes (RSA ili P‑256).

SocialLock zapravo ide obrnutim putem (prima JSON, pa na lancu radi Base64URL encode), ali ništa te ne sprječava da u svom dizajnu učiniš suprotno: parsiraš JWT off‑chain, provjeriš da je oblik ispravan i šalješ **samo**:

- `headerBase64` (string),
- `payloadBase64` (string),
- `signature` (bytes),
- plus izdvojene claimove koje želiš on‑chain (npr. hash OIB‑a, timestamp).

To značajno smanjuje on‑chain string obradu i pojednostavljuje kod.

### 3.4 Tablica – JWT‑related biblioteke / uzorci

| Biblioteka / projekt | Algoritam | Gorivo (gas) | Revidirana? | Gnosis deployment | Aktivna? | URL |
|----------------------|-----------|--------------|-------------|-------------------|----------|-----|
| **solidity‑base64 (hir0min)** | Base64 + Base64URL encode/decode za Solidity 0.8.x [20] | Nema službenih brojki; tipično stotine tisuća gasa za kodiranje većih stringova; u SocialLocku je dio ukupnih ~1.78M gas [12] | Nema javnog formalnog audita; male biblioteke ovog tipa se često revidiraju u sklopu većih audita | Može se deployati bilo gdje; koristi se indirektno u SocialLocku | Aktivno [20][12] | https://github.com/hir0min/solidity-base64 |
| **solidity‑jwt (OZ / spalladino)** | JWT pomoćne funkcije (split, Base64, JSON), uz RSA‑256 verifikaciju [12] | Nema javnih izoliranih gas brojki; SocialLock (koji ga koristi kao inspiraciju) troši ~1.78M gas za punu verifikaciju [12] | Stari POC iz OpenZeppelin ekosustava; više nije aktivno održavan kao core library | Nema poznatog Gnosis deploymenta | Neaktivno / POC | (link iz SocialLock kredita) [12] |
| **jsmnSol** | JSON parser za male JSON payloadove [22] | Ovisno o veličini JSON‑a; autor upozorava da je prikladno samo za male strukture; nema objavljenih brojki | Nije auditiran kao samostalni modul koliko je javno poznato | Teoretski prenosiv na Gnosis | Zadnji update 2017., praktički neaktivan [22] | https://github.com/chrisdotn/jsmnSol |
| **SocialLock (solidity‑google‑auth)** | Kompletna Google JWT verifikacija (RSA‑256 + Base64URL + JSON) [12][13] | ~1,780,178 gas po `withdraw` (uključuje sve korake) [12] | Nema formalnog audita; edukativni POC | Nema deploymenta na Gnosisu; samo Sepolia primjer [13] | Repozitorij aktivan, ali više kao referenca [13] | https://github.com/lucashenning/solidity-google-auth |

***

## 4. Gnosis lanac: plin, cijene i limiti

### 4.1 Trenutna cijena plina i cijena transakcija

Gnosis Chain (bivši xDai) koristi xDAI kao nativnu valutu, koja je vezana uz DAI i time ciljano drži vrijednost blizu 1 USD. Minimalni gas price je tipično **1 gwei**, a u praksi gas tracker pokazuje **≈1–2 gwei** za “slow/average/fast”.[3][4][5]

Cijena transakcije se računa kao:

$$
\text{cijena} = \text{gasUsed} \times \text{gasPrice} \times 10^{-9}\ \text{xDAI}
$$

Ako pretpostavimo **1 gwei** i **1 xDAI ≈ 1 USD**:[4][5][3]

- **2M gas @ 1 gwei**:  
  - 2,000,000 × 1e‑9 = 0.002 xDAI ≈ **0.002 USD**.
- **5M gas @ 1 gwei**:  
  - 5,000,000 × 1e‑9 = 0.005 xDAI ≈ **0.005 USD**.

Ako gas price skoči na 5 gwei, cijene su još uvijek niske:

- 2M gas: 0.01 xDAI ≈ 0.01 USD.
- 5M gas: 0.025 xDAI ≈ 0.025 USD.

Dakle, čak i **RSA‑2048 verifikacija od ~2M gas** koštat će cent ili manje u tipičnim uvjetima na Gnosisu, a P‑256 verifikacija (~70–300k gas) praktički je besplatna u apsolutnim iznosima.[5][3][4][2][1]

### 4.2 Precompile‑ovi i block gas limit

- Gnosis je EVM‑kompatibilan i koristi standardne Ethereum precompile‑ove na adresama 0x01–0x09, uključujući **modexp (0x05)** prema EIP‑198, što omogućuje efikasnu RSA modularnu eksponencijaciju.[14][24]
- Nema javno dokumentiranih dodatnih precompile‑ova specifičnih za P‑256 na Gnosisu (za razliku od npr. Arbitrum Stylus, Fuel ili Aztec koji uvode vlastite primitivne).[18][1]
- Block gas limit na Gnosisu je u praksi sličan Ethereum mainnetu (deseci milijuna gas po bloku); iako u ovoj sesiji nisam povukao točan broj, transakcija od **2–5M gas** je daleko ispod tipičnog per‑block limita i neće sama po sebi saturirati blok.  
  Jedini rizik je ako imaš vrlo kompleksan tok (više poziva, puno storage pisanja) – ali sama kripto verifikacija (čak i RSA‑4096) stane unutar jednoga bloka uz razumnu marginu.

***

## 5. ZK alternativa (RSA i P‑256)

### 5.1 P‑256 u ZK krugovima

Već spomenuti HackMD članak opisuje nekoliko P‑256 ZK proof sustava za WebAuthn:[1]

- **Risc0 Bonsai (STARK→SNARK)**  
  - On‑chain verifikacija dokaza: ≈ **220–280k gas** na EVM‑u.[1]
  - Proving se radi off‑chain u Risc0, a na lanac dolazi Groth16 dokaz.
- **Halo2 (Know Nothing Labs WebAuthn wallet)**  
  - On‑chain verifikacija: ≈ **500–520k gas**.[1]
  - Proving vrijeme: oko **4 s na M1 Pro**, do ~20 s na smartphoneu.[1]
- **Circom (zkwebauthn‑circom)**  
  - On‑chain: oko **330k gas** (230k fiksni dio + 100k po potpisu).[1]

Ti sustavi već danas pokazuju da je **“off‑chain P‑256 u ZK + on‑chain Groth16 verifikacija” apsolutno praktična**, i gas‑wise često nije skuplja od pure‑Solidity verifikatora (posebno ako koristiš skupe kurve ili dodatnu logiku).[1]

### 5.2 RSA‑2048 u ZK krugovima

Za RSA‑2048 situacija je teža:

- **Anastasia (Noir + UltraHonk)** pokazuje ZK provjeru X.509 certifikata (ECDSA i RSA) u ZK krugu, s on‑chain verifikatorom koji provjerava samo ZK dokaz, ne cijeli X.509 parsing/validaciju u EVM‑u.[25]
- Radovi oko zkEmail‑like sustava (koje ovdje nisam mogao detaljno pregledati) tipično koriste plonkovske/ultra‑plonk sheme (Halo2, UltraPlonk) s **desecima milijuna constraintova** za RSA‑2048 verifikaciju i parsing S/MIME/PKCS#7 struktura, što znači da je proving vrijeme na mobitelu sekunde do desetke sekundi, a on‑chain Groth16 verifikacija ostaje u rangu **~200–300k gas** (slično P‑256 ZK primjerima).[25][1]

To znači da je **ZK pristup za RSA‑2048 tehnički izvediv**, ali ćeš u praksi htjeti:

- ograničiti veličinu certifikacijskog lanca (npr. 2–3 certifikata kao u Anastasia demonstraciji),[25]
- parsirati ASN.1/X.509 i sve eIDAS‑specifične podatke off‑chain,  
- u ZK krug staviti samo:
  - hash(eIDAS artefakta / JWT‑a),
  - RSA‑2048 verifikaciju nad tim hashom,
  - eventualno nekoliko logičkih uvjeta (npr. da određeni OID postoji u certifikatu).

### 5.3 Trošak on‑chain verifikacije Groth16 dokaza

Za standardni Groth16 verifikator na EVM‑u (2 pairing‑a + par skalara) trošak je tipično **≈200–300k gas**, ovisno o implementaciji i optimizaciji; iste veličine su i brojke iz P‑256 ZK primjera (Risc0, Circom) jer se na kraju svode na isti tip verifikatora.[1]

Na Gnosisu bi to, uz 1 gwei, značilo oko:

- 300,000 × 1e‑9 = 0.0003 xDAI ≈ **0.0003 USD** po ZK verifikaciji.

Uspoređujući:

- **Direktni RSA‑2048 on‑chain**: ~1.5–2.5M gas, ≈0.0015–0.0025 USD.[3][4][2]
- **ZK dokaz RSA‑2048**: ~200–300k gas on‑chain, ali skuplje off‑chain (proving na mobitelu sekunde do desetke sekunda).

Ako ti je kritična **privatnost payload‑a** (npr. ne želiš otkriti cijeli eIDAS certifikat ili JWT payload on‑chain), ZK pristup je puno privlačniji, čak i ako je proving na klijentu teži.

### 5.4 Semaphore na Gnosisu

U ovoj sesiji nisam uspio pouzdano dohvatiti aktualni deployment status Semaphore protokola na Gnosis chainu (adrese/verzije), pa ne mogu tvrditi je li i koja verzija trenutno deployrana.  
Ako ti je bitno, preporuka je:

- provjeriti službeni Semaphore repo i dokumentaciju (često navode “supported networks”),
- ili na Gnosis Blockscoutu tražiti poznate Semaphore factory/adrese.

***

## 6. Postojeći projekti slični tvom use‑caseu

### 6.1 On‑chain JWT verifikacija

- **SocialLock (solidity‑google‑auth)**  
  - Ethereum smart contract koji omogućuje slanje ETH‑a na “Google račun” i povlačenje sredstava samo uz valjan Google JWT.[13][12]
  - JWT je potpisan RSA‑256 algoritmom, a ugovor:
    - preuzima RSA modulus iz Google JWKS API‑ja preko Chainlink orakla,
    - parsira header/payload (JSON) i provjerava audience/nonce,
    - radi RSA‑256 verifikaciju na lancu pomoću SolRsaVerify‑inspirirane funkcije.[13][12]
  - Potrošnja plina za jednu `withdraw` operaciju: **~1.78M gas**.[12]
- Ovo je gotovo identičan pattern onome što želiš raditi s eIDAS JWT‑om, samo s Googleom kao IdP‑om, i dobar je “gornji bound” gas troška za pure‑on‑chain JWT verifikaciju.

### 6.2 P‑256 / WebAuthn projekti na Gnosisu

- **Pixel** – ERC‑4337 AA wallet i rollup koji koristi P‑256 (secp256r1) ključeve koje generiraju HSM‑ovi na modernim telefonima; eksplicitno navodi da je deployan na **Goerli, Gnosis Chain, Scroll i Polygon zkEVM**.[18]
  - Na običnim EVM chainovima koristi **čistu Solidity P‑256 verifikaciju**, s gas troškom **~500k–1M gas** po transakciji.[18]
  - Za smanjenje troška nudi vlastiti optimistic rollup s P‑256 precompile‑om.[18]
- Ovaj projekt ti je dobar referentni primjer kako integrirati P‑256 (passkey / WebAuthn) na Gnosisu bez ikakvog precompile‑a.

### 6.3 eIDAS i X.509 na blockchainu

- **Anastasia: Cinderella's Stepsister Turning Shabby X.509 Certificates into ZK‑Friendly Proofs**  
  - Predlaže korištenje ZK‑SNARK‑ova (UltraHonk/Noir) za provjeru X.509 certifikata i Android Key Attestation lanaca, uz on‑chain verifikator; izričito navodi da je “Solidity verifier gas‑heavy” za punu X.509 validaciju, pa je ZK pristup odabran zbog manjih dokaza i brže on‑chain verifikacije.[25]
- **Automata On‑Chain PCCS + DCAP Attestation**  
  - Implementira Solidity verifikatore za Intel SGX/TDX DCAP citate, uključujući parsiranje JSON/DER struktura i CRL‑ova, te opcionalnu ZK varijantu preko Risc0/SP1.[26][27][28]
  - Iako nije eIDAS per se, arhitektura (hardverska atestacija + ZK + on‑chain verifikator) je vrlo slična onome što bi trebao eIDAS‑kompatibilni sustav.

Nisam našao jasan primjer *proizvodnog* EVM projekta koji direktno provjerava eIDAS potpise (CAdES/XAdES) na lancu; većina radova su istraživački i naginju ZK pristupima zbog složenosti PKCS#7/XMLDSig/X.509 stoga.[25]

### 6.4 zkEmail i srodni projekti

U ovoj sesiji nisam uspio dohvatiti konkretnu dokumentaciju za zkEmail implementacije, pa ne mogu navesti egzaktne constraint brojeve ili gas troškove.  
Općenito, u literaturi se opisuje pattern:

- parser e‑maila i S/MIME/PGP potpisa u ZK krugu,
- RSA/ECDSA verifikacija unutar tog kruga,
- on‑chain Groth16 ili plonkish verifikator (≈200–300k gas po dokazu).[1]

### 6.5 Automata Network DCAP attestation

Automata DCAP Attestation i On‑Chain PCCS pružaju **Solidity implementaciju kompletne DCAP verifikacije** (SGX/TDX quotes, certifikati, CRL‑ovi), s mogućnošću on‑chain ili ZK‑bazirane provjere i gotovim helperima za više mreža.[27][28][26]
Relevantno ti je kao referenca za:

- kako strukturirati kompleksne ASN.1/DER/JSON dokaze,
- kako kombinirati **pure‑Solidity parsiranje** s opcionalnim **ZK co‑procesorom** radi smanjenja on‑chain troška i povećanja privatnosti.

***

## 7. Preporuka za tvoj konkretni slučaj (Gnosis + eIDAS JWT)

S obzirom na sve gore:

- Ako **možeš birati algoritam**, P‑256 (secp256r1) s FCL ili OpenZeppelin P256 fallbackom je **znatno jeftiniji i jednostavniji** na Gnosisu od RSA‑2048:
  - ~70–300k gas vs. ~1.5–2.5M gas po verifikaciji.[2][1]
- Trošak po transakciji na Gnosisu je u **milicentima**, tako da je *ekonomski* prihvatljiva i direktna on‑chain verifikacija, pogotovo ako ti je potrebno jako malo verifikacija po korisniku.[4][5][3]
- Ako ti je bitno **sakriti payload** (npr. osobne podatke iz eIDAS JWT‑a) ili ako certifikacijski lanci postanu preteški za čist EVM, razmotri:
  - **ZK pristup**: RSA/P‑256 verifikacija + minimalni skup uvjeta u ZK krugu (Circom, Halo2, Noir) i on‑chain Groth16 verifikator (~200–300k gas).[25][1]
- Za **JWT sloj**:
  - parsiraj JWT off‑chain i šalji samo `headerBase64`, `payloadBase64`, `signature` + minimalne hashirane claimove on‑chain,
  - ili slijedi SocialLock pattern ako ti eksplicitno treba on‑chain čitljiv JSON, uz svijest da onda cijeli tok ide prema ~1.5–2M gas.[12]

Ako želiš, u sljedećem koraku mogu ti skicirati konkretan dizajn ugovora za Gnosis (varijanta s P‑256 i varijanta s RSA+ZK), s procjenom gas budžeta po funkciji i minimalnim JWT/eIDAS payloadom koji moraš vidjeti on‑chain.

Sources
[1] GitHub - pcaversaccio/p256-verifier-vyper: P256 (a.k.a. secp256r1 elliptic curve) signature verification 🐍Vyper contract. https://github.com/pcaversaccio/p256-verifier-vyper
[2] Address: 0x094b465a...1d28dc6e4 | Etherscan https://etherscan.io/address/0x094b465a27156d11f9a784d08c931201d28dc6e4
[3] RIP-7696 : generic Double Scalar Multiplication (DSM) for all curves https://ethereum-magicians.org/t/rip-7696-generic-double-scalar-multiplication-dsm-for-all-curves/19798
[4] Quick gas comparison between multiple ways to access struct elements stored in a single word https://gist.github.com/simon-something/d88f697a9508df0ddb205cbbfc35fe09
[5] How to use xDai in your Dapp https://soliditydeveloper.com/xdai
[6] GitHub - AppliedCryptoGroup/Solidity-Crypto-Library: Collection of cryptographic primitives usable in Solidity smart contracts. https://github.com/AppliedCryptoGroup/Solidity-Crypto-Library
[7] Gnosis Gas Tracker - QuickNode https://www.quicknode.com/gas-tracker/gnosis
[8] Pull requests · adria0/SolRsaVerify - GitHub https://github.com/adria0/SolRsaVerify/pulls
[9] SolRsaVerify/src/RsaVerifyOptimized.sol at master - adria0 - GitHub https://github.com/adria0/SolRsaVerify/blob/master/src/RsaVerifyOptimized.sol
[10] Support RSA signature verification #74 - ethereum/EIPs - GitHub https://github.com/ethereum/EIPs/issues/74
[11] Big number library in Solidity (for modexp) https://gist.github.com/axic/3b5317141556c203854a6b7fc5f28971
[12] GitHub - axic/ethereum-rsa: RSA signature verification in Ethereum https://github.com/axic/ethereum-rsa
[13] p256 package - github.com/deep-thought-labs/infinite ... https://pkg.go.dev/github.com/deep-thought-labs/infinite/precompiles/p256
[14] EIP-198: Big integer modular exponentiation https://eips.ethereum.org/EIPS/eip-198
[15] Highlights from the All Core Developers Execution (ACDE) Call #212 https://etherworld.co/zh/highlights-from-the-all-core-developers-execution-acde-call-212/
[16] Smart Wallets and Passkeys - Corbado https://www.corbado.com/blog/smart-wallets-passkeys
[17] Safe and Passkeys https://docs.safe.global/advanced/passkeys/passkeys-safe
[18] Pixel | ETHGlobal https://ethglobal.com/showcase/pixel-7zw9q
[19] Ethereum - Cryptocurrencies - IQ.wiki https://iq.wiki/wiki/ethereum
[20] GitHub - hir0min/solidity-base64: A solidity library for base64/base64Url encoding and decoding https://github.com/hir0min/solidity-base64
[21] openzeppelin-contracts/contracts/utils/Base64.sol at master · OpenZeppelin/openzeppelin-contracts https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Base64.sol
[22] chrisdotn/jsmnSol: A JSON parser for solidity - GitHub https://github.com/chrisdotn/jsmnSol
[23] Welcome | Solidity CBOR https://owlprotocol.github.io/solidity-cbor/docs/
[24] nethermind/src/Nethermind/Chains/gnosis.json at master ... - GitHub https://github.com/NethermindEth/nethermind/blob/master/src/Nethermind/Chains/gnosis.json
[25] [PDF] Anastasia: Cinderella's Stepsister Turning Shabby X.509 Certificates ... https://storage.googleapis.com/taikai-storage/others/3a084ce0-91c6-11f0-b9c8-29eaf3ba1d1aAnastasia.pdf
[26] Releasing DCAP library v4: A unified interface to verify TEEs on-chain https://blog.ata.network/releasing-dcap-library-v4-a-unified-interface-to-verify-tees-on-chain-2f27538babb4?gi=10611bb18719
[27] Automata DCAP Attestation - GitHub https://github.com/automata-network/automata-dcap-attestation
[28] GitHub - automata-network/automata-on-chain-pccs: Solidity Implementation of On-Chain PCCS used for Intel DCAP Attestations https://github.com/automata-network/automata-on-chain-pccs
[29] GitHub - sanjukammath/SolRsaVerify: Solidity RSA Sha256 Pkcs1 Verification https://github.com/sanjukammath/SolRsaVerify
[30] p256-verifier/README.md at master · daimo-eth/p256-verifier https://github.com/daimo-eth/p256-verifier/blob/master/README.md
[31] Ethereum Average Gas Price (Daily) - Historical Data & Tren… - YCharts https://ycharts.com/indicators/ethereum_average_gas_price
[32] SolRsaVerify | Solidity RSA Sha256 Pkcs1 Verification | Blockchain library https://kandi.openweaver.com/javascript/adria0/SolRsaVerify
[33] The First Audited P256Verifier https://daimo.com/blog/p256verifier
[34] Gnosis Chain (xDAI) Blockchain Explorer https://gnosisscan.io
[35] GitHub - adria0/SolRsaVerify: Solidity RSA Sha256 Pkcs1 Verification https://github.com/adria0/SolRsaVerify
[36] GitHub - daimo-eth/p256-verifier: P256 signature verification solidity contract https://github.com/daimo-eth/p256-verifier
[37] 2.1-1.6 Gwei | XDAI Gas Tracker | GnosisScan https://gnosisscan.io/gastracker
[38] Track Gnosis chain gas fees in Gwei | Blockscout https://gnosis.blockscout.com/gas-tracker
[39] What Are Solana Gas Fees and Why They're So Cheap | 2025 Guidelearn.backpack.exchange › articles › solana-gas-fees https://learn.backpack.exchange/articles/solana-gas-fees
[40] Android Key Attestation validation library https://github.com/google/android-key-attestation
[41] What is RIP-7212? Precompile for secp256r1 Curve Support https://www.alchemy.com/blog/what-is-rip-7212
[42] Understanding Gas Fees for On-Chain Verification https://docs.privado.id/docs/faqs/content/verifier-on-chain-verification-gas-costs/
[43] Android-Security-Reference/framework/key_attestation.md ... https://github.com/doridori/Android-Security-Reference/blob/master/framework/key_attestation.md
[44] EIP-7212: Precompiled for secp256r1 Curve Support - RIPs https://ethereum-magicians.org/t/eip-7212-precompiled-for-secp256r1-curve-support/14789
[45] MynaWallet AA Grant Progress Update #3 - a42x https://a42x.co.jp/news/2023/10/02/mynawallet-aa-ef-grant-update-03-en/
[46] GitHub - a-sit-plus/warden: Server-Side Mobile Client Attestation Library https://github.com/a-sit-plus/warden
[47] What is RIP-7212? Understanding the secp256r1 Precompile for ... https://eco.com/support/en/articles/11813610-what-is-rip-7212-understanding-the-secp256r1-precompile-for-web3
[48] Gas cost reduction · Issue #29 · lurk-lab/solidity-verifier https://github.com/lurk-lab/solidity-verifier/issues/29
[49] Anastasia - ETHTokyo 2025 hackathon @ TAIKAI https://taikai.network/en/ethtokyo/hackathons/hackathon-2025/projects/cmfk7bpn204l39gzu4pph8e58/idea
[50] RIP-7212: Precompile for secp256r1 Curve Support #8605 - GitHub https://github.com/hyperledger/besu/issues/8605
[51] Gas Cost Analysis | Electron Labs https://docs.electron.dev/quantum-proof-aggregation-for-ethereum/gas-cost-analysis
[52] Anastasia: Cinderella's Stepsister Turning Shabby X.509 Certificates ... https://github.com/yamdan/anastasia
[53] Clave https://x.com/getclave/status/1673422876500717603
[54] Solidity RSA signatures for airdrops and presales: Beating ECDSA ... https://rareskills.io/post/solidity-rsa-signatures-for-aidrops-and-presales-beating-ecdsa-and-merkle-trees-in-gas-efficiency
[55] jwt::alphabet::base64url Struct Reference - GitHub Pages https://thalhammer.github.io/jwt-cpp/structjwt_1_1alphabet_1_1base64url.html
[56] [PDF] Decentralized Release of Self-emerging Data using Smart Contracts https://arxiv.org/pdf/1902.05623.pdf
[57] Miller Rabin probabilistic primality test for Solidity, and RSA-2048 modexp https://gist.github.com/HarryR/0520edbd653664917525fb64b5d2a74f
[58] GitHub - bmeredith/solidity-json-writer: A library to aid in the generation and construction of JSON for smart contract development. https://github.com/bmeredith/solidity-json-writer
[59] [PDF] Integrating User Identity to Ethereum Smart Contract Wallet https://dspace.ut.ee/bitstreams/739eb335-afee-40b0-bd69-1ccaa39bf43e/download
[60] JWT Library https://gist.github.com/Justicea83/1abe3edeb35ba83669c1eca80a7eb11f
[61] [PDF] IKP: Turning a PKI Around with Decentralized Automated Incentives https://www.ieee-security.org/TC/SP2017/papers/290.pdf
[62] JWT-CPP - GitHub Pages https://thalhammer.github.io/jwt-cpp/
[63] List of Solidity libraries in the wild - General - OpenZeppelin Forum https://forum.openzeppelin.com/t/list-of-solidity-libraries-in-the-wild/2250
[64] How to Build a Post-Quantum Readiness Inventory - ChainScore Labs https://chainscorelabs.com/en/guides/cryptographic-principles-advanced-cryptography-and-zk-snarks/post-quantum-planning/how-to-build-a-post-quantum-readiness-inventory
[65] solidity-jwt/contracts/Base64.sol at master · OpenZeppelin/solidity-jwt https://github.com/OpenZeppelin/solidity-jwt/blob/master/contracts/Base64.sol
[66] GitHub - rdubois-crypto/FreshCryptoLib: Cryptographic Primitives for Blockchain Systems (solidity, cairo, C and rust) http://github.com/rdubois-crypto/FreshCryptoLib
[67] Utilities | OpenZeppelin Docs https://docs.openzeppelin.com/contracts/5.x/utilities
[68] crypto-lib/src/README.md at main · get-smooth/crypto-lib https://github.com/get-smooth/crypto-lib/blob/main/src/README.md
[69] openzeppelin-contracts/docs/modules/ROOT/pages/utilities.adoc at master · OpenZeppelin/openzeppelin-contracts https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/docs/modules/ROOT/pages/utilities.adoc
[70] p256 package - github.com/zenanetwork/zena/precompiles/p256 https://pkg.go.dev/github.com/zenanetwork/zena/precompiles/p256
[71] Units and Globally Available Variables https://docs.soliditylang.org/en/latest/units-and-global-variables.html?highlight=abi.encode
[72] Release v5.1.0 · OpenZeppelin/openzeppelin-contracts | Artem Chystiakov https://www.linkedin.com/posts/artemchystiakov_release-v510-openzeppelinopenzeppelin-contracts-activity-7252993938611953665-c6-q
[73] P256 Precompile - Sei Docs https://docs.sei.io/evm/precompiles/P256
[74] https://github.com/bcnmy/scw-contracts/blob/main/contracts/smart-account/modules/PasskeyValidationModules/Secp256r1.sol https://gist.github.com/mingderwang/6536895f7266b5beaf944f753e4243d9
[75] openzeppelin-contracts-upgradeable/contracts/utils/README.adoc at master · OpenZeppelin/openzeppelin-contracts-upgradeable https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/README.adoc
[76] DC Posch and Nalin Bhardwaj, Founders of Daimo https://web3galaxybrain.com/episode/DC-Posch-and-Nalin-Bhardwaj-Founders-of-Daimo
[77] P-256 https://cryspen.com/hacl-packages/c/c-v0.6.0/hacl/signature/ecdsa/p256.html
[78] A Collection of Gas Optimisation Tricks - OpenZeppelin Forum https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966
[79] The Gas Sector - JSM Group https://www.jsmgroup.com/our-sectors/gas/
[80] on-chain-jwt-verification/README.md at main · Chazzzzzzz/on-chain-jwt-verification https://github.com/Chazzzzzzz/on-chain-jwt-verification/blob/main/README.md
[81] solidity-jwt/contracts/Strings.sol at master - GitHub https://github.com/OpenZeppelin/solidity-jwt/blob/master/contracts/Strings.sol
[82] A JSON Parser For Solidity - Steemit https://steemit.com/json/@chrisdotn/a-json-parser-for-solidity
[83] GitHub - Chazzzzzzz/on-chain-jwt-verification: okx-hackathon https://github.com/Chazzzzzzz/on-chain-jwt-verification
[84] solidity-jwt/contracts/JWKS.sol at master · OpenZeppelin/solidity-jwt https://github.com/OpenZeppelin/solidity-jwt/blob/master/contracts/JWKS.sol
[85] [PDF] Configurable Smart Contracts Automation for EVM based Blockchains http://paper.ijcsns.org/07_book/202310/20231018.pdf
[86] GitHub - automata-network/proof-of-machinehood-contracts: Solidity ... https://github.com/automata-network/proof-of-machinehood-contracts
[87] solidity-jwt/contracts/Identity.sol at master · OpenZeppelin/solidity-jwt https://github.com/OpenZeppelin/solidity-jwt/blob/master/contracts/Identity.sol
[88] pillardevelopment | 2622610 - GithubHelp - Git Product Home https://githubhelp.com/pillardevelopment/?page=2
[89] How do I properly validate JWT tokens and avoid security ... https://mojoauth.com/ciam-qna/how-to-properly-validate-jwt-tokens-and-avoid-security-vulnerabilities
[90] GitHub - OpenZeppelin/solidity-jwt: Experiments with Solidity JWT. Do not use in production. https://github.com/OpenZeppelin/solidity-jwt
[91] Zeno V7 Bug Bot Analysis Report | PDF - Scribd https://www.scribd.com/document/608699528/F7-%E1%83%A1%E1%83%90%E1%83%95%E1%83%90%E1%83%A0%E1%83%AF%E1%83%98%E1%83%A8%E1%83%9D%E1%83%94%E1%83%91%E1%83%98%E1%83%A1-%E1%83%99%E1%83%A0%E1%83%94%E1%83%91%E1%83%A3%E1%83%9A%E1%83%98
[92] Part 3: Reading an Onchain Value | Chainlink Documentation https://docs.chain.link/cre/getting-started/part-3-reading-onchain-value-ts
[93] P256 Precompile - Sei Docs https://docs.sei.io/evm/precompiles/p256-precompile
[94] Super assembly optomized version of spec256r verification, takes sig verification from ~900K to ~300K gas https://gist.github.com/aleph-v/19211ec45dde57168d6c13ff119f75c8
[95] Changelog | OpenZeppelin Docs https://docs.openzeppelin.com/contracts/5.x/changelog
[96] safe-global/safe-passkey - UNPKG https://app.unpkg.com/@safe-global/safe-passkey@0.2.0/files/README.md
[97] 2024-03-coinbase/src/FreshCryptoLib/FCL.sol at main - GitHub https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol
[98] Verify Signature Solidity in Foundry - RareSkills https://rareskills.io/post/openzeppelin-verify-signature
[99] rdubois-crypto/FreshCryptoLib: Cryptographic Primitives for ... - GitHub https://github.com/rdubois-crypto/FreshCryptoLib
[100] Introducing OpenZeppelin Contracts v5.1 https://www.openzeppelin.com/news/introducing-openzeppelin-contracts-v5.1
[101] GitHub - base/FCL-ecdsa-verify-audit: We audited the sepc256r1 ... https://github.com/base-org/FCL-ecdsa-verify-audit
[102] delegation-framework/documents/HybridDeleGator.md at main https://github.com/MetaMask/delegation-framework/blob/main/documents/HybridDeleGator.md
[103] RIPs/RIPS/rip-7212.md at master · ethereum/RIPs https://github.com/ethereum/RIPs/blob/master/RIPS/rip-7212.md
[104] Catalog of EVM Precompile and System Functions across Chains https://github.com/shemnon/precompiles
[105] llms.txt - Gnosis Chain https://docs.gnosischain.com/llms.txt
[106] EIP-7212: Precompiled for secp256r1 Curve Support - Page 3 - RIPs https://ethereum-magicians.org/t/eip-7212-precompiled-for-secp256r1-curve-support/14789?page=3
[107] Precompiles - OP Stack Specification - Optimism Specs https://specs.optimism.io/protocol/precompiles.html
[108] Week 76: Current Situation of RIP-7212, Avail Launches Avail ... https://www.binance.com/en/square/post/11187408379066
[109] Expected EIPs in Ethereum's Fusaka Upgrade - EtherWorld.co https://etherworld.co/expected-eips-in-ethereums-fusaka-upgrade/
[110] Pectra Upgrade on Gnosis Chain https://docs.gnosischain.com/about/specs/hard-forks/pectra
[111] Building AA powered dApp with ZeroDev - Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/zerodev-guide
[112] Avalanche Granite network upgrade - Chainstack https://docs.chainstack.com/docs/avalanche-granite-upgrade
[113] zkSync GnosisSafeZk Assessment - OpenZeppelin https://www.openzeppelin.com/news/zksync-gnosissafezk-assessment-1
[114] The Future of Finance - Gnosis https://www.gnosis.io/chain
[115] Ethereum won't reach real adoption until we solve the mobile wallets ... https://www.reddit.com/r/ethereum/comments/1ggfsc0/ethereum_wont_reach_real_adoption_until_we_solve/
[116] 3000 Chains https://www.blog.blockscout.com/3000-chains/
[117] Predeploys & Precompiles - Blast Developer Documentation https://metalayerlabs.mintlify.app/building/predeploys-and-precompiles
[118] On Increasing the Block Gas Limit: Technical Considerations & Path ... https://ethresear.ch/t/on-increasing-the-block-gas-limit-technical-considerations-path-forward/21225
[119] Gnosis Chain Quickstart https://docs.gnosischain.com/developers/quickstart
[120] Do not add bls12 precompile, implement Pasta curves w/o trusted ... https://ethresear.ch/t/do-not-add-bls12-precompile-implement-pasta-curves-w-o-trusted-setup-instead/12808
[121] Propose to increase Block Gas Limit to 30M gas - Celo Forum https://forum.celo.org/t/propose-to-increase-block-gas-limit-to-30m-gas/5189
[122] Self-hosted Agents | Semaphore https://docs.semaphore.io/using-semaphore/self-hosted
[123] EIP-2537:Precompile for BLS12-381 Curve - Rango Exchange https://rango.exchange/learn/market-trends/eip-2537-precompile-curve
[124] Increase block gas target and gaslimit - EIPs - Ethereum Magicians https://ethereum-magicians.org/t/increase-block-gas-target-and-gaslimit/17626
[125] How to run a Gnosis Chain node - Erigon 3 https://docs.erigon.tech/get-started/easy-nodes/how-to-run-a-gnosis-chain-node
[126] expose BLS12-381 precompiles on FVM #846 - GitHub https://github.com/filecoin-project/FIPs/discussions/846
[127] Block with gas limit >30m? : r/ethereum - Reddit https://www.reddit.com/r/ethereum/comments/1ft87zc/block_with_gas_limit_30m/
[128] Issues · semaphore-protocol/semaphore - GitHub https://github.com/semaphore-protocol/semaphore/issues
[129] EIP-2537 (BLS12 precompile) discussion thread https://ethereum-magicians.org/t/eip-2537-bls12-precompile-discussion-thread/4187
[130] Top Gnosis Chain RPC Providers 2025 | Dwellir Blog https://www.dwellir.com/blog/best-gnosis-rpc-providers-2025
[131] Fileverse Private Social ZKovery Contracts - GitHub https://github.com/fileverse/zkovery
[132] ERC721 Extension for zk-SNARKs - zk-s[nt]arks - Ethereum Research https://ethresear.ch/t/erc721-extension-for-zk-snarks/13237
[133] docs(docs): add trusted setup ceremony instructions on documentation · semaphore-protocol/semaphore@3a9306c https://github.com/semaphore-protocol/semaphore/actions/runs/9267759828/workflow
[134] semaphoreui/semaphore: Modern UI and powerful API for ... https://github.com/semaphoreui/semaphore
[135] Contracts | Semaphore https://docs.semaphore.pse.dev/technical-reference/contracts
[136] semaphore-ui/README.md at master · weijiekoh/semaphore-ui https://github.com/weijiekoh/semaphore-ui/blob/master/README.md
[137] Semaphore (@SemaphoreDevs) / Posts / X - Twitter https://x.com/SemaphoreDevs
[138] @semaphore-extensions/contracts https://www.npmjs.com/package/@semaphore-extensions/contracts
[139] Getting started - Semaphore https://docs.semaphore.pse.dev/getting-started
[140] Address: 0x4DeC9E37...4A48931f8 | GnosisScan https://gnosisscan.io/address/0x4DeC9E3784EcC1eE002001BfE91deEf4A48931f8
[141] semaphore-contracts/README.md at master · Emergence-Dapp/semaphore-contracts https://github.com/Emergence-Dapp/semaphore-contracts/blob/master/README.md
[142] semaphore/ at main · semaphore-protocol/semaphore https://github.com/semaphore-protocol/semaphore?search=1
[143] zk-X509: Privacy-Preserving On-Chain Identity from Legacy PKI via ... https://arxiv.org/html/2603.25190v2
[144] EthRome Hackathon | Tobias Leinss https://leinss.xyz/projects/ethrome/
[145] Getting started | Semaphore https://docs.semaphore.pse.dev/es/getting-started
[146] Highly secure zkEmail accounts(Javascript/Rust) - LinkedIn https://www.linkedin.com/pulse/highly-secure-zkemail-accountsjavascriptrust-mahdad-kiyani-yyrme
[147] Usage Guide | ZK Email Verifier - Introduction https://docs.zk.email/zk-email-verifier/usage-guide
[148] Cryptography | OpenZeppelin Docs https://docs.openzeppelin.com/community-contracts/api/utils/cryptography
[149] @zk-email/email-tx-builder-circom - NPM https://www.npmjs.com/package/@zk-email/email-tx-builder-circom
[150] zkemail/zk-regex - GitHub https://github.com/zkemail/zk-regex
[151] Unleashing ZK: a new era of performance, privacy, and interoperability https://blog.hyli.org/unleashing-zk-a-new-era-of-performance-privacy-and-interoperability/
[152] ZK Email - aayush's thoughts https://blog.aayushg.com/zkemail/
[153] @zk-email/zk-regex-circom CDN by jsDelivr - A CDN for npm and ... https://www.jsdelivr.com/package/npm/@zk-email/zk-regex-circom
[154] Grant Proposal: Obsidion Wallet - Aztec - Aztec https://forum.aztec.network/t/grant-proposal-obsidion-wallet/6191
[155] Prove who sent an email https://zkemail.vercel.app
[156] zk-email-verify/packages/circuits/email-verifier.circom at main - GitHub https://github.com/zkemail/zk-email-verify/blob/main/packages/circuits/email-verifier.circom
[157] Launching Odyssey Testnet Chapter 1 - Ithaca https://ithaca.xyz/updates/odyssey
[158] Overview | ZK Email Verifier https://docs.zk.email/zk-email-verifier/
[159] zk-email-verify/packages/circuits/utils/bytes.circom at main - GitHub https://github.com/zkemail/zk-email-verify/blob/main/packages/circuits/utils/bytes.circom
[160] https://raw.githubusercontent.com/OpenZeppelin/doc... https://raw.githubusercontent.com/OpenZeppelin/docs/refs/heads/main/content/community-contracts/api/utils/cryptography.mdx
[161] Common Circom Pitfalls and How to Dodge Them — Part 2 https://blog.zksecurity.xyz/posts/circom-pitfalls-2/
[162] Halo2 prover time - Technology - Zcash Community Forum https://forum.zcashcommunity.com/t/halo2-prover-time/39358
[163] Performance and Benchmarks - Mopro https://zkmopro.org/docs/performance/
[164] groth16_solana - Rust - Docs.rs https://docs.rs/groth16-solana/latest/groth16_solana/
[165] Elliptic Curves, Pairings, and RSA in Circom, Applied ZK - Day 2 https://www.youtube.com/watch?v=xX9I5CMa5uE
[166] rsa-challenge-with-halo2/README.md at main · Cardinal-Cryptography/rsa-challenge-with-halo2 https://github.com/Cardinal-Cryptography/rsa-challenge-with-halo2/blob/main/README.md
[167] [Noir] benchmark for different platforms · Issue #414 · zkmopro/mopro https://github.com/zkmopro/mopro/issues/414
[168] Performance Analysis of Groth16 zkSNARK https://computingonline.net/computing/article/view/4329
[169] DKIM key length and algorithms: 1024 vs 2048, RSA vs Ed25519 ... https://dmarcpal.com/learn/dkim-key-length-1024-vs-2048-rsa-vs-ed25519
[170] Formal Verification of Halo2 Circuits in Lean - Nethermind https://www.nethermind.io/blog/formal-verification-of-halo2-circuits-in-lean
[171] Exceeds - Team AI Productivity Dashboard https://myteam.exceeds.ai/profile/0xvikas@gmail.com
[172] Security and gas improvements to the Groth16 verifier contract#36 https://github.com/iden3/snarkjs/pull/36
[173] circom-rsa-verify/README.md at main · zkp-application/circom-rsa-verify https://github.com/zkp-application/circom-rsa-verify/blob/main/README.md
[174] Halo2 Proving Systems - Emergent Mind https://www.emergentmind.com/topics/halo2-based-proving-systems
[175] chore: CI changes for mimalloc PR · noir-lang/noir@2fa62d1 - GitHub https://github.com/noir-lang/noir/actions/runs/21531219667
[176] [Literature Review] Know Your Contract: Extending eIDAS Trust into ... https://www.themoonlight.io/en/review/know-your-contract-extending-eidas-trust-into-public-blockchains
[177] Know Your Contract: Extending eIDAS Trust into Public Blockchains https://arxiv.org/html/2601.13903v1
[178] Adrian Doerk - EUDI-Wallet & eIDAS - LinkedIn https://www.linkedin.com/posts/adrian-doerk_eudiwallet-eudi-wallet-activity-7332790936663375873-hsBe
[179] [PDF] Know Your Contract: Extending eIDAS Trust into Public Blockchains https://arxiv.org/pdf/2601.13903.pdf
[180] [PDF] Ethereum Transactions and Smart Contracts among Secure Identities https://ceur-ws.org/Vol-2334/DLTpaper1.pdf
[181] #eudiwallet #eidas #potentials | A-Trust GmbH - LinkedIn https://www.linkedin.com/posts/a-trust-gmbh_eudiwallet-eidas-potentials-activity-7369270554329120768-2MnK
[182] Blockchain & E-Signatures: Future-Proof Your Digital Contracts 2026 https://yousign.com/blog/blockchain-e-signatures-future-proof-digital-contracts
[183] Know Your Contract: Extending eIDAS Trust into Public Blockchains https://arxiv.org/abs/2601.13903
[184] #eudiwallet #interoperability #eidas #digitalidentity #arf #lissi | Lissi ... https://www.linkedin.com/posts/lissi_eudiwallet-interoperability-eidas-activity-7372158554629246977-0pZY
[185] Secure online signatures using blockchain - Coexya https://www.coexya.eu/en/our-offer/electronic-signature-on-blockchain/
[186] Distributed Networks under eIDAS 2: Regaining Data Sovereignty https://www.linkedin.com/pulse/distributed-networks-under-eidas-2-regaining-data-andr%C3%A9-casterman-icwte
[187] [PDF] Notification update for HR electronic identity scheme 20/11/2023 ... https://ec.europa.eu/digital-building-blocks/sites/download/attachments/62885743/21112023_Notification%20update%20for%20HR%20eID%20scheme%20NIAS.pdf?version=1&modificationDate=1709628276200&api=v2
[188] Privacy-Preserving eIDAS Compliance in Blockchain Wallets via zkVM https://ieeexplore.ieee.org/document/11038609/
[189] Know Your Contract: Extending eIDAS Trust into Public Blockchains https://web3.arxiv.org/pdf/2601.13903
[190] [PDF] Positive preliminary assessment of the satisfactory fulfilment of ... https://www.tweedekamer.nl/downloads/document?id=2025D53482
[191] Unlocking Trust: Exploring Attestation as a Service on Ethereum https://www.linkedin.com/pulse/unlocking-trust-exploring-attestation-service-mukul-tripathi
[192] Ethereum Attestation Service: EAS https://attest.org
[193] [PDF] Security Target - Common Criteria https://www.commoncriteriaportal.org/nfs/ccpfiles/files/epfiles/616-LSS%20ST%20v1.12.pdf
[194] Bankless - 187 - The Ethereum Attestation Service Transcript and ... https://podscripts.co/podcasts/bankless/187-the-ethereum-attestation-service
[195] trust.signals[] extension: consolidated signal type specification #1628 https://github.com/a2aproject/A2A/issues/1628
[196] Supporting Document - PP-Module for SSL/TLS Inspection Proxy https://www.niap-ccevs.org/static_html/protection-profile/477/SD%20for%20SSLTLS%20Inspection%20Proxy/index.html
[197] Zipwire.ProofPack.Ethereum 0.4.0 - NuGet https://packages.nuget.org/packages/Zipwire.ProofPack.Ethereum/
[198] Protocol Overview | CapSign - GitBook https://capsign.gitbook.io/capsign/protocol-developers/protocol
[199] Functional Package for Transport Layer Security (TLS) https://commoncriteria.github.io/pp/tls/tls-release.html
[200] Binance Is Positioning as the Home of Sovereign Infrastructure https://www.binance.com/sk/square/post/306455925856977
[201] AI Agents Hub: Use Cases showcase #17619 - GitHub https://github.com/ethereum/ethereum-org-website/issues/17619
[202] Foundation Mission Request - Farcaster Attestations [TECHNICAL] https://github.com/ethereum-optimism/ecosystem-contributions/issues/194
[203] Attestation | Xiang Xie https://xiangxiecrypto.github.io/2024-08-28-zkAttestation/
[204] Ethereum Attestation Service: Building Trust Online with EAS https://www.linkedin.com/posts/octantapp_eas-activity-7435462398607695873-_qAu
[205] ERC-8092: Associated Accounts - Ethereum Magicians https://ethereum-magicians.org/t/erc-8092-associated-accounts/26858
[206] farcaster-attestation/farcaster-solidity https://www.npmjs.com/package/@farcaster-attestation/farcaster-solidity
[207] Fake GLV in-circuit - HackMD https://hackmd.io/@yelhousni/Hy-aWld50
[208] 🔥 Gas and Benchmarks | Fhenix https://docs.fhenix.zone/docs/devdocs/Writing%20Smart%20Contracts/Gas-and-Benchmarks
[209] Solidity - Obvious Wallet https://hackmd.io/@albertsu/a-primer-on-secp256r1
[210] Fake GLV: You don't need an efficient endomorphism to implement ... https://ethresear.ch/t/fake-glv-you-dont-need-an-efficient-endomorphism-to-implement-glv-like-scalar-multiplication-in-snark-circuits/20394
[211] GasBad - Comparing gas efficiency of Solidity libraries https://www.emanuelecivini.com/post/gas-bad/
[212] FCL-ecdsa-verify-audit/docs/secp256r1-ecdsa-verify-solidity-review-testing-plan.pdf at main · base/FCL-ecdsa-verify-audit https://github.com/base/FCL-ecdsa-verify-audit/blob/main/docs/secp256r1-ecdsa-verify-solidity-review-testing-plan.pdf
[213] 2023-10-ethena/4naly3er-report.md at main · code-423n4/2023-10-ethena https://github.com/code-423n4/2023-10-ethena/blob/main/4naly3er-report.md
[214] Address: 0x6d3c6081...bd4702d69 | Etherscan https://etherscan.io/address/0x6d3c60810c713d1eaca221bcaab3e17bd4702d69
[215] buidl-wallet-contracts/src/msca/6900/v0.8/modules/multisig ... - GitHub https://github.com/circlefin/buidl-wallet-contracts/blob/master/src/msca/6900/v0.8/modules/multisig/WeightedMultisigValidationModule.sol
[216] adria0’s gists https://gist.github.com/adria0
[217] Key and ID attestation - Android Open Source Project https://source.android.com/docs/security/features/keystore/attestation
[218] server - platform/external/android-key-attestation https://android.googlesource.com/platform/external/android-key-attestation/+/c544eb01be2a3bf891e2e9d57da3e5bbd52a0662/server
[219] Verify hardware-backed key pairs with key attestation | Security https://developer.android.com/privacy-and-security/security-key-attestation
[220] Key and ID attestation | Android Open Source Project https://source.android.google.cn/docs/security/features/keystore/attestation?hl=en
[221] ecdsa-signature · GitHub Topics https://github.com/topics/ecdsa-signature
[222] http://mirrors.dotsrc.org/devuan/d1h_cache http://mirrors.dotsrc.org/devuan/d1h_cache
[223] popup.js for metamask 4.3.0 - GitHub Gist https://gist.github.com/danfinlay/8ea6c50ed868f6e9444d8e0b1bf26e5a
[224] SafetyNet Attestation Backend Example Implementation for Validating Android Device Authenticity https://gist.github.com/schirrmacher/caf2a8bbfdb3d59d95af80dee99d42e7
[225] The Rule of LAW not AI - by Swen Werner https://swenldn.substack.com/p/the-rule-of-law-not-ai
[226] SafetyNet Attestation API | Android Developers https://developer.android.google.cn/privacy-and-security/safetynet/attestation?hl=EN-GB
[227] Solidity wrapper for Ethereum Byzantium's BigInt `modexp` built-in contract 0x5 https://gist.github.com/riordant/226f8882556a5c7981b239e4e5d96918
[228] Notebooks https://notebooks.githubusercontent.com/view/ipynb?browser=chrome&color_mode=auto&commit=1ca84d4ef6524dae09b4d4e8c5e6d1c6b6260a27&device=unknown&enc_url=68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f6c70656e65742f79616e6465782d6269672d646174612d616e616c797369732f316361383464346566363532346461653039623464346538633565366431633662363236306132372f5765656b322d486976652d41737369676e6d656e742d322d444d4c2533412d46696e642d4d6f73742d506f70756c61722d546167732e6970796e62&logged_in=false&nwo=lpenet%2Fyandex-big-data-analysis&path=Week2-Hive-Assignment-2-DML%3A-Find-Most-Popular-Tags.ipynb&platform=android&repository_id=186616288&repository_type=Repository&version=98
[229] Inside Android's SafetyNet Attestation https://www.blackhat.com/docs/eu-17/materials/eu-17-Mulliner-Inside-Androids-SafetyNet-Attestation.pdf
[230] GitHub topics: pkcs1 https://repos.ecosyste.ms/hosts/GitHub/topics/pkcs1?order=desc&sort=stargazers_count
[231] GitHub - yangfh2004/SolSha2Ext: An Solidity library to provide SHA384 and SHA512 as an extension in the SHA2 family https://github.com/yangfh2004/SolSha2Ext
[232] Threshold Cryptography - Lit Protocol Documentation https://litprotocol.mintlify.app/learning-lit/threshold-cryptography
[233] EIP-712: Typed structured data hashing and signing https://eips.ethereum.org/EIPS/eip-712
[234] Validator Deposits - Gnosis Chain https://docs.gnosischain.com/node/manual/validator/deposit
[235] safe-smart-account/docs/error_codes.md at main - GitHub https://github.com/safe-global/safe-smart-account/blob/main/docs/error_codes.md
[236] Gnosis Price: GNO Live Price Chart, Market Cap & News Today | CoinGecko https://www.coingecko.com/en/coins/gnosis
[237] Cryptography | OpenZeppelin Docs https://docs.openzeppelin.com/contracts/5.x/api/utils/cryptography
[238] kl0519-0ea1f855-610d-4ea8-98a6-bc93b1d22a8e.txt - Hugging Face https://huggingface.co/datasets/max-id/gaianet-qdrant-snapshot/raw/92f9fd85513316db55ec307231c740293986fafe/kl0519-0ea1f855-610d-4ea8-98a6-bc93b1d22a8e/kl0519-0ea1f855-610d-4ea8-98a6-bc93b1d22a8e.txt
[239] Make the output of `gno precompile` more parseable · Issue #1636 · gnolang/gno https://github.com/gnolang/gno/issues/1636
[240] ArbSys | v3.0.0-beta.3 pre-release https://docs.opengsn.org/soldoc/contracts/arbitrum/arbsys
[241] The Dai stablecoin source code https://gist.github.com/jacks0n9/333bae10a64fe627fc453cd406f3be90
[242] Precompile Contracts | Axon Documentation https://docs.axonweb3.io/contract/precompile_contracts/
[243] Moonbeam: Monitoring the Conviction Voting contract - Chainstack https://docs.chainstack.com/docs/moonbeam-monitoring-the-conviction-voting-contract
[244] Ethereum Precompiled Contracts https://docs.bifrostnetwork.com/bifrost-network/developer-documentations/ethereum-api/ethereum-precompiled-contracts
[245] Backend ENVs: Common - Blockscout Docs https://docs.blockscout.com/setup/env-variables/backend-env-variables
[246] go-ethereum/docs/postmortems/2021-08-22-split ... - GitHub https://github.com/ethereum/go-ethereum/blob/master/docs/postmortems/2021-08-22-split-postmortem.md
[247] Address: 0xd1ce9000...11555e4b3 | Etherscan https://etherscan.io/address/0xd1ce90003a10e6dab877890ab1fd96511555e4b3
[248] Blast: Tracking Automatic, Void, Claimable accounts - Chainstack https://docs.chainstack.com/docs/blast-tracking-automatic-void-claimable-accounts
[249] xDai $STAKE collab - Rainbow Bridge - #5 by illia - Ecosystem https://gov.near.org/t/xdai-stake-collab-rainbow-bridge/308/5
[250] Address: 0x4a34f5ce...686F9C3cd | Etherscan https://etherscan.io/address/0x4a34f5ceCa539b2eB42F1cbED8cCB3D686F9C3cd
[251] nimbus-eth2/CHANGELOG.md at stable - GitHub https://github.com/status-im/nimbus-eth2/blob/stable/CHANGELOG.md
[252] Chains List — Ankr https://www.ankr.com/docs/rpc-service/chains/chains-list/
[253] Big integer modular exponentiation (EIP-198) gas cost https://ethereum-magicians.org/t/eip-2565-big-integer-modular-exponentiation-eip-198-gas-cost/4150
[254] EIPs/EIPS/eip-198.md at master · ethereum/EIPs https://github.com/ethereum/EIPs/blob/master/EIPS/eip-198.md
[255] Baremetal | Docs - InfraDAO Overview https://docs.infradao.com/archive-nodes-101/fuse/baremetal
[256] EIP-2565: Big integer modular exponentiation (EIP-198) gas cost https://ethereum-magicians.org/t/eip-2565-big-integer-modular-exponentiation-eip-198-gas-cost/4150/17
[257] gnosischain repositories · GitHub https://github.com/orgs/gnosischain/repositories?type=all
[258] Address: 0xdE0A5B2c...f6C11B391 | Etherscan https://etherscan.io/address/0xdE0A5B2c20199a3e28C09bAE0900934f6C11B391
[259] Previous ENV Variable Home Page - Blockscout Docs https://docs.blockscout.com/setup/env-variables/deprecated-env-variables/env-variables
[260] Address: 0x00000000...000000005 - Monadscan https://monadscan.com/address/0x0000000000000000000000000000000000000005
[261] Test Modexp https://eest.ethereum.org/verkle@v0.0.6/tests/byzantium/eip198_modexp_precompile/test_modexp/
[262] Solidity Base64 utilities https://gist.github.com/ryancharris/ed5c4f161f2ab049adf41a7f3eed2229
[263] spomky-labs / base64url https://packagist.org/packages/spomky-labs/base64url
[264] Why Base64 is used in JWTs? https://stackoverflow.com/questions/58341833/why-base64-is-used-in-jwts
[265] Spomky-Labs/base64url: Base64 URL Safe Encoding/ ... https://github.com/Spomky-Labs/base64url
[266] What Is On-Chain Attestation? The API Primitive That Replaces ... https://insumermodel.com/blog/what-is-on-chain-attestation-api.html
[267] Contracts — Solidity 0.8.35-develop documentation https://docs.soliditylang.org/en/latest/contracts.html
[268] Unlocking JSON Web Tokens (JWT): Security, Base64 Encoding and Beyond https://blog.devgenius.io/unlocking-json-web-tokens-jwt-security-base64-encoding-and-beyond-bd8a81021619?gi=a8dd8b5aa4d7
[269] On-chain identity proof verification design patterns: an ... - GitHub https://github.com/WebOfTrustInfo/rwot11-the-hague/blob/master/final-documents/onchain_identity_verification_flows.md
[270] GitHub - moleculeprotocol/IPNFT: IP-NFTs are building blocks for ... https://github.com/moleculeprotocol/IPNFT
[271] Exploring ABI Encodings in Solidity: Gas Cost Savings | Abhijit Roy https://www.linkedin.com/posts/abhi3700_exploring-abi-encodings-in-solidity-gas-activity-7092459074566447104-jOpU
[272] Units and Globally Available Variables https://docs.soliditylang.org/en/latest/units-and-global-variables.html
[273] JOHNSEN'S GAS TREATMENT 12 FL.OZ. https://www.johnsens.com/content/products/sds/4679.pdf
[274] GitHub - TechnoGeek01/solidity-gas-optimizations https://github.com/TechnoGeek01/solidity-gas-optimizations
[275] Autenticação social baseada em Solidity— Enviando Cryptos para ... https://pt.w3d.community/devaraujo/autenticacao-social-baseada-em-solidity-enviando-cryptos-para-qualquer-conta-do-google-35m
[276] Read All The Things - A Smart Contract Design Philosophy https://www.linkedin.com/pulse/read-all-things-smart-contract-design-philosophy-matthew-willox
[277] Address: 0x68674876...50bf79129 | Etherscan https://etherscan.io/address/0x686748764c5c7aa06fec784e60d14b650bf79129
[278] Utils | OpenZeppelin Docs https://docs.openzeppelin.com/contracts/5.x/api/utils
[279] Address: 0xC837E029...EB0a628fA | Etherscan https://etherscan.io/address/0xC837E02982992B701A1B5e4E21fA01cEB0a628fA

