# ERC-4337 na Gnosis lancu — Claude Opus 4.6 izvještaj

**Prompt:** [03-erc4337-gnosis.md](../prompts/03-erc4337-gnosis.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# ERC-4337 i gasless transakcije na Gnosis Chainu

**Gnosis Chain (chain ID 100) ima zrelu ERC-4337 infrastrukturu sa četiri aktivna bundlera i šest paymaster pružatelja usluga.** Za konkretni slučaj — novi korisnici bez xDAI-ja koji trebaju pozvati `claimCertiliaSBT()` sa 2–5M gas troška — postoji više održivih rješenja. **Najjednostavnije i najbrže rješenje je Gelato Relay** sa `sponsoredCallERC2771` metodom, dok ERC-4337 s Pimlico ili Biconomy paymaster-om nudi standardiziraniji, ali složeniji pristup. Ključna prednost Gnosis Chaina je izuzetno niska cijena gasa: sponzoriranje **10.000 korisnika košta samo ~30 xDAI (~$30)** pri konzervativnoj cijeni gasa od 1 Gwei, a pri trenutnim cijenama čak i manje od $5.

---

## EntryPoint ugovori i bundleri na Gnosis Chainu

Obje glavne verzije EntryPoint ugovora potvrđeno su deployane na Gnosis Chain (chain ID 100) i verificirane na GnosisScan-u:

- **EntryPoint v0.6.0**: `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` — aktivan, sa USDC/WETH balansima koji potvrđuju realnu upotrebu
- **EntryPoint v0.7.0**: `0x0000000071727De22E5E9d8BAf0edAc6f37da032` — aktivan, ~161 xDAI balans
- **EntryPoint v0.8.0**: `0x4CE0051A05590c30F23F9E2ae30af4cb89965cf1` — najnovija verzija, podržana od Etherspot Skandha bundlera

Četiri aktivna bundlera pružaju pouzdanu infrastrukturu:

**Pimlico (Alto Bundler)** je najšire integrirani bundler na Gnosis Chainu. RPC endpoint: `https://api.pimlico.io/v2/gnosis/rpc?apikey=API_KEY`. Podržava EntryPoint v0.6 i v0.7. Bundler EOA adresa `0x4337005db25DbAD41Da5692ba1188751eE5D98b6` ima 14.33 xDAI i preko 130.000 transakcija. Pimlico je partner Safe ekosustava i Gnosis Chain službena dokumentacija ga navodi kao ključnog pružatelja.

**Etherspot Skandha** ima najdublju povezanost s Gnosis Chainom jer je dizajniran za Nethermind čvorove (Gnosisov execution klijent). Podržava **v0.6, v0.7 i v0.8** — najširi raspon verzija. RPC: `https://rpc.etherspot.io/v2/100?api-key=API_KEY` za v0.7/v0.8, ili `/v1/100` za v0.6. Defaultni `userOpGasLimit` je **25.000.000**, što je dovoljno za transakcije od 5M gasa. Dostupan je i kao open-source za self-hosting.

**Biconomy** podržava Gnosis mainnet s punom v0.7 (Nexus) podrškom — Bundler, Sponsorship Paymaster, Token Paymaster i Nexus v1.0.2 smart account. Pristup je preko Biconomy dashboarda s jedinstvenim API ključem za sve lance.

**Particle Network** nudi potpuno besplatan bundler bez potrebe za autentikacijom: `https://bundler.particle.network/100`. Podržava v0.6/v0.7 i sve standardne ERC-4337 metode. Idealan za brzo testiranje i prototipiranje.

Alchemy **ne podržava** Account Abstraction na Gnosis Chainu (samo standardni RPC). StackUp je **ugasio** hosted servis i preusmjerio korisnike na Etherspot.

---

## Paymaster pružatelji usluga — usporedba

| Pružatelj | Gnosis podrška | ERC-4337 verzija | Tip paymastera | Cijena / besplatni plan | Ograničenja | Dokumentacija |
|-----------|---------------|-------------------|----------------|------------------------|-------------|---------------|
| **Pimlico** | ✅ Da | v0.6, v0.7 | Verifying PM, ERC-20 PM (USDT, EURe) | 10% pribitak na actualGasCost; testnet besplatno | Tiered API ključ | docs.pimlico.io |
| **Biconomy** | ✅ Da | v0.7 (Nexus) | Sponsorship PM, Token PM | Gas Tank financiran; dashboard | Specifične politike sponzorstva | docs.biconomy.io |
| **Gelato** | ✅ Da | v0.6, v0.7 + Relay | Gas Tank (1Balance, USDC), ERC-20, OnChain PM | USDC gas tank; post-execution billing | Cross-chain USDC deposit | docs.gelato.cloud |
| **Etherspot** | ✅ Da | v0.6, v0.7, v0.8 | Arka PM (sponzorstvo + ERC-20) | Developer plan; open-source self-host | API ključ potreban za hosted | etherspot.io/arka-paymaster |
| **Candide** | ✅ Da | v0.6, v0.7, v0.8 | InstaGas (sponzorstvo), ERC-20 | Dashboard; API ključ | Javne i privatne gas politike | docs.candide.dev |
| **ZeroDev** | ✅ Da | v0.6, v0.7 | Sponsor, ERC-20 (preko Pimlico) | Tiered po broju UserOps | Dashboard projekt | docs.zerodev.app |
| **Alchemy** | ❌ Ne | — | — | — | — | — |
| **StackUp** | ❌ Ugašen | — | — | — | — | — |

**Biconomy omogućuje precizno konfiguriranje sponzorstva**: kroz "Policies" tab na dashboardu možete odabrati **točno koji smart contract i koja funkcija** se sponzorira. Registrirate ABI ugovora i whitelistate `claimCertiliaSBT()` — savršeno za vaš use case. Pimlico također nudi sponsorship policies s per-user limitima i globalnim spending limitima.

**Safe{Core} Gas Station Program** nudi **$250.000+ u gas kreditima** na Gnosis Chainu, koristeći Pimlico kao paymaster infrastrukturu. Ako gradite s Safe smart accountima, možete se prijaviti za besplatne gas kredite.

---

## Vlastiti (custom) paymaster — implementacija i troškovi

Ako komercijalni pružatelji ne zadovoljavaju potrebe ili želite potpunu kontrolu, deployment vlastitog Verifying Paymastera je izvediv. Evo ključnih komponenti:

**Referentne implementacije** su zrele i audited. **eth-infinitism/account-abstraction** repozitorij sadrži `VerifyingPaymaster.sol` u `contracts/samples/` — paymaster koji prihvaća UserOps potpisane od strane ovlaštenog potpisinika (signer). **OpenZeppelin** (v5.x) nudi `PaymasterSigner` — produkcijski spreman ekvivalent s authorization signature uzorkom. Obje implementacije rade s EntryPoint v0.7 na Gnosis Chainu.

**Kako funkcionira Verifying Paymaster**: korisnik konstruira UserOperation i šalje ga vašem backend servisu. Backend provjerava uvjete (je li funkcija `claimCertiliaSBT()`, je li korisnik već sponzoriran), potpisuje hash UserOp-a privatnim ključem, i vraća potpis. Potpis se ugrađuje u `paymasterAndData` polje. EntryPoint na lancu poziva `validatePaymasterUserOp()` koji verificira potpis — ako je valjan, paymaster-ov depozit pokriva gas.

**Implementacija pravila "samo 1 sponzorirana transakcija po adresi"** najlakše se rješava **off-chain** u backend servisu: održavajte bazu podataka sponzoriranih adresa i jednostavno odbijte potpisati UserOp za adrese koje su već koristile sponzorstvo. On-chain alternativa koristi `mapping(address => bool) public hasBeenSponsored` u `_postOp()` funkciji, ali je off-chain pristup jednostavniji i fleksibilniji.

**Ograničavanje na specifičnu funkciju** (`claimCertiliaSBT()`) radi se u backendu: dekodirate `userOp.callData`, ekstrahirate function selector (prvi 4 bajta unutarnjeg poziva), i verificirate da odgovara `bytes4(keccak256("claimCertiliaSBT()"))`. Također provjerite ciljnu adresu ugovora.

**Troškovi financiranja paymastera** (depozit u EntryPoint):

| Scenarij | Cijena gasa | Trošak po UserOp (3M gas) | 1.000 korisnika | 10.000 korisnika |
|----------|------------|---------------------------|-----------------|-------------------|
| Trenutni (ultra-nizak) | ~0.1 Gwei | 0.0003 xDAI | 0.3 xDAI (~$0.30) | 3 xDAI (~$3) |
| Umjeren | 1 Gwei | 0.003 xDAI | 3 xDAI (~$3) | **30 xDAI (~$30)** |
| Visok (zagušenje) | 3 Gwei | 0.009 xDAI | 9 xDAI (~$9) | 90 xDAI (~$90) |

Paymaster mora pozvati `entryPoint.depositTo(paymasterAddress)` s xDAI vrijednošću i `paymaster.addStake(unstakeDelaySec)` za staking (anti-sybil mehanizam). Za 10.000 korisnika pri 1 Gwei, **dovoljno je deponirati ~35 xDAI** (30 za gas + margina za ERC-4337 overhead).

---

## Smart Account tvornice — što je deployano na Gnosisu

ERC-4337 **zahtijeva Smart Contract Account** — obični EOA (Externally Owned Account) ne može koristiti paymaster jer EntryPoint poziva `sender.validateUserOp()`, a EOA nema kôd za izvršavanje tog poziva. Korisnicima se smart account kreira automatski kroz `initCode` u prvom UserOp-u.

**Safe{Core} Protocol** je potpuno aktivan na Gnosis Chainu. Safe v1.4.1 ugovori su deployani, a **Safe4337Module** pruža ERC-4337 kompatibilnost: verzija 0.2.0 za EntryPoint v0.6 i verzija 0.3.0 za EntryPoint v0.7. Safe službena dokumentacija uključuje detaljan vodič specifičan za Gnosis Chain s Pimlico integracijom. Ovo je najzrelija opcija s najvećim ekosustavom.

**Kernel (ZeroDev)** je deployiran na Gnosisu — Gnosis Chain dokumentacija sadrži službeni vodič. Kernel v3.3 Meta Factory je na `0xd703aaE79538628d27099B8c4f621bE4CCd142d5`. ZeroDev dashboard omogućuje kreiranje projekata s Gnosis mrežom, pružajući BundlerRPC i PaymasterRPC. Podržava ERC-7579 plugine, passkey autentikaciju i session keyeve.

**EIP-7702 mijenja pravila igre**: od **30. travnja 2025.** EIP-7702 je aktivan na Gnosis Chainu (Pectra nadogradnja, tjedan dana prije Ethereum mainneta). EIP-7702 omogućuje EOA-ovima da privremeno delegiraju izvršavanje koda smart contractu putem novog tipa transakcije (type `0x04`). To znači da **EOA može ponašati se kao smart contract account** i koristiti paymaster bez trajnog prelaska na smart wallet. eth-infinitism je dodao `Simple7702Account` u v0.8 release upravo za ovaj slučaj. Međutim, inicijalna autorizacijska transakcija i dalje zahtijeva gas, pa je za potpuno gasless onboarding još uvijek potreban relay/sponsor uzorak.

---

## Alternative ERC-4337: meta-transakcije i relay sustavi

Za vaš specifični use case — jedna gasless funkcija za onboarding — ERC-4337 je moćan ali složen alat. Postoje jednostavnije alternative:

**Gelato Relay sa `sponsoredCallERC2771`** je **optimalno rješenje** za ovaj use case. Produkcijski je dokazan na Gnosis Chainu — Gnosis Pay (Gnosisov vlastiti proizvod) koristi Gelato Relay u produkciji. Implementacija zahtijeva: (1) modifikaciju smart contracta da nasljeđuje `ERC2771Context` i koristi `_msgSender()` umjesto `msg.sender`, (2) integraciju Gelato Relay SDK-a u frontend (~20 linija koda), (3) financiranje Gas Tanka preko Gelato dashboarda USDC-om. **Vrijeme implementacije: 1–2 dana.** Nema potrebe za bundlerom, smart accountom, EntryPointom ili paymaster ugovorom.

**EIP-2771 s vlastitim relayerom** daje potpunu kontrolu bez ovisnosti o trećim stranama. Koristite OpenZeppelinov `ERC2771Forwarder.sol` (zamjena za zastarjeli MinimalForwarder). Smart contract mora koristiti `_msgSender()`. Backend relayer (Express.js/Python) prima potpisane poruke od korisnika i šalje ih na Forwarder ugovor. **Vrijeme implementacije: 3–5 dana.** Gas overhead je minimalan (~15k gas, <0.5%).

**OpenGSN v3 je deployiran na Gnosisu** (RelayHub: `0xFD81BeBA349d6a1ffaFb47db59E785c69479ab47`, Forwarder: `0xB2b5841DBeF766d4b521221732F9B618fCf34A87`), ali je **v3.0.0-beta.3** — beta status je rizik za produkciju. Ugovor mora biti GSN-kompatibilan, a relay fee može biti značajan (70% na nekim mrežama).

**Vlastiti relayer** je najfleksibilnija opcija: korisnik potpisuje poruku off-chain, relayer ju šalje, smart contract verificira potpis s `ecrecover`. Zahtijeva dodavanje meta-tx funkcije u ugovor (`claimCertiliaSBTMeta(bytes signature, address user, uint256 nonce, uint256 deadline)`). **Vrijeme implementacije: 3–7 dana.**

| Pristup | Složenost | Izmjene ugovora | Infrastruktura | Gas overhead | Vrijeme impl. | Zrelost na Gnosisu |
|---------|-----------|-----------------|----------------|-------------|----------------|---------------------|
| ERC-4337 full stack | Vrlo visoka | Nije potrebno (novi smart account) | Bundler + Paymaster + EntryPoint | ~250k gas (+8%) | 2–4 tjedna | Aktivno, 4+ bundlera |
| Gelato Relay (ERC2771) | **Niska** | Da (`_msgSender()`) | **Ništa** (Gelato hosted) | ~15k gas (+0.5%) | **1–2 dana** | **Produkcijski dokazano** |
| EIP-2771 + vlastiti relayer | Srednja | Da (`_msgSender()`) | Forwarder + backend server | ~15k gas (+0.5%) | 3–5 dana | Standardni OZ ugovori |
| Custom relayer | Srednje-niska | Da (meta-tx funkcija) | Samo backend server | ~25k gas (+0.8%) | 3–7 dana | N/A (gradite sami) |
| OpenGSN v3 | Srednje-visoka | Da (`_msgSender()`) | RelayHub + Forwarder + PM | ~50–100k gas (+2%) | 1–2 tjedna | Beta, neizvjestan |

---

## Analiza troškova na Gnosis Chainu

Gnosis Chain je **izuzetno jeftin**. GnosisScan Gas Tracker (travanj 2026.) prikazuje cijene gasa **ispod 0.000001 Gwei** — praktički besplatno. Čak i pri konzervativnoj pretpostavci od 1 Gwei, troškovi su zanemarivi.

**ERC-4337 overhead** za transakciju od 3M gasa iznosi otprilike **250.000 gas** (+8.3%). To uključuje: `preVerificationGas` (~50k), validaciju smart accounta (~125k), validaciju paymastera (~40k) i EntryPoint `handleOps` wrapper (~42k). Ukupno: **~3.250.000 gas** za UserOperation umjesto 3.000.000 za regularnu transakciju. U dolarima na Gnosis Chainu, ta razlika je **manja od $0.001 po transakciji** — potpuno zanemariva.

**Usporedba s Ethereum mainnetom** stavlja Gnosis prednost u perspektivu: ista transakcija od 3M gasa na Ethereumu pri 10 Gwei košta **~$58** — to je **~19.000 puta skuplje** od Gnosis Chaina pri 1 Gwei. Za 10.000 korisnika, Ethereum bi koštao **~$583.000** naspram **~$30** na Gnosisu.

Praktična implikacija: **troškovi servera za relay backend** (10–50 $/mjesec) vjerojatno premašuju ukupni trošak gasa za tisuće korisnika. To znači da je najvažniji faktor pri odabiru pristupa **složenost implementacije i održavanja**, a ne ušteda na gasu.

---

## Zaključak — preporuke za implementaciju

Gnosis Chain pruža bogatu ERC-4337 infrastrukturu s **potvrđenim EntryPoint v0.6 i v0.7 ugovorima**, četiri aktivna bundlera (Pimlico, Etherspot, Biconomy, Particle Network) i šest paymaster pružatelja. Međutim, za specifični use case — jednorazovna gasless `claimCertiliaSBT()` transakcija — **potpuni ERC-4337 stack je prevelik alat**.

**Preporučeni pristup prema prioritetu:**

1. **Gelato Relay (`sponsoredCallERC2771`)** — najbrža implementacija (1–2 dana), produkcijski dokazano na Gnosisu, nema potrebe za bundlerom/paymaster-om, minimalna modifikacija ugovora. Jedini nedostatak: ovisnost o Gelato infrastrukturi.

2. **Pimlico/Biconomy ERC-4337 paymaster** — ako želite standardizirani pristup ili planirate proširiti funkcionalnost (batched transakcije, session keys, social login). Biconomy posebno nudi konfigurabilne politike sponzorstva po funkciji. Zahtijeva smart account za svakog korisnika, ali to se rješava automatski kroz `initCode`.

3. **Vlastiti EIP-2771 relayer** — ako želite potpunu neovisnost o trećim stranama. OpenZeppelin `ERC2771Forwarder` + jednostavan backend. Gas overhead je minimalan.

Ključni uvidi koje ne treba previdjeti: **EIP-7702 je aktivan na Gnosisu od travnja 2025.** — to omogućuje EOA korisnicima korištenje paymastera bez trajnog prelaska na smart wallet, što može pojednostaviti budući razvoj. Također, **Safe{Core} Gas Station Program** nudi značajne gas kredite na Gnosisu — vrijedi se prijaviti za besplatno sponzorstvo tijekom faze razvoja i ranog rasta.