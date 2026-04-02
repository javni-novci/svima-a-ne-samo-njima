# eIDAS 2.0 / EUDI novčanik — Claude Opus 4.6 izvještaj

**Prompt:** [05-eidas2-eudi.md](../prompts/05-eidas2-eudi.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# eIDAS 2.0 i EUDI Wallet: Kompletni tehnički vodič za hrvatski kontekst

**Uredba (EU) 2024/1183 stupila je na snagu 20. svibnja 2024., a svaka država članica mora ponuditi barem jedan EUDI novčanik do kraja prosinca 2026.** Ovo znači radikalan pomak s centraliziranog OIDC modela (kakav koristi Certilia Mobile.id) na decentralizirani model verifikabilnih akreditiva putem OpenID4VP protokola. Za sustav koji danas prosljeđuje Certilia JWT tokene na Gnosis blockchain, ključno je razumjeti da EUDI koristi **ECDSA P-256 potpise** (umjesto Ethereumovog secp256k1), da je **SD-JWT VC** primaran format za online prezentaciju, te da je on-chain verifikacija P-256 potpisa danas izvediva na Gnosisu uz ~330k gas korištenjem daimo-eth/p256-verifier biblioteke. Certilia OIDC i EUDI Wallet koegzistirat će **3-5 godina**, no sustav treba već sada planirati dual-path arhitekturu.

---

## 1. Regulatorni vremenski okvir: ključni datumi i rokovi

Uredba eIDAS 2.0 amandira (ne zamjenjuje) izvornu Uredbu (EU) 910/2014. Legislativni proces trajao je od lipnja 2021. do travnja 2024., a implementacija je strukturirana u fazama s jasnim rokovima.

| Datum | Prekretnica |
|-------|------------|
| **20. svibnja 2024.** | Uredba (EU) 2024/1183 stupila na snagu |
| **21. studenoga 2024.** | Rok za prvu seriju provedbenih akata Komisije |
| **4. prosinca 2024.** | Objavljena prva serija provedbenih uredbi (CIR 2024/2977 o PID-u, CIR 2024/2980 o notifikacijama, CIR 2024/2981 o certifikaciji, CIR 2024/2982 o protokolima) |
| **~24. prosinca 2024.** | Provedbeni akti stupili na snagu (20 dana nakon objave u OJ) |
| **18. ožujka 2025.** | Rok za uspostavu postupaka za peer-review eID shema |
| **21. svibnja 2025.** | Druga serija provedbenih akata (kvalificirani certifikati za WAC, sukladnost) |
| **18. rujna 2025.** | Provedbeni akti o interoperabilnom okviru |
| **21. svibnja 2026.** | QTSP-ovi s prethodnim kvalificiranim statusom moraju dokazati usklađenost |
| **~Prosinac 2026.** | **Svaka država članica mora ponuditi barem jedan EUDI Wallet** (24 mjeseca od stupanja na snagu provedbenih akata). Javni sektor mora prihvaćati EUDI novčanike. |
| **~Prosinac 2027.** | **Obvezno prihvaćanje od reguliranih privatnih sektora** (banke, telekomi, energetika, zdravstvo, VLOP-ovi s 45M+ korisnika) |
| **2030.** | EU Digital Decade cilj: 80% građana koristi digitalni identitet |

Postojeće notificirane eIDAS 1.0 sheme (uključujući hrvatsku, notificiranu 2018.) **nastavljaju važiti** tijekom tranzicije. Korištenje EUDI novčanika za građane ostaje **dobrovoljno i besplatno** — države članice ne smiju ograničiti pristup uslugama korisnicima koji ne koriste novčanik.

---

## 2. Gdje je Hrvatska u implementaciji?

Hrvatska ima solidnu postojeću infrastrukturu, ali je kasno formalizirala EUDI-specifični razvojni put. Prema procjeni iz svibnja 2025. bila je među **četiri države članice koje "još nisu započele" EUDI inicijativu** (uz Bugarsku, Maltu i Rumunjsku), no do jeseni 2025. napredak je evidentan.

**Odgovorna tijela u Hrvatskoj:**

| Tijelo | Uloga |
|--------|-------|
| **Ministarstvo digitalne transformacije** | Nadležno ministarstvo, vodi EUDI implementaciju. Igor Ljubi, voditelj Odjela, primarni glasnogovornik. |
| **AKD (Agencija za komercijalnu djelatnost)** | Tehnički implementator/razvijatelj. Razvija **Certilia** mobilni identitetski novčanik. AKD je državna tvrtka koja proizvodi hrvatske identifikacijske dokumente i pruža kvalificirane usluge povjerenja. |
| **Nacionalna radna skupina** | Međuresorna skupina koja izrađuje nacionalni provedbeni akt. |
| **FINA** | Uloga u širem ekosustavu elektroničkih usluga, ali **nije primarno tijelo** za EUDI Wallet. |

Certilia aplikacija (AKD) već funkcionira kao proto-EUDI novčanik s **350.000+ korisnika**, **100.000 dnevnih interakcija** i preko **11 milijuna kvalificiranih udaljenih potpisa**. Podržava eID, zdravstvene iskaznice, studentske kartice, digitalnu vozačku dozvolu i EU Digital Travel Credential (DTC pilot s Finskom na Zračnoj luci Zagreb). Planirano izdavanje certificiranog EUDI novčanika je **Q4 2026.**, no Igor Ljubi je na ENISA Trust Services Forumu u listopadu 2025. izrazio zabrinutost: *"Zabrinut sam zbog certifikacije. Ako nema sheme, nema ni novčanika."*

---

## 3. Tehničke specifikacije EUDI novčanika

Arhitektura EUDI novčanika definirana je u **Architecture Reference Framework (ARF)**, čija je najnovija verzija **v2.8.0** objavljena **2. veljače 2026.** na GitHubu (`eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework`) i na eudi.dev portalu.

### Formati akreditiva: dualni pristup

EUDI Wallet mandatira **dva formata** za PID (Person Identification Data):

| Karakteristika | SD-JWT VC | ISO/IEC 18013-5 (mdoc) |
|----------------|-----------|------------------------|
| **Kodiranje** | JSON (JWT) | CBOR |
| **Primarno korištenje** | Online/udaljena prezentacija | Blizinska prezentacija (BLE, NFC) |
| **Identifikator formata** | `vc+sd-jwt` | `mso_mdoc` |
| **Potpisni mehanizam** | JWS (ES256) | COSE_Sign1 |
| **Selektivno otkrivanje** | Hash-based (IETF SD-JWT) | Individualno soljeni i hashirani atributi (MSO) |
| **PID tip** | `urn:eudi:pid:1` | `eu.europa.ec.eudi.pid.1` |

**W3C Verifiable Credentials Data Model (VCDM) NIJE mandatiran** — ARF eksplicitno navodi da bi dodavanje W3C VCDM kao trećeg formata "uvelo značajne rizike za interoperabilnost, sigurnost i operativnu učinkovitost."

### Kriptografski algoritmi

**Primarni algoritam je ECDSA s P-256 krivuljom (ES256)** — jedini algoritam podržan od strane CC-certificiranih sigurnosnih elemenata (Samsung Knox Vault, Google Titan M2). Privatni ključevi moraju biti pohranjeni u **WSCD-u** (Wallet Secure Cryptographic Device) — tamper-resistant hardver.

- **Podržano po SOG-IS:** ECDSA P-384 (ES384), P-521 (ES512) sa SHA-384/SHA-512
- **Brainpool krivulje:** Registrirane za COSE, ali NISU preporučene
- **RSA:** Nije specificiran kao primarni algoritam
- **Ed25519:** Nije referencirano u ARF-u
- **Post-kvantna kriptografija:** Još nije mandatirana, ali ENISA identificira wallet providere kao prioritetne za rano usvajanje PQC-a

### Selektivno otkrivanje — temeljni zahtjev

**Da, selektivno otkrivanje je obvezno.** Svi PID atributi moraju biti selektivno otkrivljivi. Oba formata implementiraju selektivno otkrivanje putem **soljenih i hashiranih elemenata podataka**:

U **SD-JWT VC**, svaki otkrivljivi claim zamijenjeni je hashom u JWT tijelu, a originalne vrijednosti dostavljaju se kao zasebne "disclosures" (base64url-kodirani JSON nizovi `[salt, claim_name, claim_value]`). Korisnik/novčanik bira koje disclosures uključiti. U **mdoc formatu**, svaki atribut je individualno soljen i hashiran unutar MSO strukture.

Ovaj mehanizam **ne pruža punu unlinkability** — ponovljena prezentacija istog akreditiva omogućuje korelaciju. Mitigacija je **batch issuance** (izdavanje više kopija istog akreditiva s različitim random vrijednostima).

### Zero-knowledge proofs — planirani, ali ne za lansiranje

ZKP podrška je **predviđena regulativom** (Recital 14 navodi: *"Države članice trebale bi integrirati različite privacy-preserving tehnologije, poput zero knowledge proofs, u EUDI Wallet"*), ali **neće biti mandatirana pri lansiranju**. ARF Discussion Topic G definira visoko-razinske zahtjeve (ZKP_01 do ZKP_08) i dva pristupa u razvoju:

- **BBS+ / BBS# multi-message potpisi** (TS14) — omogućuju učinkovito selektivno otkrivanje s konstantnom veličinom potpisa, snažno preporučeni od vanjskih kriptografa
- **Aritmetički krugovi (zk-SNARKs)** (TS13) — općenamjenski ZKP-ovi, veća složenost

Na GitHubu EU već postoji Swift biblioteka `av-lib-ios-longfellow-zkp` za ZKP generiranje i verifikaciju mdoc akreditiva.

### Protokoli za prezentaciju i izdavanje

| Funkcija | Protokol |
|----------|----------|
| **Udaljena prezentacija** | OpenID for Verifiable Presentations (**OpenID4VP** 1.0) + W3C Digital Credentials API |
| **Blizinska prezentacija** | ISO/IEC 18013-5 (BLE, NFC, WiFi Aware) |
| **Izdavanje akreditiva** | OpenID for Verifiable Credential Issuance (**OpenID4VCI**) |
| **Upit za akreditive** | DCQL (Digital Credentials Query Language) |
| **Provjera opoziva** | Status List Token |

---

## 4. PID atributi i jedinstveni identifikator

### Obvezni PID atributi (CIR 2024/2977)

`family_name`, `given_name`, `birth_date`, `birth_place`, `nationality` — plus obavezni metapodaci: `expiry_date`, `issuing_authority`, `issuing_country`.

### Opcionalni atributi

`resident_address`, `resident_country`, `resident_city`, `personal_administrative_number`, `portrait`, `family_name_birth`, `given_name_birth`, `sex`, `email_address`, `mobile_phone_number`, `age_over_18`, `age_in_years` i drugi.

### Jedinstveni identifikator osobe

**`personal_administrative_number`** je opcionalni PID atribut koji je "jedinstven među svim administrativnim brojevima osoba koje izdaje pružatelj identifikacijskih podataka." U Hrvatskoj to odgovara **OIB-u** (Osobni identifikacijski broj). **Ne postoji EU-wide jedinstveni identifikator** — jedinstvenost je ograničena na PID Provider/državu članicu. Format je `tstr` (CBOR) odnosno `string` (JSON), bez specificirane strukture na razini EU.

### Blockchain/DLT u ARF-u

**Blockchain NIJE sastavni dio EUDI Wallet arhitekture.** Trust model temelji se na **Trusted Lists**, X.509 PKI i pristupnim certifikatima. Međutim, eIDAS 2.0 uvodi **Qualified Electronic Ledger** kao novu kvalificiranu uslugu povjerenja (CIR 2025/2531), a EBSI postoji kao potencijalna DLT infrastruktura.

---

## 5. Pilotni projekti i hrvatski sudjelovanje

### EU Large Scale Pilots — pregled

| Pilot | Koordinator | Države | Fokus | Hrvatska? |
|-------|-------------|--------|-------|-----------|
| **POTENTIAL** | Francuska/Njemačka | 19 MS + Ukrajina | eGov, banke, SIM, mDL, eSign, ePrescription | Neizvjesno |
| **EWC** | Švedska/Finska | Svih 27 MS | DTC, plaćanja, LPID | **DA** — putem Identyuma |
| **NOBID** | Norveška | 8 nordijsko-baltičkih zemalja | Autorizacija plaćanja | NE |
| **DC4EU** | Španjolska | 25 zemalja | Obrazovne diplome, socijalno osiguranje, EBSI | NE |
| **APTITUDE** (2. val) | Francuska | 11 MS + Ukrajina | Putovanje, registracija vozila, interoperabilnost | NE |
| **WE BUILD** (2. val) | Nizozemska | 27 zemalja | B2B, KYC/KYB, pravne osobe | Vjerojatno DA |

**Identyum** (Identity Consortium d.o.o., Varaždin) opisuje se kao *"jedini ID Wallet provider iz Hrvatske"* u EWC konzorciju. AKD je proveo **DTC pilot** (Digital Travel Credential) na Zračnoj luci Zagreb u suradnji s finskim partnerima — proračun projekta **2,3 milijuna EUR** (hrvatski dio 1,1M, 95% EU-financirano), s **14.000+ putnih dokumenata** dodanih u Certiliu.

### Blockchain + EUDI projekti

- **DC4EU eksplicitno koristi EBSI** za trust registre, DPKI i verifikaciju akreditiva u obrazovanju
- **EBSI-VECTOR** projekt olakšava prijelaz s EBSI-kompatibilnih novčanika na EUDI-certificirane
- **Blockstand EU "Web3 Passport"** (2025.) — arhitektura za ZK on-chain atestacije s EBSI backbone-om, nullifier-based Sybil zaštita
- **"Know Your Contract"** (arXiv 2025.) — proširenje eIDAS povjerenja na javne blockchaine kriptografskim vezanjem smart contracta na kvalificirane elektroničke pečate
- **IOTA Web3 ID** odabran za European Blockchain Sandbox (2. kohorta)

### Dostupni open source kod

Glavni GitHub repozitorij: **github.com/eu-digital-identity-wallet** sadrži:

- `eudi-app-android-wallet-ui` / `eudi-app-ios-wallet-ui` — prototip EUDI Wallet aplikacija
- `eudi-lib-android-wallet-core` — Android Wallet Core SDK (Maven: `eu.europa.ec.eudi:eudi-lib-android-wallet-core`)
- `eudi-lib-ios-wallet-kit` — iOS Wallet Kit
- `eudi-lib-jvm-openid4vci-kt` — JVM/Kotlin OpenID4VCI biblioteka
- `eudi-srv-wallet-provider` — Wallet Provider servis s OpenId4VCI 1.0, Swagger UI
- `eudi-doc-architecture-and-reference-framework` — ARF dokumentacija

Dodatno, EWC konzorcij objavljuje RFC-ove na `github.com/EWC-consortium/eudi-wallet-rfcs`.

---

## 6. Utjecaj na Certiliu i postojeći sustav

### Certilia OIDC neće nestati odmah

**Certilia Mobile.id i EUDI Wallet koegzistirat će tijekom tranzicijskog perioda od minimalno 3-5 godina.** Uredba ne ukida postojeće notificirane sheme, a alternativni put mora postojati za korisnike bez novčanika. Prema konzultantu REELIANT: *"Prilagodite produkcijske tokove da prihvate prve EUDI atestacije prije roka u prosincu 2026. Održavajte koegzistenciju s postojećim mehanizmima tijekom tranzicije."*

Štoviše, AKD pozicionira Certiliu kao **osnovu za hrvatski EUDI novčanik** — direktor AKD-a Jure Sertić izjavio je da "Certilia ima sve preduvjete za budući EU digitalni novčanik."

### OIDC vs. OpenID4VP — fundamentalna promjena paradigme

Ovo je **najkritičnija tehnička promjena** za korisnikov sustav. EUDI Wallet koristi OpenID4VP, ne klasični OIDC:

| Aspekt | OIDC (Certilia danas) | OpenID4VP (EUDI Wallet) |
|--------|------------------------|--------------------------|
| **Model** | Centralizirani: RP→IdP→RP | Decentralizirani: Holder→Verifier |
| **IdP u trenutku autentifikacije** | DA — IdP autenticira korisnika u realnom vremenu | NE — Issuer nije kontaktiran pri prezentaciji |
| **Format tokena** | ID Token (JWT, potpisan od IdP) | Verifiable Presentation (SD-JWT VC ili mdoc) |
| **Response type** | `id_token` ili `code` | `vp_token` |
| **Selektivno otkrivanje** | Ograničeno (scopes) | Nativno (per-claim disclosure) |
| **Privatnost** | IdP vidi svaku autentifikaciju | Issuer nije uključen pri prezentaciji |
| **Trust model** | RP vjeruje IdP endpointu | Verifier provjerava kriptografski potpis Issuera u Trusted List |

**OpenID4VP flow:** Verifier kreira Authorization Request (`response_type=vp_token`, `presentation_definition` ili `dcql_query`, `nonce`), novčanik prikazuje korisniku tražene claims, korisnik odobrava selektivno otkrivanje, novčanik vraća `vp_token` s Key Binding JWT-om, verifier validira potpis issuera, key binding, nonce, status akreditiva i trust chain.

### Potrebne promjene u arhitekturi

Za dual-path podršku (Certilia OIDC + EUDI OpenID4VP):

1. **Dodati OpenID4VP Verifier komponentu** — zasebni servis koji generira Authorization Request, procesira `vp_token`, validira SD-JWT VC, provjerava issuer potpise (P-256) i Trusted Issuer Registry
2. **Protocol Router** — na ulaznoj točki detektirati koristi li korisnik Certilia OIDC ili EUDI Wallet i usmjeriti na odgovarajući flow
3. **Normalizacija claimova** — oba puta moraju proizvesti uniformirani interni set claimova za downstream logiku (uključujući blockchain forwarding)
4. **WSO2 Identity Server** — ne podržava nativno OpenID4VP Verifier ulogu; opcije su: (a) zasebni verifier servis (walt.id SDK, Gataca), (b) čekati WSO2 podršku, (c) koristiti EU referentnu implementaciju

---

## 7. Blockchain kompatibilnost: on-chain verifikacija EUDI akreditiva

### P-256 potpisi na Gnosis Chainu

EUDI akreditivi koriste **ECDSA P-256 (secp256r1)** potpise, dok Ethereum/Gnosis nativno koristi **secp256k1**. Verifikacija P-256 na EVM-u:

| Opcija | Gas trošak | Status |
|--------|-----------|--------|
| **daimo-eth/p256-verifier** (fallback ugovor) | **~330.000 gas** | Deployano na bilo koji EVM chain na adresi `0xc2b78104907F722DABAc4C69f826a522B2754De4` |
| **RIP-7212 precompile** | ~3.450 gas | Usvojeno na L2 lancima (Polygon, Optimism, Base, Arbitrum) |
| **EIP-7951 precompile** | ~6.900 gas | Predloženo za Ethereum Fusaka — JOŠ NIJE na Gnosisu |

**Praktično za Gnosis Chain danas:** Koristiti `daimo-eth/p256-verifier` s ~330k gas po verifikaciji. Pri tipičnim Gnosis cijenama gasa (1-2 gwei), to iznosi **~0,0003-0,0007 xDAI (~$0,0003-$0,0007)** — potpuno prihvatljivo. Kada Gnosis usvoji P-256 precompile (nakon Fusaka alignmenta), isti ugovor automatski koristi precompile i pada na **~3.400-6.900 gas**.

```solidity
import "p256-verifier/P256.sol";

bytes32 hash;  // SHA-256 hash potpisanog JWT sadržaja
uint256 r, s;  // ECDSA komponente potpisa
uint256 x, y;  // P-256 javni ključ koordinate

bool valid = P256.verifySignature(hash, r, s, x, y);
```

### SD-JWT on-chain: hibridni pristup

**Potpuni SD-JWT parsing na chainu je nepraktičan** (JSON parsing, base64url dekodiranje, disclosure hash verifikacija — previsok gas trošak). Preporučeni **hibridni pristup:**

```
EUDI Wallet → [OpenID4VP] → Verifier Backend (off-chain)
    ↓
Backend validira puni SD-JWT (disclosure hashevi, issuer cert chain, opoziv)
    ↓
Backend ekstrahira minimalne claimove + proof potpisa
    ↓
Backend šalje na Smart Contract:
  - hash(claims), issuerSignature(r,s), issuerPubKey(x,y)
    ↓
Smart Contract verificira P-256 potpis (via p256-verifier)
```

### Sybil zaštita: nullifier iz OIB-a

EUDI PID atribut `personal_administrative_number` (u Hrvatskoj = **OIB**) može služiti kao temelj za Sybil zaštitu. **Direktno postavljanje OIB-a na blockchain je GDPR kršenje** (osobni podatak na nepromjenjivom javnom ledgeru). Umjesto toga:

**Jednostavniji pristup (bez ZK):**
```
nullifier = keccak256(abi.encodePacked(personal_administrative_number, service_id))
```
Backend verificira SD-JWT off-chain, izračuna nullifier, šalje samo nullifier + dokaz potpisa na smart contract. Ugovor provjerava jedinstvenost nullifiera (mapping korištenih nullifiera). **Kompromis:** backend saznaje OIB, ali on-chain podaci su pseudonimni.

**Napredniji pristup (s ZK):** Korisnik generira ZK proof da posjeduje validan EUDI PID potpisan od trusted issuera, da njegov `personal_administrative_number` hashira na specifični nullifier, te da nullifier nije prethodno korišten — **bez otkrivanja stvarnog OIB-a**. Alati: Iden3/Privado ID (Polygon ID), Circom krugovi, ili budući EUDI ZKP mehanizmi (BBS+, Longfellow).

---

## 8. Konkretne preporuke za korisnikov projekt

### Kratkoročno (2026.)

- **Održavati Certilia OIDC integraciju** — funkcionirat će još minimalno 3-5 godina
- **Implementirati OpenID4VP Verifier** kao zasebni servis uz WSO2 IS, koristeći walt.id SDK ili EU referentnu implementaciju
- **Deployati daimo-eth/p256-verifier** na Gnosis Chain za P-256 verifikaciju (~330k gas)
- **Dizajnirati normalizacijski sloj** koji mapira claimove iz oba izvora (OIDC `sub`/`name`/OIB → OpenID4VP `family_name`/`given_name`/`personal_administrative_number`) u uniformirani format

### Srednjoročno (2027.)

- **Implementirati nullifier-based Sybil zaštitu** koristeći `personal_administrative_number` → hash → on-chain provjera jedinstvenosti
- **Pratiti Gnosis Fusaka upgrade** za nativni P-256 precompile (pad s 330k na ~3.4-6.9k gas)
- **Testirati s hrvatskim EUDI Wallet-om** kada bude dostupan (planirano Q4 2026.)
- **Razmotriti EBSI integraciju** za trust registry korištenje

### Dugoročno (2028.+)

- **Implementirati ZK proof verifikaciju** kada EUDI ARF formalizira ZKP podršku (BBS+, zk-SNARKs)
- **Prijeći potpuno na OpenID4VP** kada Certilia OIDC bude deprecated
- **Istražiti on-chain verifikaciju bez backendu** koristeći ZK bridge pattern za potpunu decentralizaciju

### Ključni rizici

- **Certifikacijska shema za EUDI Wallet u Hrvatskoj** još nije uspostavljena — ovo je najveći rizik za Q4 2026. rok
- **WSO2 IS ne podržava OpenID4VP** — potreban zasebni verifier servis
- **P-256 precompile na Gnosisu** ovisi o Fusaka alignmentu — vremenski okvir neizvjestan
- **`personal_administrative_number` je opcionalni atribut** — Hrvatska mora odlučiti hoće li ga uključiti u PID (gotovo sigurno da, jer OIB je fundamentalan za hrvatsku administraciju)

---

## Zaključak: što je zapravo novo i važno

EUDI Wallet nije samo nadogradnja eIDAS-a — to je **paradigmatski pomak od centraliziranih identitetskih providera prema korisničkoj kontroli nad akreditivima**. Za sustave koji koriste blockchain, ovo otvara mogućnost izravne kriptografske veze između EU identiteta i on-chain logike, bez posrednika. Ključni uvid je da **SD-JWT VC format s P-256 potpisima i selektivnim otkrivanjem** pruža tehničku osnovu za blockchain-kompatibilnu identifikaciju, ali zahtijeva hibridni off-chain/on-chain pristup. Potpuna on-chain verifikacija bez backendu bit će praktična tek s ZKP integracijom koju ARF planira nakon lansiranja. Hrvatska je u specifičnoj poziciji gdje AKD/Certilia ima infrastrukturu koja može evoluirati u EUDI Wallet, ali certifikacijska neizvjesnost čini Q4 2026. rok rizičnim. Dual-path arhitektura (OIDC + OpenID4VP) s P-256 verifikacijom na Gnosisu je najsigurnija strategija za sustav koji mora biti spreman za obje ere.