# ERC-4337 na Gnosis lancu — Perplexity izvještaj

**Prompt:** [03-erc4337-gnosis.md](../prompts/03-erc4337-gnosis.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

Na Gnosis lancu su danas dostupni i EntryPoint v0.6 i v0.7, a oko njih postoji sasvim zrela ERC‑4337 infrastruktura (bundleri + paymasteri) od Pimlica, Biconomyja, Particle Networka i ZeroDeva, dok Gelato i OpenGSN nude meta‑transakcije (EIP‑2771) kao jednostavniju alternativu za tvoj use‑case “jedno mintanje po korisniku bez xDAI”. Uz tipičan gas na Gnosisu (~2 gwei) čak i teška 3M–5M gas operacija košta ispod jednog centa po korisniku, pa je sponzoriranje 1k–10k korisnika red veličine par do nekoliko desetaka xDAI.[1][2][3][4][5][6][7][8]

***

## 1. ERC‑4337 infrastruktura na Gnosisu

### EntryPoint ugovori

Na Gnosis lancu su verificirane obje standardne ERC‑4337 EntryPoint verzije:

- **EntryPoint v0.6.0**  
  - Adresa: `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` (isti “globalni” v0.6 kao na drugim EVM lancima).[9][10]
  - Na GnosisScan‑u označen kao “Entry Point 0.6.0”, sa stotinama tisuća transakcija preko 29 chainova, uključujući Gnosis.[10]

- **EntryPoint v0.7.0**  
  - Adresa: `0x0000000071727de22e5e9d8baf0edac6f37da032` (verificirani “EntryPoint 0.7.0” na GnosisScan‑u).[page:0x0000000071727de22e5e9d8baf0edac6f37da032]  
  - To je nova verzija iz eth‑infinitism repozitorija (solc 0.8.23, Paris EVM, ERC‑165, ReentrancyGuard, itd.).[page:0x0000000071727de22e5e9d8baf0edac6f37da032]

Biblioteke poput `permissionless.js` i thirdweb SDK‑ova imaju konstante `ENTRYPOINT_ADDRESS_V06` i `ENTRYPOINT_ADDRESS_V07` koje upravo odgovaraju ovim adresama i koriste se i na Gnosisu. Gnosis dokumentacija i primjer s Pimlicom eksplicitno koriste `ENTRYPOINT_ADDRESS_V06` za Gnosis lanac.[11][12][13]

### Bundleri na Gnosisu (proizvodni)

S aktualnih dokumenata i Gnosis vodiča može se identificirati nekoliko praktično korištenih bundlera na chainu 100:

1. **Pimlico Alto Bundler (gnosis)**  
   - Provider: Pimlico (AA infra specijalist).  
   - RPC (bundler):  
     - Auth: `https://api.pimlico.io/v1/gnosis/rpc?apikey=API_KEY`[12]
     - Public (bez API key‑a, s nižim QoS): `https://api.pimlico.io/v1/gnosis/rpc` (putem generičkog formata `…/v1/gnosis/rpc`).[14]
   - Podržani EntryPointi: v0.6 putem `ENTRYPOINT_ADDRESS_V06` (službeni primjeri za Safe na Gnosisu), Pimlico općenito podržava i v0.7, ali Gnosis primjer još koristi v0.6.[12][14]
   - Rate limit / ograničenja:  
     - Besplatan plan: 1M “credits” mjesečno (~1.300 UserOp‑ova, ~950 ako su sponzorirani), 500 req/min.[15][16]
     - Pay‑as‑you‑go: 10M credits mjesečno, 5.000 req/min, dodatni credits po 1 USD/100k; surcharge 10% na iznos sponzoriranog gasa za Verifying/ERC20 paymastere na mainnetima.[16][15]
     - Maksimalni gas po UserOp‑u nije eksplicitno dokumentiran; u praksi se orijentiraš na limit bloka Gnosisa i vlastito testiranje.

2. **Biconomy Bundler (gnosis)**  
   - Provider: Biconomy (MEE / ERC‑4337 infra).  
   - Podržan chain: **Gnosis Mainnet i Chiado** s punom MEE podrškom.[17]
   - ERC‑4337 podrška:  
     - Tablica “ERC‑4337 Infra Support” eksplicitno navodi Gnosis Mainnet s **Bundler = ✅**, Sponsorship Paymaster = ✅, Token Paymaster = ✅ na EntryPoint v0.7 (Nexus v1.0.2).[17]
     - Dodatna tablica navodi Gnosis i za EntryPoint v0.6 u sklopu V2 Smart Accounts.[17]
   - RPC pristup: bundler URL i paymaster URL dobivaš iz Biconomy dashboarda (format tipa `https://bundler.biconomy.io/api/v2/{chainId}/{apiKey}`), dok SDK (`@biconomy/bundler`) uzima `bundlerUrl`, `chainId` i `entryPointAddress` kao parametre.[18][19]
   - Ograničenja: nije javno specificiran max gas po UserOp‑u; Biconomy navodi podršku za 70+ EVM chainova i multi‑chain izvršenje, pa se najveća ograničenja najčešće vežu uz tvoj plan i interne QoS limite.[20]

3. **Particle Network Bundler (gnosis)**  
   - Provider: Particle Network (Smart Wallet‑as‑a‑Service).  
   - Gnosis podrška: Particle nativno integrira Gnosis mainnet i testnet za ERC‑4337 (SimpleAccount, Biconomy, CyberConnect kao opcije smart accounta).[page:particle-network][1]
   - Bundler: “Particle Network Bundler” je dio AA SDK‑a, kojim se konstruiraju i šalju UserOp‑ovi.[page:particle-network][21]
   - Paymaster: na testnetima koristi vlastiti Particle Paymaster; na mainnetima je **Biconomy default paymaster** (ali možeš i direktno koristiti Particle Omnichain Paymaster preko `https://paymaster.particle.network`).[page:particle-network][22][23]
   - Ograničenja: gasless je besplatan na testnetima; na mainnetima trošiš USDT depozit i sam definiraš pravila kroz webhook (npr. “samo prva transakcija po addressu”).[23][22]

4. **ZeroDev / Kernel Bundler (gnosis)**  
   - Provider: ZeroDev, kernel‑based smart accounts.  
   - Gnosis podrška: Gnosis AA vodič eksplicitno kaže da u ZeroDev dashboardu možeš izabrati **network chain = Gnosis**, nakon čega dobivaš `BundlerRPC` i `PaymasterRPC` vrijednosti.[7]
   - Bundler/Paymaster: ZeroDev project dashboard ti daje konkretne RPC URL‑ove za bundler i paymaster prilagođene tom projektu.[7]
   - Podržane verzije EntryPointa: ZeroDev je fokusiran na v0.6 i v0.7, ali Gnosis specifični vodič ne pinna verziju; biraš preko SDK konfiguracije (Kernel je kompatibilan s oba).[24][7]
   - Ograničenja: nisu javno specificirani; konfiguracija i limiti (gas, broj UserOp‑ova) ovise o planu i dogovoru.

Osim ovih, MetaMask Smart Accounts Kit nudi gotov AA stack (smart account + bundler + gas manager) i eksplicitno navodi Gnosis Chain kao podržanu mrežu, ali bundler i gas manager su “skriveni” iza njihovog SDK‑a, ne kao otvoreni generički bundler endpoint.[25]

***

## 2. Paymaster pružatelji na Gnosisu

Tražena tablica (fokus na lancu 100):

| Pružatelj | Gnosis podrška | ERC‑4337 verzija | Cijena | Ograničenja | URL dokumentacije |
|-----------|----------------|------------------|--------|-------------|-------------------|
| **Pimlico** | Da – `gnosis` mainnet u listi podržanih chainova; službeni Gnosis vodič “Building on Safe AA with Pimlico” koristi Pimlico bundler + paymaster na Gnosisu.[12][14] | Primjeri za Gnosis koriste **EntryPoint v0.6** (`ENTRYPOINT_ADDRESS_V06`); Pimlico općenito podržava i v0.7 preko istog API‑ja.[12][14] | Free plan: 0 USD/mj, 1M credits/mj (~1.300 UserOp‑ova), samo testnet paymaster; Pay‑as‑you‑go: 0 USD/mj, 10M credits/mj, zatim ~0,0075 USD/UserOp (ili 0,0105 USD kod sponzoriranih) + 10% surcharge na sponzorirani gas na mainnetima.[15][16] | Free plan: bez mainnet paymastera; rate limit 500 req/min (free) i 5.000 req/min (pay‑as‑you‑go); preVerification/verification gas i max gas po UserOp‑u ovise o simulaciji, nema javnog hard‑capa.[15] | https://docs.pimlico.io/ i Gnosis vodič: https://docs.gnosischain.com/technicalguides/account-abstraction/Safe%20and%20supported%20AA%20infra%20providers/pimilico[12][14] |
| **Biconomy** | Da – Gnosis Mainnet i Chiado navedeni kao podržani u MEE tablici i u listi “Bundlers & Paymaster supported networks”.[17] | Za Gnosis Mainnet: pun **EntryPoint v0.7** stack (Nexus Smart Account v1.0.2) s Bundlerom, Sponsorship PM i Token PM = ✅; dodatno podržava i EntryPoint v0.6 u starijem V2 SA modu.[17] | Pricing je “enterprise‑like”; nema javne detaljne cjenike, ali Paymaster podržava sponzorirani mod (gas plaćaš u pozadini) i ERC‑20 mod; integrira se preko dashboarda i paymaster URL‑a.[26][27][28] | Moraš kreirati projekt na dashboardu i dobiti bundlerUrl + paymasterUrl; uvjeti sponzoriranja (npr. “first 20 tx”, “drže NFT”) definiraju se kroz pravila na dashboardu.[29][30] | https://account-abstraction-docs.biconomy.io/ i https://docs.biconomy.io/contracts-and-audits/supported-chains[31][17][26] |
| **Gelato Relay** | Da – Safe nativno koristi Gelato Relay za gasless transakcije na Gnosisu; Gnosis vodič “Building on Safe AA with Gelato” koristi **Gelato 1Balance** na Gnosisu.[32][2] | Gelato Relay na Gnosisu funkcionira kao **vlastiti relay/meta‑tx sustav**, ne kao ERC‑4337 EntryPoint bundler; integrira se preko Safe Relay Kitu, ne preko `eth_sendUserOperation`.[2] | Cjenik Relay‑a je po broju zahtjeva (1.000/mj free, 50k i 200k u plaćenim planovima) + **gas premium ~20%** na free planu za sve mreže.[17][33] | Potreban 1Balance račun (top‑up USDC na Polygonu, pa se troši na Gnosisu); ograničenja volumena i rate‑limita ovise o tarifi; “SyncFee” mod dozvoljava da korisnik plati gas iz vlastitog Safe‑a iako nema xDAI na EOA‑u.[32][2] | https://docs.gelato.cloud/web3-services/relay i Gnosis vodič: https://docs.gnosischain.com/technicalguides/account-abstraction/Safe%20and%20supported%20AA%20infra%20providers/gelato[32][2][17] |
| **Alchemy (Account Kit)** | Nema dedicated bundlera/paymastera za Gnosis, ali **SDK radi na bilo kojem EVM‑u** ako mu daš bundler URL; službeni primjer navodi podršku za Ethereum mainnet, Polygon i Sepolia.[34][35] | Account Kit trenutno targetira uglavnom EntryPoint v0.6 mreže (Ethereum + L2) gdje Alchemy hosta svoj Rust bundler; za Gnosis bi trebao koristiti tuđi bundler (npr. Pimlico/Biconomy) s njihovim SDK‑om.[36][37][35] | Nema javnog cjenika samo za AA; koristiš standardni Alchemy pricing (po requestu) + zaseban “gas policy” ako koristiš njihov Gas Manager; za Gnosis ne nude prvi‑razrednu podršku.[36][34] | Ograničenja: nema njihovog hostanog bundlera na Gnosisu, pa ne dobivaš “managed” Gas Manager; i dalje možeš koristiti AA SDK s trećim bundlerom i vlastitim paymasterom.[37][35] | https://docs.alchemy.com i Account Kit FAQ: https://www.alchemy.com/support/what-are-the-supported-chain-for-account-kit[36][35] |
| **StackUp** | Ne – lista podržanih mreža ograničena je na Ethereum, Arbitrum, Optimism, Avalanche, Polygon, Base, BNB; Gnosis se ne spominje.[38] | Njihov Go‑based bundler podržava ERC‑4337, ali **nema produkcijskog deploymenta na Gnosisu** prema službenoj dokumentaciji.[38][39] | Nudili su managed bundler/paymaster za podržane mreže; za Gnosis bi morao self‑hostati open‑source bundler i sam ga povezati na EntryPoint.[39] | Nema javnog paymastera niti infrastrukture za chainId 100. | https://docs.stackup.fi/docs/supported-networks i https://github.com/stackup-wallet/stackup-bundler[38][39] |
| **Particle Network** | Da – Gnosis Mainnet i Testnet eksplicitno navedeni; postoji demo “gnosis‑aa‑connect”.[page:particle-network][40] | Native ERC‑4337 (SimpleAccount + drugi account tipovi); Particle koristi vlastiti Bundler i Paymaster, a na mainnetima po defaultu delegira paymaster Biconomyju.[page:particle-network][21] | Testnet: gasless sponzoriranje je **besplatno**; Mainnet: moraš depositati USDT na Ethereum/BNB u Particle Omnichain Paymaster, koji se automatski pretvara u native gas token na ciljnom chainu.[22][23][40] | Sponsorship pravila (npr. “prva transakcija po adresi”) definiraš preko web‑hookova (`before_paymaster_sign`), pa možeš jako fino kontrolirati tko dobiva besplatan gas.[22][23] | Gnosis vodič: https://docs.gnosischain.com/technicalguides/account-abstraction/particle-network i Particle Paymaster docs: https://developers.particle.network/guides/aa/paymaster[page:particle-network][22][23] |
| **Safe{Core} Gas Station (preko Pimlica)** | Da – program eksplicitno navodi 50k USD kredita za Gnosis Chain za projekte koji koriste Safe + 4337 modul + Pimlico.[41] | Safe smart account koristi ERC‑4337 modul preko EntryPointa (v0.6 danas u većini primjera) i Pimlico kao preferiranog paymastera/bundlera.[41][12] | Program daje besplatne gas kredite (grants) koji se troše u Pimlico paymasteru; iznos i trajanje ovise o tome koliko dobiješ (npr. dio od ukupno 50k USD na Gnosisu).[41] | Ograničeno na projekte koji integriraju Safe Core SDK / contracts i koriste Pimlico; nije generički paymaster za bilo koji ugovor, nego grant‑program koji ti smanjuje trošak korištenja Pimlica.[41] | Objave: https://safefoundation.org/blog/making-every-app-gasless-launching-the-safe-core-multi-chain-gas-station[41] |
| **OpenGSN (EIP‑2771)** | Da – Gnosis Chain je službeno podržan; postoje mrežni parametri, RelayHub, Forwarder i primjer Paymastera (“Accept‑Everything”).[3][42] | OpenGSN je **EIP‑2771 meta‑transakcijski sustav**, ne ERC‑4337, ali rješava isti problem – korisnik bez xDAI može slati transakcije preko relayera + paymastera.[43] | Softver je open‑source; plaćaš vlastiti gas na paymaster računu; mreža ima i javne relayere, ali za ozbiljan volumen u praksi se self‑hosta.[3][43] | Forwarder i Paymaster adrese za Gnosis su zadane; ograničenja su uglavnom koliko si spreman stake‑ati i koliki throughput tvoj relayer može podnijeti.[3][42] | Gnosis stranica: https://docs.opengsn.org/networks/gnosis/gnosis.html i opći docs: https://docs.opengsn.org[3][43] |

***

## 3. Vlastiti ERC‑4337 Paymaster

Ako ti nijedan komercijalni provider ne odgovara ili želiš punu kontrolu, možeš deployati vlastiti paymaster.

### Dostupne implementacije

- **eth‑infinitism/account‑abstraction**  
  - Sadrži `BasePaymaster`, `DepositPaymaster`, `TokenPaymaster` i **`VerifyingPaymaster`** kao referentne implementacije u `contracts/samples/`.[44][45]
  - `VerifyingPaymaster` točno odgovara tvom use‑caseu: off‑chain servis odlučuje hoće li sponzorirati UserOp, potpisuje ga, a on‑chain paymaster provjerava potpis i sponzorira gas.[46][47][48]

- **OpenZeppelin “community contracts” paymasters**  
  - OpenZeppelin ima dokumentaciju i pomoćne kontrakte za Paymastere unutar svoje AA ponude; fokus su sigurnosni patterni i integracija s njihovim Account kontraktima.[45][49][50]

- **Pimlico ERC20 Paymaster (open‑source)**  
  - Pimlicov repo `erc20-paymaster` je napredniji primjer TokenPaymastera koji dopušta plaćanje gasa u ERC‑20 tokenima uz orakul za cijene.[51]

- **Primjeri servera**  
  - Postoje referentne implementacije “simple paymaster servera” koje koriste `VerifyingPaymaster.sol` i standardizirani JSON‑RPC (`pm_sponsorUserOperation`), npr. imelon2/simple‑paymaster‑server.[47][48]

### Složenost deploya

- On‑chain dio:  
  - Iz perspektive Solidityja, deploy vlastitog VerifyingPaymastera na Gnosis je relativno jednostavan – prilagodiš sample iz eth‑infinitism repoa, u konstruktorski parametar staviš `EntryPoint` (npr. v0.6 ili v0.7) i dodaš svoju logiku (npr. whitelist, mappingi).[44][45]
  - Moraš uplatiti depozit na paymaster preko EntryPointa (`depositTo`) da bi imao čime plaćati gas.[page:0x0000000071727de22e5e9d8baf0edac6f37da032]

- Off‑chain dio:  
  - Treba ti mali servis (Node/Go/…) koji prima UserOp, radi sve off‑chain provjere (rate‑limit, identitet, KYC ako treba) i potpisuje paket koji `VerifyingPaymaster` zna verificirati.[48][46][47]
  - Taj servis izlaže `pm_sponsorUserOperation` API kako bi tvoj dApp mogao automatski “dopuniti” `paymasterAndData` prije slanja UserOp‑a bundleru.[48]

### Pravilo “samo 1 sponzorirana transakcija po adresi”

To možeš implementirati na dva mjesta:

1. **Direktno u Paymasteru (on‑chain)**  
   - U paymaster kontrakt dodaš mapping:  
     `mapping(address => bool) public hasSponsored;`  
   - U `_validatePaymasterUserOp` (ili overrideu) parsiraš `userOp.callData` da dođeš do `msg.sender` odnosno adrese smart accounta / EOA‑a koji želi mintati.  
   - Ako je `hasSponsored[user] == true`, vratiš `SIG_VALIDATION_FAILED` ili revert‑aš, čime sprječavaš drugo sponzoriranje.[45][46]
   - Ako je `false`, označiš na `true` i sponzoriraš gas.

2. **U ciljanom SBT ugovoru**  
   - U `claimCertiliaSBT()` možeš staviti `require(!claimed[msg.sender])` i tako spriječiti **bilo kakav** drugi mint (bez obzira tko plaća gas).  
   - To je jednostavnije i robusnije jer vrijedi i za direktne (nesponzorirane) pozive.

Za tvoj use‑case je vrlo razumno napraviti **kombinaciju**: SBT ugovor limitira mint na 1 po adresi, a paymaster eventualno provjerava još neki off‑chain kriterij (npr. “ima verified e‑mail”).

### Koliko xDAI treba u paymasteru?

Ako grubo procijenimo:

- **UserOp ukupni gas**:  
  - Tvoj `claimCertiliaSBT()` ~3M gas (kriptografska verifikacija).[user question]  
  - ERC‑4337 overhead (preVerification + validation + EntryPoint bookkeeping) za jednostavne račune je tipično ~40k gas u “best case” mjeranjima za bundle od 10 UserOp‑ova, ali s Safeom, paymasterom i nešto punijim calldata može narasti na 100k–200k.[4][36][52]
  - Konservativno: **3,2M–3,5M gas** po UserOp‑u.

- **Gas cijena na Gnosisu**:  
  - Gnosis koristi xDAI (stabilan oko 1 USD) kao native token, s cijenama plina reda par gwei i tipičnim transakcijama < 0,01 xDAI.[5][6][8][53]
  - Uzmimo 2 gwei kao razumnu radnu pretpostavku.

Trošak u xDAI:  
$$\text{cijena} \approx \text{gas} \times \text{gasPrice} = 3{,}3\text{M} \times 2 \times 10^{-9} \approx 0{,}0066 \text{ xDAI}$$ po UserOp‑u (što je oko 0,0066 USD).[6][4][5]

Otprilike:

- 1.000 korisnika: ~6,6 xDAI.  
- 10.000 korisnika: ~66 xDAI.  

Ako želiš dodatni buffer (fluktuacije gas cijene, neuspjele UserOp‑ove), stavi, recimo, **2× tu vrijednost**, tj. reda 15 xDAI za 1.000 korisnika ili 150 xDAI za 10.000 korisnika.

***

## 4. Smart Account tvornice na Gnosisu

### Safe{Core} Protocol (Safe AA)

- **Safe smart accounti** su dostupni na Gnosisu (Chain ID 100) i službeno podržani u Safe dokumentaciji.[54]
- Gnosis AA vodiči pokazuju kako stvoriti Safe smart account s `ENTRYPOINT_ADDRESS_V06` i koristiti ga s Pimlico bundlerom/paymasterom na Gnosisu (permissionless.js primjeri).[55][12]
- Uz Safe{Core} Gas Station program možeš čak dobiti gas kredite koji se troše u Pimlico paymasteru.[41]

Za tvoj scenario, Safe kao smart account ti omogućuje:

- spajanje standardnih walleta (MetaMask) i social login rješenja,  
- integraciju s Paymasterima (Pimlico, Biconomy, Particle) bez da sam pišeš account logiku.

### ZeroDev Kernel

- ZeroDev je izgradio modularni smart account framework **Kernel**, i Gnosis dok jasno kaže da u ZeroDev dashboardu možeš birati `network chain = Gnosis` i dobiti `BundlerRPC` i `PaymasterRPC`.[7]
- Kernel podržava ERC‑7579 plugine, što je overkill ako ti treba samo jedno mintanje po korisniku, ali može biti korisno ako planiraš napredniju logiku u budućnosti.[24][7]

### SimpleAccount (referentna implementacija)

- `SimpleAccount.sol` i `SimpleAccountFactory.sol` postoje u eth‑infinitism repozitoriju kao referentni accounti za v0.6/v0.7 EntryPoint, ali **ne postoji službeno “globalno” SimpleAccountFactory deployment za Gnosis** u dokumentaciji – lako ga sam deployaš ako želiš minimalni account umjesto Safe/Kernel‑a.[56][57][45]
- Particle Network već koristi SimpleAccount kao jednu od opcija smart accounta na Gnosisu, tako da možeš i to dobiti preko njihovog AA SDK‑a bez vlastitog deploya.[page:particle-network][58]

### Trebam li smart account ili EOA?

- **ERC‑4337 zahtijeva smart account** (contract account) koji implementira `validateUserOp`, tako da čist EOA **ne može** direktno biti `sender` u UserOp‑u prema trenutnoj specifikaciji.[59][60][55]
- Ako želiš zadržati korisnike kao obične EOA‑e (MetaMask novčanik) i samo im omogućiti 1 gasless funkciju, **meta‑transakcije (EIP‑2771 / OpenGSN ili Gelato Relay)** su znatno jednostavnije.

Za tvoj slučaj (jedan težak poziv `claimCertiliaSBT()` po korisniku, bez zahtjeva za “trajnim” smart walletom), **čist EOA + meta‑transakcije** (OpenGSN ili vlastiti relayer) često je povoljniji kompromis od uvođenja punog 4337 stacka.

***

## 5. Alternativa ERC‑4337: meta‑transakcije

Ako zaključiš da je 4337 pretežak za tvoj use‑case, imaš sljedeće opcije:

### EIP‑2771 & OpenGSN

- OpenGSN je implementacija EIP‑2771 (trusted forwarder + relayer + paymaster), a Gnosis je službeno podržana mreža s definiranim **RelayHub**, **Forwarder** i primjerom Paymastera.[3][42]
- Arhitektura:  
  - Korisnik potpisuje meta‑transakciju (poruku) umjesto da plaća gas.  
  - Tvoj (ili javni) relayer šalje stvarnu transakciju i plaća gas.  
  - `Forwarder` ugovor verificira potpis i prosljeđuje poziv tvojem `Recipient` ugovoru; u ugovoru i dalje vidiš originalnog korisnika (`_msgSender()`).[43]

Za tebe to znači:

- Tvoj `CertiliaSBT` ugovor postaje GSN recipient.  
- Korisnik potpisuje poruku “pozovi `claimCertiliaSBT()`”;  
- Tvoj relayer (ili GSN relayer) šalje stvarnu transakciju i plaća gas u xDAI.  
- Pravilo “1 mint po adresi” rješavaš jednostavno u samom SBT ugovoru.

### Gelato Relay (gasless, ne‑4337)

- Safe + Gelato već su demonstrirali gasless kampanju na Gnosisu, gdje je Gnosis Chain sponzorirao transakcije preko Gelato 1Balance sustava.[page:free-transactions-on-gnosis-chain-with-safe-wallet]  
- U Gnosis vodiču, Gelato Relay Kit omogućuje da sponzoriraš transakcije na Gnosisu (1Balance, SyncFee); to su **meta‑transakcije** (relayer uzima tvoju signature i šalje stvarnu tx).[2][32][61]
- Prednost je što ne moraš misliti o EntryPointu niti UserOp strukturi; mana je veća vendor‑lock‑in na Gelato API.

### Vlastiti relayer (najjednostavnije)

Za tvoj konkretan scenarij (jedna target funkcija, ograničenje 1× po adresi):

- Napišeš vrlo jednostavan backend (Node/Go/Rust) s dva endpointa:  
  1. Klijent šalje potpisanu poruku `claimCertiliaSBT()` (EIP‑712 / personal_sign, po želji).  
  2. Backend provjerava potpis, provjerava u vlastitoj bazi ili na chainu je li `claim` već pozvan za tu adresu i, ako sve prolazi, šalje običnu `claimCertiliaSBT()` transakciju s vlastitog EOA‑a koji ima xDAI.  
- Ovo **nije** 4337 niti 2771; to je “custom meta‑tx”, ali za jedan specifičan ugovor je daleko najjednostavnije.  

Kompleksnost:

- ERC‑4337: trebaš EntryPoint, smart account (Safe/Kernel/SimpleAccount), bundler, paymaster i često poseban RPC stack.[55][59]
- EIP‑2771 / OpenGSN: trebaš implementirati recipient pattern i pokrenuti GSN relayer + paymaster, ali nemaš UserOp/Bundler sloj.[3][43]
- Vlastiti relayer: najmanje protokolskog “overheada”, najviše custom koda – ali za jednu funkciju to je realno par stotina linija server‑side koda.

Za tvoju aplikaciju (SBT mint, 1× po korisniku, gas heavy) bih ozbiljno razmotrio:

- **Opcija A (meta‑tx)**: CertiliaSBT + vlastiti relayer ili OpenGSN; nema AA sloja, korisnik ostaje EOA, a ugovor brine o ograničenju 1×.  
- **Opcija B (4337)**: Safe smart account + Pimlico ili Biconomy paymaster na Gnosisu, posebno ako ti je AA ionako u planu za druge feature‑e.

***

## 6. Troškovi: UserOp overhead i cijene na Gnosisu

### Koliki je overhead UserOperationa?

Analize implementacije eth‑infinitism EntryPointa pokazuju da je za “idealni” slučaj (jednostavan račun, bundle od 10 UserOp‑ova) **overhead po UserOp‑u ~39,8k gasa**, otprilike 2× overhead obične EOA transakcije.[4]

Realno, uz:

- zreliji smart account (Safe/Kernel),  
- paymaster validaciju,  
- veći `callData` (kriptografske dokaze),  

možeš očekivati ukupni overhead (preVerification + verification + EntryPoint bookkeeping) u rasponu **50k–150k gasa** po UserOp‑u. To je i dalje mali dodatak na tvojih 2–5M gasa.[36][52][62]

### Ako `claimCertiliaSBT()` troši oko 3M gasa

Konzervativna procjena:

- `callGasLimit` ≈ 3.000.000  
- Overhead (preVerification + validation + paymaster): recimo 200.000  
- Ukupno: **3.200.000 gas** po UserOp‑u.

Na Gnosisu:

- xDAI ≈ 1 USD.[6]
- Tipične gas cijene: 1–3 gwei, median oko 2 gwei.[8][53][5]

Trošak:

- Pri 2 gwei:  
  - 3.200.000 × 2 × 10⁻⁹ ≈ **0,0064 xDAI** po operaciji (~0,0064 USD).  
- Pri 3 gwei (konzervativnije):  
  - 3.200.000 × 3 × 10⁻⁹ ≈ **0,0096 xDAI** (~0,01 USD).

Za 3M gas call plus 200k overhead, razlika u odnosu na “plain” transakciju je praktički zanemariva na jeftinom chainu poput Gnosisa; overhead 4337 ti diže cijenu možda za 10–20% u gasu, što je i dalje ispod centa.

### Trošak za 1.000 i 10.000 korisnika

Uz 3,2M gas i 2 gwei:

- **1.000 korisnika**  
  - 1.000 × 0,0064 xDAI ≈ **6,4 xDAI** (~6,4 USD).  
- **10.000 korisnika**  
  - 10.000 × 0,0064 xDAI ≈ **64 xDAI** (~64 USD).

Ako uzmeš gornji raspon (5M gas po korisniku, 3 gwei gas price, dodatni EntryPoint overhead):

- 5.200.000 × 3 × 10⁻⁹ ≈ 0,0156 xDAI po korisniku.  
- 1.000 korisnika ≈ 15,6 xDAI, 10.000 korisnika ≈ 156 xDAI.

U oba slučaja radi se o **desecima xDAI‑a za 10k korisnika**, što je prilično malo u odnosu na UX benefit koji dobiješ.

***

## Praktična preporuka za tvoj use‑case

Za “prvo mintanje SBT‑a bez xDAI‑a, 1× po korisniku, gas heavy (2–5M)” na Gnosisu:

- Ako želiš **što jednostavniju arhitekturu** i korisnici mogu ostati EOA:  
  - Implementiraj **EIP‑2771 kompatibilni ugovor** (OpenGSN recipient) ili čak custom relayer.  
  - U samom `claimCertiliaSBT()` stavi `require(!claimed[msg.sender])` i limitirat ćeš se na 1 mint po adresi.  
  - Tvoj relayer/Paymaster drži, recimo, 100–200 xDAI i pokriva 10k–20k korisnika.[43][3][6]

- Ako želiš uvesti **AA i smart account‑e (Safe/Kernel)**:  
  - Razmisli o **Safe smart accountima + Pimlico Verifying Paymaster** na Gnosisu (ili Biconomy Sponsorship/Token Paymaster ako ti više paše njihov dashboard).[26][12][17]
  - Pravilo “1 sponzorirani mint po adresi” rješavaš kombinacijom mappinga u Paymasteru i `require` u SBT ugovoru.  
  - Možeš se i prijaviti na **Safe{Core} Gas Station** za gas kredite na Gnosisu, čime dodatno smanjuješ vlastiti trošak.[41]

Ako želiš, mogu ti skicirati konkretne ABI strukture i primjer koda za:

- minimalni VerifyingPaymaster na Gnosisu s mappingom `hasSponsored[address]`, ili  
- minimalni EIP‑2771 recipient za OpenGSN i jednostavni relayer.

Sources
[1] Understanding Account Abstraction - Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/
[2] Building on Safe AA with Gelato - Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/Safe%20and%20supported%20AA%20infra%20providers/gelato
[3] Network Gnosis Chain (formerly xDAI) | v3.0.0-beta.3 pre-release https://docs.opengsn.org/networks/gnosis/gnosis.html
[4] Unpacking ERC-4337 - Frontier Research https://frontier.tech/unpacking-erc-4337
[5] Gas Fees Calculator - Cryptoneur: Empowering Crypto Entrepreneurs https://cryptoneur.xyz/en/gas-fees-calculator
[6] What Is Gnosis Chain? https://www.gnosis.io/blog/what-is-gnosis-chain
[7] Building AA powered dApp with ZeroDev - Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/zerodev-guide
[8] Gnosis Chain (xDAI) Blockchain Explorer https://gnosisscan.io
[9] EntryPoint 0.6.0 Released https://erc4337.mirror.xyz/cSdZl9X-Hce71l_FzjVKQ5eN398ial7QmkDExmIIOQk
[10] Entry Point 0.6.0 | Address: 0x5ff137d4...a026d2789 | GnosisScan https://gnosisscan.io/address/0x5ff137d4b0fdcd49dca30c7cf57e578a026d2789
[11] The EntryPoint Contract - ERC-4337 Documentation https://docs.erc4337.io/smart-accounts/entrypoint-explainer.html
[12] Building on Safe AA with Pimilico - Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/Safe%20and%20supported%20AA%20infra%20providers/pimilico
[13] Common Utils | Thirdweb .NET SDK https://portal.thirdweb.com/dotnet/utils
[14] Supported Chains | Pimlico Docs https://docs.pimlico.io/guides/supported-chains
[15] Pricing | Pimlico Docs https://docs.pimlico.io/guides/pricing
[16] Pay-as-you-go pricing simulation https://docs.pimlico.io/infra/platform/pricing
[17] Relay Pricing | Gelato https://docs.gelato.cloud/web3-services/relay/subscriptions-and-payments/relay-pricing
[18] Bundler https://docs.biconomy.io/bundler
[19] @biconomy/bundler https://www.npmjs.com/package/@biconomy/bundler
[20] Execute Multi-Chain Transactions with One Signature https://docs.biconomy.io/faq/multi-chain-execution
[21] Particle Network Docs - Chain Abstraction & Universal Accounts https://particlenetwork-fccf74d2.mintlify.app/aa/sdks/desktop/web
[22] Paymaster - Particle Network docs https://developers.particle.network/guides/aa/paymaster
[23] Paymaster - Particle Network Docs https://particlenetwork-fccf74d2.mintlify.app/aa/guides/paymaster
[24] Introducing Kernel — Minimal & Extensible Smart Contract Account ... https://docs.zerodev.app/blog/kernel-minimal-extensible-account-for-aa-wallets
[25] Supported networks - MetaMask developer documentation https://docs.metamask.io/smart-accounts-kit/0.1.0/get-started/supported-networks/
[26] Paymaster | Biconomy https://docs.biconomy.io/3.0/paymaster
[27] Supported Networks | Biconomy Docs https://account-abstraction-docs.biconomy.io/smartAccountsV2/supportedNetworks/
[28] Biconomy - Universal Execution Layer for Web3 https://biconomy.io
[29] Paymaster Methods | Biconomy https://docs.biconomy.io/Paymaster/methods
[30] Biconomy Paymaster - Web3 Wallet Tools - Alchemy https://www.alchemy.com/dapps/biconomy-paymaster
[31] Supported chains - Biconomy Docs https://docs.biconomy.io/contracts-and-audits/supported-chains
[32] Building Account Abstraction with Safe | Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/Safe%20and%20supported%20AA%20infra%20providers/
[33] Relay Pricing - Gelato https://docs.gelato.cloud/Relay/Subscription-and-payments/Relay-pricing
[34] account_abstraction_alchemy https://pub.dev/documentation/account_abstraction_alchemy/latest/
[35] What are the supported chain for Account kit - Alchemy https://www.alchemy.com/support/what-are-the-supported-chain-for-account-kit
[36] How ERC-4337 Gas Estimation Works - Alchemy https://www.alchemy.com/blog/erc-4337-gas-estimation
[37] List of 8 Account Abstraction (ERC-4337) Bundlers (2025) https://www.alchemy.com/dapps/best/account-abstraction-erc-4337-bundlers
[38] Supported Networks - Stackup https://docs.stackup.fi/docs/supported-networks
[39] GitHub - stackup-wallet/stackup-bundler: A fast, reliable, and ... https://github.com/stackup-wallet/stackup-bundler
[40] GitHub - Particle-Network/gnosis-aa-connect: Demo application showcasing Particle Connect and AA on the Gnosis chain https://github.com/Particle-Network/gnosis-aa-connect
[41] Launching the Safe{Core} Multi-chain Gas Station - Safe Foundation https://safefoundation.org/blog/making-every-app-gasless-launching-the-safe-core-multi-chain-gas-station
[42] GSN Deployment Addresses | v3.0.0-beta.3 pre-release https://docs.opengsn.org/networks.html
[43] Ethereum Gas Station Network (GSN) | v3.0.0-beta.3 pre-release https://docs.opengsn.org
[44] account-abstraction/contracts/samples/VerifyingPaymaster.sol at 6dea6d8752f64914dd95d932f673ba0f9ff8e144 · eth-infinitism/account-abstraction https://github.com/eth-infinitism/account-abstraction/blob/6dea6d8752f64914dd95d932f673ba0f9ff8e144/contracts/samples/VerifyingPaymaster.sol
[45] EIP-4337 – Ethereum Account Abstraction Incremental Audit https://www.openzeppelin.com/news/eip-4337-ethereum-account-abstraction-incremental-audit
[46] Paymasters with Trampoline - Part II - erc4337 https://erc4337.substack.com/p/paymasters-with-trampoline-part-ii
[47] GitHub - imelon2/simple-paymaster-server: Simple paymaster server by ERC-7677 Paymaster Web Service Capability standard https://github.com/imelon2/simple-paymaster-server
[48] JSON-RPC API for verifying paymasters https://hackmd.io/@stackup/H1oIvV-qi
[49] Paymasters | OpenZeppelin Docs https://docs.openzeppelin.com/community-contracts/paymasters
[50] Account Abstraction - OpenZeppelin Docs https://docs.openzeppelin.com/contracts/5.x/account-abstraction
[51] An ERC-4337 Paymaster contract by Pimlico that is able to ... - GitHub https://github.com/pimlicolabs/erc20-paymaster
[52] An analysis of PreVerificationGas - Stackup https://www.stackup.fi/resources/an-analysis-of-preverificationgas
[53] 2.1-1.6 Gwei | XDAI Gas Tracker | GnosisScan https://gnosisscan.io/gastracker
[54] Smart Account Supported Networks - Safe Docs https://docs.safe.global/advanced/smart-account-supported-networks?version=v1.2.0
[55] What is ERC-4337? https://docs.safe.global/advanced/erc-4337/overview
[56] Address: 0x9406Cc61...E44976454 | Etherscan Sepolia https://sepolia.etherscan.io/address/0x9406Cc6185a346906296840746125a0E44976454
[57] ERC4337, the Epicenter of UX Innovation https://xangle.io/en/research/detail/1460
[58] Introducing Social Logins & Modular Account Abstraction on Gnosis https://blog.particle.network/introducing-social-logins-modular-account-abstraction-on-gnosis/
[59] What is account abstraction? | ForkLog https://forklog.com/en/what-is-account-abstraction/
[60] ERC-4337: Account Abstraction via Entry Point Contract ... https://ethereum-magicians.org/t/erc-4337-account-abstraction-via-entry-point-contract-specification/7160
[61] Introducing Gnosis Pay: The Future of Crypto Payments is Here https://gelato.cloud/blog/introducing-gnosis-pay-gelato-relay
[62] ERC-4337: Predicting Gas Consumption for UserOperations https://blog.particle.network/bundler-predicting-gas/
[63] Executing Token Swap on Uniswap V3 with Pimlico's ERC20 ... https://www.buildbear.io/docs/tutorials/pimlico-paymaster
[64] Account Abstraction Wallet: Complete Guide to Smart Contract ... https://www.cobo.com/post/account-abstraction-wallet
[65] Entry Point 0.7.0 | Address: 0x00000000...6f37da032 https://gnosisscan.io/address/0x0000000071727de22e5e9d8baf0edac6f37da032
[66] Entry Point | Address: 0x0576a174...8A0C91B57 | Etherscan https://etherscan.io/address/0x0576a174D229E3cFA37253523E645A78A0C91B57
[67] Paymasters - World Developer Docs https://docs.world.org/world-chain/providers/paymasters
[68] Gnosis Chain Account Abstraction Transactions Information https://gnosisscan.io/txsAA
[69] Supported Tokens - Pimlico Docs https://docs.pimlico.io/guides/how-to/erc20-paymaster/supported-tokens
[70] Build dApps with Account Abstraction Using Banana SDK - YouTube https://www.youtube.com/watch?v=Ff5udllRy9Y
[71] @biconomy/account-contracts - npm https://www.npmjs.com/package/@biconomy/account-contracts
[72] Exploring multi-chain account abstraction solutions: native support ... https://web3.okx.com/learn/exploring-multi-chain-account-abstraction-solutions-native-support-and-erc
[73] How to create and use a Biconomy account with permissionless.js https://docs.pimlico.io/guides/how-to/accounts/use-biconomy-account
[74] Account Abstraction by Stackup https://www.quicknode.com/builders-guide/tools/account-abstraction-by-stackup
[75] What is Account Abstraction? https://www.stackup.fi/resources/what-is-account-abstraction
[76] Gelato Bounties | Account Abstraction Hackathon https://gelato.cloud/blog/account-abstraction-hackathon
[77] Building AA powered dApp with Particle Network | Gnosis Chain https://docs.gnosischain.com/technicalguides/account-abstraction/particle-network
[78] Free Transactions on Gnosis Chain with Safe{Wallet}! https://www.gnosis.io/blog/free-transactions-on-gnosis-chain-with-safe-wallet
[79] Account abstraction - Optimism Documentation https://docs.optimism.io/app-developers/tools/account-abstraction
[80] Alchemy Account Abstraction https://magic.link/posts/alchemy-account-abstraction-guide
[81] Support Gnosis Chain (formerly xDai) · Issue #3643 - GitHub https://github.com/ledgerwatch/erigon/issues/3643
[82] Chainstack announces support for Gnosis Chain https://chainstack.com/chainstack-announces-support-for-gnosis-chain/
[83] account-abstraction-alchemy/README.md at main · wormfare/account-abstraction-alchemy https://github.com/wormfare/account-abstraction-alchemy/blob/main/README.md
[84] AA Infra Providers | Monad Developer Documentation https://docs.monad.xyz/tooling-and-infra/account-abstraction/infra-providers
[85] GitHub - wormfare/account-abstraction-alchemy: Flutter (Dart) ERC-4337: Account Abstraction https://github.com/wormfare/account-abstraction-alchemy
[86] ERC-4337: The Complete Guide to Ethereum Account Abstraction in ... https://www.cobo.com/post/what-is-erc-4337
[87] gnosischain/configs - GitHub https://github.com/gnosischain/configs
[88] Account Abstraction - Plasma Docs - Build on Plasma https://www.plasma.to/docs/plasma-chain/tools/account-abstraction
[89] OP Research: AA Wallet Evolution Illustration - Binance https://www.binance.com/en-IN/square/post/1250141
[90] ZeroDev: Project Guide | Latest Updates, Presale & Airdrop https://web3.bitget.com/en/dapp/zerodev-25958
[91] Supported Networks - ZeroDev docs https://docs.zerodev.app/sdk/faqs/chains
[92] Introduction to Gnosis Chain for Gnosis Pay Users https://help.gnosispay.com/hc/en-us/articles/39672761159828-Introduction-to-Gnosis-Chain-for-Gnosis-Pay-Users
[93] Gas Price Oracle - Gnosis Chain https://docs.gnosischain.com/tools/Oracle%20Providers/gas-price
[94] List of 22 Infrastructure Tools on Gnosis Chain (2025) - Alchemy https://www.alchemy.com/dapps/list-of/infrastructure-tools-on-gnosis-chain
[95] List of 8 RPC Node Providers on Gnosis Chain (2025) - Alchemy https://www.alchemy.com/dapps/list-of/rpc-node-providers-on-gnosis-chain
[96] Subscriptions and Payments https://docs.gelato.network/developer-services/relay/subscriptions-and-payments
[97] alchemy-sdk-js/docs-md/enums/Network.md at master - GitHub https://github.com/alchemyplatform/alchemy-sdk-js/blob/main/docs-md/enums/Network.md
[98] GitHub - 5afe/safe-gelato-relay-service: The Safe Gelato Relay Service is a web service which allows relaying transactions via the Gelato Relay Service. https://github.com/5afe/safe-gelato-relay-service
[99] Contract Trading Fees, Limits, and Rules - Biconomy https://biconomy.zendesk.com/hc/en-us/articles/44424584128153-Contract-Trading-Fees-Limits-and-Rules
[100] Biconomy Review 2026: Is It the Right Exchange for You? - FXEmpire https://www.fxempire.com/exchanges/biconomy
[101] Pimlico Reviews and Pricing 2025 | F6S https://www.f6s.com/software/pimlico
[102] biconomy/paymaster - NPM https://www.npmjs.com/package/@biconomy%2Fpaymaster
[103] Pimlico Paymaster - Web3 Wallet Tools - Lancers Technology https://lancers.technology/web3-explorer/dapp/pimlico-paymaster
[104] Plans & Packages | Pimlico https://www.pimlicosolutions.com/pricing-plans/list
[105] AA demo leveraging Particle WaaS for account management and ... https://gist.github.com/TABASCOatw/61be56f0c2346f757692a1b33248bda9
[106] Stackup Bundler - Web3 Wallet Tools - Alchemy https://www.alchemy.com/dapps/stackup-bundler
[107] arddluma/awesome-list-rpc-nodes-providers: A curated list ... - GitHub https://github.com/arddluma/awesome-list-rpc-nodes-providers
[108] Gnosis Chain tooling - Chainstack Docs https://docs.chainstack.com/docs/gnosis-tooling
[109] Account abstraction - PortalHQ.io https://docs.portalhq.io/resources/account-abstraction
[110] Run a Node | Gnosis Chain https://docs.gnosischain.com/node/
[111] ERC-7677: Paymaster Web Service Capability - Ethereum Magicians https://ethereum-magicians.org/t/erc-7677-paymaster-web-service-capability/19530
[112] Latest Blocks https://gnosis.dex.guru
[113] Permissionless.js Quickstart Guide - Safe Docs https://docs.safe.global/advanced/erc-4337/guides/permissionless-quickstart
[114] Releases · eth-infinitism/account-abstraction - GitHub https://github.com/eth-infinitism/account-abstraction/releases
[115] Gnosis Beacon Chain Explorer: Gnosis https://beacon.gnosisscan.io
[116] Address: 0x0576a174...8A0C91B57 | Etherscan Sepolia https://sepolia.etherscan.io/address/0x0576a174D229E3cFA37253523E645A78A0C91B57
[117] GNOSIS/SCAN https://www.gnosis.io/blog/KY74djhdO6ergyNCawvrYPka5FE_xYyrZ7NzA0N3Kds
[118] signerToEcdsaKernelSmartAcco... https://github.com/pimlicolabs/permissionless.js/issues/219
[119] Entry Point 0.6.0 | Address: 0x5FF137D4...a026d2789 | CeloScan https://celoscan.io/address/0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789
[120] Gnosis Chain Node Tracker | GnosisScan https://gnosisscan.io/nodetracker
[121] EntryPoint | Address 0x0576a174d229e3cfa37253523e645a78a0c91b57 | PolygonScan https://polygonscan.com/address/0x0576a174d229e3cfa37253523e645a78a0c91b57
[122] Migration Guide | Pimlico Docs https://docs.pimlico.io/references/permissionless/v0_1/how-to/migration-guide
[123] secret gist https://gist.github.com/livingrockrises/1b9ff79ee3489ee8b5a28a0df3b023c5
[124] llms.txt - Gnosis Chain https://docs.gnosischain.com/llms.txt
[125] List Supported Chains - Introduction https://tokensniffer.readme.io/reference/get-supported-networks
[126] GitHub - bcnmy/biconomy-paymasters: Paymasters development project https://github.com/bcnmy/biconomy-paymasters
[127] polygon token paymaster https://gist.github.com/livingrockrises/a7cd4a08d25f0f6d6e822f661761eb9a
[128] Complete implementation of Biconomy Quickstart Guide https://gist.github.com/Rahat-ch/ebf7b152fd1a1a8ee6e097adcee50d06
[129] chains/_data/chains/eip155-100.json at master · ethereum-lists/chains https://github.com/ethereum-lists/chains/blob/master/_data/chains/eip155-100.json
[130] How to create and use a Biconomy account with permissionless.js https://docs.pimlico.io/references/permissionless/v0_1/how-to/accounts/use-biconomy-account
[131] Methods | Biconomy Docs https://account-abstraction-docs.biconomy.io/smartAccountsV2/account/methods/
[132] 2023-01-biconomy/README.md at main · code-423n4/2023-01-biconomy https://github.com/code-423n4/2023-01-biconomy/blob/main/README.md
[133] Account.sol - GitHub Gist https://gist.github.com/z0r0z/b670a1808c05d6f0c8be6a585111371d
[134] Developer's Guide to ERC-4337 #5 | Developing a Paymaster https://blog.blockmagnates.com/developers-guide-to-erc-4337-5-developing-a-paymaster-6ce61ef5630f
[135] Developer’s Guide to ERC-4337 #5 | Developing a Paymaster https://blog.blockmagnates.com/developers-guide-to-erc-4337-5-developing-a-paymaster-6ce61ef5630f?gi=e7b198e69ceb
[136] ERC-4337 Account Abstraction Incremental Audit - OpenZeppelin https://www.openzeppelin.com/news/erc-4337-account-abstraction-incremental-audit
[137] GNOSIS/SCAN - Gnosis Chain https://www.gnosischain.com/blog/KY74djhdO6ergyNCawvrYPka5FE_xYyrZ7NzA0N3Kds
[138] Account Abstraction Security Guide - ChainLight Blog & Research https://blog.chainlight.io/patch-thursday-account-abstraction-security-guide-c348cc5e36ee
[139] repo.sourcify.dev https://repo.sourcify.dev/10200/0x1e0e7a2e7573566764f11361E38EFe4d38136a3C
[140] SimpleAccountFactory | Address 0x7192244743491fcb3f8f682d57ab6e9e1f41de6e | OP Mainnet Etherscan https://optimistic.etherscan.io/address/0x7192244743491fcb3f8f682d57ab6e9e1f41de6e
[141] Centre: USDC Token | Address 0xddafbb505ad214d7b80b1f830fccc89b60fb7a83 | GnosisScan https://gnosisscan.io/address/0xddafbb505ad214d7b80b1f830fccc89b60fb7a83
[142] Gnosis Token (GNO) Token Tracker | Arbiscan https://arbiscan.io/token/0xa0b862f60edef4452f25b4160f177db44deb6cf1?a=0x1231deb6f5749ef6ce6943a275a1d3e7486f4eae
