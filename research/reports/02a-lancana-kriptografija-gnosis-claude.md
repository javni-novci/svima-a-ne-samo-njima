# Lančana RSA/P-256 verifikacija — Claude Opus 4.6 izvještaj

**Prompt:** [02-lancana-kriptografija-gnosis.md](../prompts/02-lancana-kriptografija-gnosis.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# On-chain verifikacija RSA i P-256 potpisa na Gnosis Chainu za Certilia JWT

**Izravna on-chain verifikacija JWT tokena s hrvatskog eIDAS davatelja Certilia potpuno je izvediva na Gnosis Chainu, uz zanemarive troškove.** RSA-2048 verifikacija korištenjem OpenZeppelin biblioteke košta samo **~25.000–35.000 gas-a** (~$0,00 na Gnosis Chainu), dok P-256 softverska verifikacija bez precompilea košta **~200.000–330.000 gas-a** — i dalje daleko ispod razine blok limita. Certilia Identity Provider koristi OAuth2/OpenID Connect standard i vraća JWT tokene s verificiranim osobnim podacima (ime, prezime, OIB). S obzirom na to da Gnosis Chain ima gotovo besplatan gas (~$0,002 za 2M gas-a), čak i najskuplji pristup izravne verifikacije staje manje od jednog centa. ZK alternativa (Noir framework) dodatno smanjuje on-chain trošak na **~200.000–290.000 gas-a** za Groth16/PLONK dokaz, uz mogućnost privatnosti i selektivnog otkrivanja podataka.

---

## RSA verifikacija: OpenZeppelin je zlatni standard

Za JWT potpisan s RS256 algoritmom (RSA-SHA256 s PKCS#1 v1.5 paddingom), jasna preporuka je **OpenZeppelin RSA biblioteka** (dostupna od verzije 5.1). Ova biblioteka koristi EIP-198 `modexp` precompile na adresi `0x05` za modularnu eksponencijaciju — srce RSA verifikacije. `modexp` precompile dostupan je na Gnosis Chainu jer je potpuno EVM-kompatibilan.

Sama `modexp` operacija za RSA-2048 s eksponentom e=65537 košta **~5.461 gas-a** (prema EIP-2565 formuli). Ukupna verifikacija, uključujući SHA-256 hash, PKCS#1 v1.5 padding provjeru i calldata overhead, procjenjuje se na **~25.000–35.000 gas-a** za RSA-2048, odnosno **~45.000–60.000 gas-a** za RSA-4096.

Važno je napomenuti da **nijedna Solidity biblioteka ne podržava RSA-PSS** — dostupan je isključivo PKCS#1 v1.5. Ako Certilia koristi PSS padding, bilo bi potrebno razviti prilagođeno rješenje ili koristiti ZK pristup.

| Biblioteka | Algoritam | Gorivo (gas) | Revidirana? | Gnosis deployment | Aktivna? | URL |
|---|---|---|---|---|---|---|
| **OpenZeppelin RSA v5.1+** | PKCS#1 v1.5 SHA-256 | ~25–35k (RSA-2048) | ✅ Da (OZ, Immunefi bounty) | ✅ Radi (modexp 0x05) | ✅ Vrlo aktivna | [github.com/OpenZeppelin/openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) |
| adria0/SolRsaVerify | PKCS#1 v1.5 SHA-256 | ~25–35k (RSA-2048) | ❌ Ne | ✅ Radi | ⚠️ Zastarjela (preporuča OZ) | [github.com/adria0/SolRsaVerify](https://github.com/adria0/SolRsaVerify) |
| OZ/solidity-jwt | PKCS#1 v1.5 (via SolRsaVerify) | ~150–300k (puni JWT) | ❌ Eksperimentalna | ❌ Nije deployano | ❌ Neodržavana | [github.com/OpenZeppelin/solidity-jwt](https://github.com/OpenZeppelin/solidity-jwt) |
| zkEmail (ZK pristup) | RSA-SHA256 u ZK krugu | ~200–300k (Groth16 dokaz) | ✅ zkSecurity | ❌ Nije na Gnosisu | ✅ Vrlo aktivna | [github.com/zkemail/zk-email-verify](https://github.com/zkemail/zk-email-verify) |
| RareSkills RSA | Sirovi RSA (bez paddinga) | ~5–7k (e=3) | ❌ PoC | ❌ Ne | ❌ Samo PoC | [rareskills.io](https://rareskills.io/post/solidity-rsa-signatures-for-aidrops-and-presales-beating-ecdsa-and-merkle-trees-in-gas-efficiency) |

Originalni `adria0/SolRsaVerify` (Adrià Massanet) bio je pionirska implementacija, ali autor eksplicitno preporuča korištenje OpenZeppelin verzije. `axic/ethereum-rsa` (Alex Beregszaszi) je povijesni artefakt iz 2016. koji nije kompatibilan s modernim EVM-om. Google-ov `android-key-attestation` je Java biblioteka — **ne postoji Solidity port**. Ethereum Attestation Service (EAS) koristi isključivo secp256k1 ECDSA i **nema RSA komponentu**.

---

## P-256 verifikacija: RIP-7212 još nije na Gnosisu

Za JWT potpisan s ES256 algoritmom (ECDSA s P-256/secp256r1 krivuljom), situacija je složenija. **RIP-7212 precompile za secp256r1 verifikaciju NIJE aktiviran na Gnosis Chainu** (stanje: travanj 2026.). Gnosis prati Ethereumov raspored nadogradnji, a RIP-7212 je predviđen za **Osaka hardfork** na Ethereum L1. Gnosis je aktivirao Pectra u travnju/svibnju 2025., ali ta nadogradnja nije uključivala RIP-7212.

Lancima koji imaju RIP-7212 (Polygon, Optimism, Base, Arbitrum) verifikacija košta samo **3.450 gas-a**. Na Gnosisu je potrebna softverska verifikacija, što je značajno skuplje ali i dalje izvedivo.

| Biblioteka | Gorivo (gas) | Revidirana? | Gnosis deployment | Aktivna? | URL |
|---|---|---|---|---|---|
| **RIP-7212 precompile** | **3.450** | N/A (protokol) | ❌ Nije dostupan | — | [RIP-7212 spec](https://github.com/ethereum/RIPs/blob/master/RIPS/rip-7212.md) |
| **OpenZeppelin P256 v5.1+** | ~200–330k (softver) / 3.450 (precompile) | ✅ Da (OZ) | ✅ Radi (softverski fallback) | ✅ Vrlo aktivna | [github.com/OpenZeppelin/openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) |
| **FreshCryptoLib (FCL)** | ~205k (bez precomputacije) / ~69k (s precomputacijom) | ⚠️ Djelomično (Coinbase audit) | ✅ Radi | ✅ Aktivna | [github.com/rdubois-crypto/FreshCryptoLib](https://github.com/rdubois-crypto/FreshCryptoLib) |
| **Daimo p256-verifier** | ~330k (softver) / ~3.4k (precompile) | ✅ Veridise (10/2023) | ⚠️ Nije deployan, ali deployable | ✅ Aktivna | [github.com/daimo-eth/p256-verifier](https://github.com/daimo-eth/p256-verifier) |
| Coinbase webauthn-sol | ~330k (koristi FCL) | ✅ Code4rena | ✅ Radi | ✅ Aktivna | [github.com/base/webauthn-sol](https://github.com/base/webauthn-sol) |

OpenZeppelin P256.sol nudi tri moda: `verify()` koji automatski pokušava RIP-7212 precompile pa pada na softverski fallback, `verifyNative()` koji koristi samo precompile, i `verifySolidity()` koji koristi isključivo čisti Solidity. Za Gnosis Chain preporuča se `verifySolidity()` ili `verify()` (koji će automatski koristiti softver dok precompile nije dostupan).

**FreshCryptoLib s precomputacijom** nudi najnižu cijenu softverske verifikacije od **~69.000 gas-a**, ali zahtijeva **~3,2M gas-a** za početni deployment precomputacijske tablice. To se isplati nakon otprilike 30 transakcija.

### Ključna usporedba: RSA-2048 vs. P-256 na Gnosisu

**RSA-2048 je trenutno ~8–13x jeftiniji od P-256 na Gnosis Chainu** — ~25–35k gas-a naspram ~200–330k gas-a. Razlog je što RSA koristi `modexp` precompile (0x05) koji je dostupan na svim EVM lancima, dok P-256 nema nativni precompile na Gnosisu. Ipak, na Gnosis Chainu s trenutnim cijenama gas-a, oba pristupa koštaju zanemarivih ~$0,00.

---

## JWT parsiranje: potpis nad sirovim base64url podacima je optimalan pristup

JWT struktura je `base64url(header).base64url(payload).signature`, pri čemu se potpis računa nad **sirovim ASCII bajtovima** `header.payload` segmenta — ne nad dekodiranim sadržajem. To otvara ključnu optimizaciju: **nije potrebno dekodirati Base64 niti parsirati JSON on-chain**.

Preporučeni pristup je sljedeći:

- **Off-chain**: ekstrakcija i validacija JWT claimova (sub, aud, nonce, exp) iz dekodiranog payloada
- **On-chain**: prosljeđivanje sirovih `header.payload` bajtova + potpisa + očekivanih vrijednosti claimova kao zasebnih parametara
- **Smart contract**: verificira potpis nad sirovim bajtovima i provjerava da proslijeđene claim vrijednosti odgovaraju kontekstu

Ovaj pristup smanjuje ukupni gas s ~150.000–300.000+ (puni JWT parsing) na samo **~20.000–60.000 gas-a** (samo verifikacija potpisa za RSA-2048). Postoji eksperimentalni `OpenZeppelin/solidity-jwt` projekt koji implementira puni pipeline (Base64 → JSON → RSA verifikacija), ali je označen kao "Ne koristi u produkciji" i koristi zastarjeli Solidity ^0.5.0.

Za slučaj da je ipak potrebno Base64 dekodiranje on-chain, **Solady Base64** biblioteka je najefikasnija opcija (~1.500–3.000 gas-a za tipični JWT payload), dok je `JsmnSolLib` jedini poznati JSON parser za Solidity, ali s visokim troškovima (~50.000–200.000+ gas-a ovisno o broju claimova).

---

## Gnosis Chain: gotovo besplatan gas i dovoljno prostora u bloku

Gnosis Chain ima jedinstvenu prednost za ovu primjenu — **izuzetno niske cijene gas-a** jer se gas plaća u xDAI-u (stabilna valuta pegged 1:1 na DAI ≈ $1,00).

| Parametar | Vrijednost |
|---|---|
| **Trenutna cijena gas-a** | <0,000001 Gwei (gotovo nula) |
| **Cijena xDAI** | ~$1,00 USD |
| **Trošak 2M gas transakcije** | ~$0,002 (pri 1 Gwei konzervativno) |
| **Trošak 5M gas transakcije** | ~$0,005 (pri 1 Gwei konzervativno) |
| **Limit gas-a po bloku** | ~17 milijuna gas-a |
| **2M gas kao % bloka** | 11,8% — staje bez problema |
| **5M gas kao % bloka** | 29,4% — staje udobno |
| **Prosječna iskorištenost bloka** | ~31,6% |
| **Vrijeme bloka** | ~5 sekundi |

Gnosis Chain podržava **identične EVM precompile-ove kao Ethereum mainnet**: ecrecover (0x01), SHA-256 (0x02), modexp (0x05), BN-128 pairing (0x06-0x08), Blake2f (0x09). **Ne postoje dodatni precompile-ovi** koje Ethereum nema — ali također **nema RIP-7212** za secp256r1. Nema nikakvog rizika da bi transakcija od 2–5M gas-a bila prevelika za blok — to zauzima manje od trećine bloka.

---

## ZK alternativa: Noir framework kao najbrže rješenje

Ako je potrebna privatnost (npr. selektivno otkrivanje podataka iz JWT-a bez razotkrivanja punog tokena on-chain) ili ako se koriste potpisi koji nisu podržani u Solidity-ju (npr. RSA-PSS), ZK pristup je izvediv i troškovno efikasan.

**Noir (Aztec) je jasni pobjednik** za obje kriptografske operacije:

| Operacija | Circom (R1CS) | Noir (UltraHonk) | Noir prednost |
|---|---|---|---|
| **RSA-2048 verifikacija** | ~536.000 ograničenja | **~7.131 gate-ova** | **75x manje** |
| **RSA-2048 proving** | Nekoliko sekundi (server) | **262ms** (desktop) | Znatno brže |
| **P-256 ECDSA verifikacija** | >1M ograničenja (neizvedivo u pregledniku) | Nativna black-box funkcija | Dramatično bolje |
| **P-256 proving (M1 MacBook)** | Nije praktično u pregledniku | **2,06s** (multi-thread) | Jedini praktični framework |
| **P-256 proving (Android)** | Neizvedivo | **~6s** (Samsung Galaxy A23) | Jedini praktični framework |

Noir kompilira u ACIR i automatski generira Solidity verifier contracte za on-chain verifikaciju. **Groth16 verifikacija on-chain** košta **~200.000–230.000 gas-a** (koristi BN254 pairing precompile na adresi 0x08, dostupan na Gnosisu), dok **PLONK verifikacija** košta **~290.000 gas-a** ali ne zahtijeva per-circuit trusted setup.

**Ključne napomene o mobilnom provingu**: P-256 u Noiru radi na Android uređajima (~6s) i M1 MacBook-u (~2s), ali **iPhone 16 javlja out-of-memory grešku**. RSA-2048 u Noiru je dramatično brži (262ms na desktopu), ali mobilno testiranje nije široko dokumentirano. Circom RSA-2048 krugovi zahtijevaju ~536.000 ograničenja, što je previše za mobilne uređaje.

**Semaphore V4 je deployiran na Gnosis Chainu** na determinističkim adresama:
- SemaphoreVerifier: `0x4DeC9E3784EcC1eE002001BfE91deEf4A48931f8`
- Semaphore: `0x8A1fd199516489B0Fb7153EB5f075cDAC83c693D`

---

## Postojeći projekti: bogat ekosustav za inspiraciju

Nekoliko aktivnih projekata rješava gotovo identičan problem verifikacije identitetskih potpisa on-chain:

**zkLogin (Sui)** je najzreliji sustav za on-chain JWT verifikaciju. Koristi Groth16 ZK dokaze za verifikaciju OAuth JWT-ova (Google, Facebook, Apple) bez razotkrivanja JWT-a on-chain. Arhitektura je izravno primjenjiva na EVM lance s Groth16 verifierom.

**zkEmail** verificira DKIM potpise (RSA-SHA256) u ZK krugovima i potvrđuje dokaze on-chain. Projekt aktivno razvija **zk-jwt** komponentu koja radi istu stvar za JWT tokene. RSA verifikacija se odvija unutar Circom/Noir ZK krugova, dok on-chain ide samo Groth16 dokaz (~200–300k gas-a).

**Anon Aadhaar** (PSE/Ethereum Foundation) verificira RSA potpise indijske Aadhaar kartice u ZK krugovima — gotovo identičan use case kao Certilia JWT, samo za indijski identitetski sustav. Dostupna je i **Noir** verzija.

**Self Protocol** (bivši Proof of Passport / zkPassport) verificira ICAO potpise biometrijskih putovnica korištenjem ZK dokaza, s **7M+ korisnika** u 174 zemlje. Migrirali su s Circom-a na **Noir Ultra Plonk**.

**zk-X509** (ožujak 2025.) je novi akademski rad koji koristi SP1 zkVM za verifikaciju punih X.509 certifikatnih lanaca (uključujući RSA-2048 i ECDSA P-256) u ZK dokazima, s on-chain verifikacijom od ~300k gas-a.

Za eIDAS na blockchainu, **EBSI** (European Blockchain Services Infrastructure) je EU-financirana infrastruktura za prekograničnu verifikaciju vjerodajnica, ali koristi Verifiable Credentials i DID-ove, ne izravnu on-chain verifikaciju potpisa. **Nema projekata koji izravno verificiraju eIDAS potpise on-chain** — ovo bi bio pionirski projekt.

| Projekt | Lanac | Kripto verifikacija | Pristup | On-chain gas |
|---|---|---|---|---|
| **zkLogin (Sui)** | Sui | RSA/ECDSA JWT | ZK Groth16 | Nativni Sui tx |
| **zkEmail / zk-jwt** | Bilo koji EVM | RSA-SHA256 (DKIM/JWT) | Circom/Noir ZK + Groth16 | ~200–300k |
| **Anon Aadhaar** | Bilo koji EVM | RSA (Aadhaar/UIDAI) | Circom ZK + Groth16 | ~200–300k |
| **Self Protocol** | Bilo koji EVM | RSA/ECDSA (ICAO) | Noir Ultra Plonk | ~200–290k |
| **Automata DCAP** | Ethereum, Base, OP | RSA (X.509 cert chain) | On-chain + SNARK | ~300k (SNARK) |
| **zk-X509** | Bilo koji EVM | RSA-2048, ECDSA P-256 | SP1 zkVM + Groth16 | ~300k |
| **Coinbase Smart Wallet** | 248+ EVM lanaca | P-256 (passkey) | On-chain ECDSA | ~330k (softver) |

---

## Zaključak: preporučena arhitektura za Certilia JWT verifikaciju

Za Certilia JWT verifikaciju na Gnosis Chainu postoje tri jasna pristupa, ovisno o prioritetima:

**Najjednostavniji pristup (ako JWT koristi RS256)**: koristiti OpenZeppelin RSA biblioteku (v5.1+) za izravnu on-chain verifikaciju potpisa nad sirovim `header.payload` bajtovima. Ukupni trošak: **~25.000–35.000 gas-a ≈ $0,00 na Gnosisu**. Claimove ekstrahirati off-chain i proslijediti kao zasebne parametre. Ovo je produkcijski spremno, revidirano rješenje koje radi odmah.

**Najpraktičniji pristup (ako JWT koristi ES256/P-256)**: koristiti OpenZeppelin P256 ili FreshCryptoLib za softversku verifikaciju. Trošak: **~69.000–330.000 gas-a ≈ $0,00 na Gnosisu**. FCL s precomputacijom daje najnižu cijenu (~69k gas), ali zahtijeva 3,2M gas za inicijalni deployment.

**Najprivatniji pristup (za selektivno otkrivanje)**: koristiti Noir framework za ZK dokaz valjanosti JWT-a. RSA-2048 proving traje ~262ms na desktopu, P-256 traje ~2–6s. On-chain Groth16/PLONK verifikacija: **~200.000–290.000 gas-a**. Ovaj pristup omogućuje dokazivanje da korisnik ima validan Certilia JWT s određenim atributima (npr. "stariji od 18") bez razotkrivanja punog JWT-a. Uzor za arhitekturu: zkLogin (Sui), Anon Aadhaar, i zk-jwt iz zkEmail ekosustava.

Ključan nalaz: **RSA-2048 izravna verifikacija na Gnosis Chainu košta manje od jednog centa** i stane u manje od 1% bloka. Čak i najskuplji pristup (puna P-256 softverska verifikacija od ~330k gas-a) košta zanemarivo na Gnosisu. Gas cijena nikada nije prepreka — odluka između pristupa trebala bi se temeljiti na zahtjevima za privatnošću, složenosti implementacije i kriptografskom algoritmu koji Certilia koristi u JWT potpisu.