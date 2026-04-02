# Monerium EURe — Claude Opus 4.6 izvještaj

**Prompt:** [07-monerium-eure-gnosis.md](../prompts/07-monerium-eure-gnosis.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# Monerium EURe na Gnosis Chainu — cjelovita tehnička i regulatorna analiza

**EURe (Monerium) je jedini regulirani EUR stablecoin s ozbiljnom likvidnošću na Gnosis Chainu**, sa ~18,3 milijuna tokena u opticaju, potpunom ERC-20 kompatibilnošću uključujući EIP-2612 Permit, te integracijom u Gnosis Pay Visa karticu i Safe{Wallet}. Za projekt podcast tržišta s payment splitter ugovorima, EURe je tehnički potpuno kompatibilan sa `approve()` + `transferFrom()` obrascem, ali nosi regulatorni rizik zamrzavanja adresa putem ugrađenog blacklist mehanizma — što zahtijeva implementaciju **pull payment** uzorka umjesto push distribucije. Monerium trenutno ne naplaćuje naknade za SEPA ↔ EURe konverziju, nudi potpuni API i TypeScript SDK, te sandbox okruženje za razvoj. Ovo istraživanje pokriva sve tehničke specifikacije, regulatornu situaciju pod MiCA uredbom, integracijske mogućnosti i alternative na Gnosis Chainu.

---

## 1. Tehničke specifikacije: ugovor, funkcije i arhitektura

### Adrese pametnih ugovora

| Komponenta | Adresa | Napomena |
|-----------|--------|---------|
| **V2 Proxy (trenutni)** | `0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430` | ERC1967 Proxy — preporučena adresa |
| **V1 Frontend (zastarjeli)** | `0xcB444e90D8198415266c6a2724b7900fb12FC56E` | Balansevi se zrcale s V2 |
| **Implementacija (V2)** | `0x60cb9fdd0fcfd9bb3b2b721864db5e7c07f4635d` | "GnosisControllerToken" na GnosisScan |
| **Deployer** | `0xc5f3370131bb7ce0d28d83735447576aaed1b993` | "Monerium: Deployer" |

EURe je **potpuno kompatibilan ERC-20 token** s **18 decimala** i podrškom za EIP-2612 Permit (gasless odobrenja putem off-chain potpisa). Token podržava i ERC-677 `transferAndCall()` za pozivanje primateljskog ugovora te ERC-1271 validaciju potpisa pametnih ugovora — što je ključno za Safe kompatibilnost.

### Privilegirane funkcije

EURe V2 ugovor sadrži sljedeće ključne privilegirane funkcije:

**`mint(address to, uint256 amount)`** — kovanje novih tokena, dostupno isključivo **System** ulogama. Svaki system račun ima ograničeni mint allowance koji se smanjuje pri svakom kovanju. Aktivni system račun na Gnosis Chainu: `0x882145B1f9764372125861727d7be616c84010Ef`.

**`burn(address from, uint256 amount, bytes32, bytes signature)`** — spaljivanje tokena, zahtijeva **System** ulogu I potpis vlasnika tokena (verificiran putem `SignatureChecker.isValidSignatureNow()`). Monerium ne može spaliti tokene korisnika bez njihovog potpisa.

**`recover(address from, address to, ...)`** — oporavak sredstava s nepristupačnog novčanika na novi, uz potpis koji je korisnik dao pri povezivanju novčanika. Spaljuje s stare adrese i kuje na novu.

**Blacklisting** — implementiran u zasebnom **`BlacklistValidatorUpgradeable.sol`** ugovoru koji je zajednički svim Monerium tokenima (EURe, GBPe, USDe, ISKe) na istom lancu. Admin uloga može dodavati/uklanjati adrese s crne liste. Nema eksplicitne `freeze()` funkcije — zamrzavanje se postiže putem blacklista koji odbija transfere u oba smjera.

### Sustav kontrole pristupa

| Uloga | Tko kontrolira | Ovlasti |
|-------|----------------|---------|
| **Owner** | Gnosis Safe multi-sig | Dodavanje/uklanjanje Admin i System uloga; postavljanje limita kovačnica; postavljanje validatora; autorizacija nadogradnji ugovora |
| **Admin** | Gnosis Safe multi-sig (Monerium administratori) | Povećanje mint allowanca; dodavanje/uklanjanje adresa s crne liste |
| **System** | EOA adrese Monerium infrastrukture | Kovanje tokena; spaljivanje tokena (uz potpis vlasnika) |

### Nadogradivost ugovora

EURe V2 koristi **UUPS (Universal Upgradeable Proxy Standard)** obrazac putem OpenZeppelina. Proxy ugovor je `ERC1967Proxy.sol` koji koristi EIP-1967 storage slotove. **Funkcija `_authorizeUpgrade()` ograničena je na Owner** (Gnosis multi-sig Safe). Jedna zajednička implementacija (`Token.sol`) opslužuje četiri zasebna proxyja za svaku valutu. Ugovori su auditirani od strane **Ackee Blockchain Security**.

### Opskrba i volumen

| Metrika | Vrijednost |
|---------|-----------|
| Ukupna opskrba na Gnosis Chainu | **~18,3 milijuna EURe** |
| Ukupna opskrba na svim lancima | ~28 milijuna EURe |
| Tržišna kapitalizacija | ~31,7 milijuna USD |
| Broj držatelja (V2) | **30.487** |
| Ukupan broj transakcija (V2) | 355.033 |
| 24h volumen trgovanja (DEX, svi lanci) | ~50.000 — 1.750.000 USD (ovisno o izvoru) |
| Najaktivniji DEX par | sDAI/EURE na Oku Trade (~805.000 USD dnevno) |

Kovanje novih tokena odvija se aktivno — na GnosisScan su vidljive mint transakcije svakih nekoliko minuta, što ukazuje na kontinuirani SEPA on-ramp promet.

---

## 2. Regulatorna situacija: zamrzavanje, MiCA i KYC

### Monerium kao regulirani izdavač

Monerium EMI ehf. drži **licencu elektroničke novčane institucije (EMI)** — prvu takvu licencu na svijetu za blockchain izdavanje — izdanu u lipnju 2019. od strane islandskog regulatora (danas **Središnja banka Islanda**). Licenca je izdana temeljem islandskog Zakona br. 17/2013 (izdavanje elektroničkog novca) i Zakona br. 18/2018 (AML mjere), u skladu s EU Direktivom 2009/110/EC. Licenca je **passportirana na svih 28 EU država članica plus Island, Norvešku i Lihtenštajn** (čitav EEA prostor, ~520 milijuna ljudi).

### Mehanizam zamrzavanja

**Monerium može zamrznuti EURe na bilo kojoj adresi** putem `BlacklistValidatorUpgradeable` pametnog ugovora. Kada je adresa na crnoj listi:

- **Ne može slati EURe** — svaki `transfer()` ili `transferFrom()` s te adrese se odbija
- **Ne može primati EURe** — svaki transfer na tu adresu se također odbija
- **`approve()` + `transferFrom()` obrazac je također blokiran** — validator provjerava i `from` i `to` adresu
- **Sredstva su efektivno zamrznuta na mjestu** — EURe saldo ostaje na adresi, ali je nepomičan

Prema službenoj izjavi Moneriuma (veljača 2022.), zadržavaju pravo zamrzavanja u slučaju: **(1)** potencijalne sigurnosne prijetnje za Monerium ili njegove korisnike, te **(2)** usklađenosti sa zakonom, propisom ili sudskim nalogom od islandskih vlasti, islandskih sudova ili drugih državnih tijela s jurisdikcijom nad Moneriumom.

### Poznati slučajevi zamrzavanja

Jedini javno dokumentirani slučaj je **zamrzavanje 611.000 EURe** nakon hakiranja LCX AG burze u siječnju 2022. na Ethereumu. Adresa `0x165402279f2c081c54b00f0e08812f3fd4560a05` je stavljena na crnu listu, a sredstva su naknadno prekovana na novu LCX adresu. **Na Gnosis Chainu nisu pronađeni javno dokumentirani slučajevi zamrzavanja**, ali točan broj se može utvrditi upitom `BlacklistAdded` eventova na blacklist validator ugovoru.

### MiCA uredba (2023/1114) i zamrzavanje

Članci 44–47, koje korisnik navodi, zapravo pripadaju **Glavi III (tokeni s referencom na imovinu / ART)**, a ne Glavi IV (tokeni elektroničkog novca / EMT). Relevantni članci za EURe kao EMT nalaze se u **Glavi IV, članci 48–58**:

- **Članak 48** — Zahtjevi za ponudu/uvrštenje: Samo ovlaštene kreditne institucije ili EMI mogu izdavati EMT-ove. EMT-ovi se "smatraju elektroničkim novcem"
- **Članak 49** — Izdavanje i otkupljivost: Otkup po nominalnoj vrijednosti u bilo kojem trenutku
- **Članak 50** — Zabrana kamata na EMT-ove
- **Članak 51** — Zahtjevi za bijelu knjigu (white paper) za EMT
- **Članak 54** — Ulaganje sredstava: najmanje **30% primljenih sredstava mora biti deponirano** u zasebnim računima kod kreditnih institucija
- **Članak 55** — Planovi oporavka i otkupa
- **Članci 56–58** — Značajni EMT-ovi (pod nadzorom EBA-e)

**Ključni članak za ovlasti zamrzavanja je Članak 94 (Glava VII)** — Ovlasti nadležnih tijela. Stavak 2 eksplicitno navodi ovlast: ***"zahtijevati zamrzavanje ili sekvestraciju imovine, ili oboje"***. To je primarna pravna osnova za zamrzavanje kripto-imovine pod MiCA-om.

Regulatori koji mogu zahtijevati zamrzavanje uključuju: **nacionalna nadležna tijela (NCA)** poput Središnje banke Islanda i NCA-ova drugih EEA država, **Europsko nadzorno tijelo za bankarstvo (EBA)** za "značajne" EMT-ove, **Europsku središnju banku (ECB)** s pravom veta, te **nacionalne sudove**. HANFA kao hrvatsko nacionalno nadležno tijelo može zatražiti zamrzavanje putem suradnje s islandskim NCA-om ili izravno ako ima jurisdikciju nad aktivnostima na hrvatskom tržištu.

### Lista zamrznutih adresa

**Ne postoji javno održavana lista zamrznutih adresa** na Moneriumu. Međutim, stanje crne liste je **na lancu i može se upitati**: funkcija `isWhitelisted(address)` na Blacklist ugovoru vraća `false` za blokirane adrese, a `BlacklistAdded` i `BlacklistRemoved` eventi se mogu pratiti za rekonstrukciju pune povijesti. Monerium pruža i Dune Analytics dashboard na `dune.com/monerium/monerium-eure`.

---

## 3. Integracijske mogućnosti: API, SDK, Safe i Gnosis Pay

### Monerium API

Monerium nudi potpuni REST API (verzija **2.0.0**, OpenAPI specifikacija) za programatsku SEPA ↔ EURe konverziju:

| Okruženje | API URL | Web aplikacija |
|-----------|---------|----------------|
| **Produkcija** | `https://api.monerium.app` | `https://monerium.app` |
| **Sandbox** | `https://api.monerium.dev` | `https://sandbox.monerium.dev` |

**SEPA → EURe tok (on-ramp):** Korisnik registrira Monerium račun → prolazi KYC → povezuje web3 novčanik (potpisuje "I hereby declare that I am the address owner") → zahtijeva IBAN povezan s Gnosis Chain adresom → šalje EUR putem SEPA na taj IBAN → Monerium automatski kuje EURe na povezanu adresu. **Monerium pokriva gas naknade za kovanje.** SEPA Instant settlement može trajati samo nekoliko minuta; standardni SEPA **1–2 radna dana**.

**EURe → SEPA tok (off-ramp):** Korisnik potpisuje poruku formata `"Send EUR {iznos} to {IBAN} at {timestamp}"` → šalje nalog putem API-ja (`POST /orders`) → Monerium spaljuje EURe i šalje EUR putem SEPA. Za naloge iznad **15.000 EUR** potrebno je priložiti popratni dokument (račun ili ugovor).

Ključni API endpointi uključuju: upravljanje IBAN-ovima (`POST/GET /ibans`), naloge (`POST/GET /orders`), adrese (`POST/GET /addresses`), bilance (`GET /balances`), webhook pretplate za praćenje statusa naloga u stvarnom vremenu, te autentifikaciju putem OAuth Authorization Code Flow, Client Credentials ili SIWE (Sign In with Ethereum).

### Naknade

**Monerium trenutno ne naplaćuje nikakve naknade.** Ovo uključuje:

- SEPA → EURe konverzija: **0 EUR**
- EURe → SEPA konverzija: **0 EUR**
- Kovanje i spaljivanje tokena: **0 EUR** (gas pokriva Monerium)
- Jedan IBAN po korisniku: **0 EUR**
- Pristup API-ju: **0 EUR**

Monerium zarađuje od ulaganja zaštićenih sredstava korisnika u kratkoročne, visokokvalitetne likvidne instrumente (državne obveznice). Rezerve su **nadkolateralizirane na 102%**. Monerium napominje da mogu uvesti naknade u budućnosti uz prethodnu obavijest.

### KYC proces

Monerium podržava **osobne i poslovne profile**. KYC se provodi putem standardnog europskog eKYC procesa (automatizirano) i tipično zahtijeva osobnu iskaznicu/putovnicu. Za poslovne profile potrebna je registracijska dokumentacija tvrtke i identifikacija stvarnih vlasnika. IBAN računi mogu se otvoriti iz **više od 160 zemalja**. IBAN-ovi su izdani od strane **LHV**, estonske banke, kao virtualni IBAN-ovi (VIBAN). Za partnere s posebnim privilegijama moguća je i **KYC Reliance** opcija gdje se Monerium oslanja na KYC partnera.

### JavaScript/TypeScript SDK

Monerium nudi službeni SDK: **`@monerium/sdk`** (verzija **3.4.4**, ~1.028 tjednih preuzimanja na npm-u). Repozitorij: `github.com/monerium/js-monorepo`. Dostupan je i **`@monerium/sdk-react-provider`** (v1.3.1) s React hookovima (`useAuth()`, `useProfile()`, `useMonerium()`). SDK podržava sva tri načina autentifikacije, upravljanje nalozima, bilancama, IBAN-ovima i webhookovima, te uključuje potpune TypeScript definicije.

### Safe{Wallet} integracija

**Safe može primati i slati EURe na Gnosis Chainu bez posebne konfiguracije** — EURe se ponaša kao standardni ERC-20 token. Monerium je integriran kao **službena Safe App** unutar Safe{Wallet} sučelja te kao dio **Safe{Core} SDK Onramp Kita** (uz Stripe). Korisnici mogu izravno iz Safe sučelja: kreirati Monerium račun, povezati Safe s IBAN-om, primati EUR putem SEPA, te slati EURe na bilo koji IBAN.

Gnosis službeni blog potvrđuje: *"Jedna od jedinstvenih značajki EURe je mogućnost slanja i primanja eura izravno između bankovnih računa i Safe multisig novčanika."*

### Gnosis Pay

**Gnosis Pay je samoskrbnička Visa debitna kartica** koja koristi **EURe kao primarnu valutu za EUR transakcije**. Arhitektura:

Svaki korisnik dobiva **Gnosis Pay Safe** — modificirani Safe pametni ugovor na Gnosis Chainu s dva sigurnosna modula: **Delay Module** (3-minutno kašnjenje za sve odlazne transakcije, sprječava dvostruku potrošnju) i **Roles Module** (definira dozvole potrošnje za Gnosis Pay). Kada korisnik koristi karticu na POS-u, Visa mreža komunicira s Gnosis Pay infrastrukturom koja pokreće on-chain EURe transfer iz korisnikovog Safea u izdavačev Safe. Izdavač zatim podmiruje Visu off-chain putem Monerium IBAN-a.

Gnosis Pay trenutno podržava **EURe, GBPe i xDAI** kao valutu za plaćanje, te **USDC.e** kao dodatnu opciju. Kartica košta **30 EUR** i dostupna je u cijelom EEA prostoru.

---

## 4. Rizici za payment splitter projekt

### Rizik zamrzavanja payment splittera

Ovo je **primarni rizik** za podcast tržište. Ako Monerium stavi payment splitter adresu na crnu listu, **svi tokovi sredstava kroz taj ugovor potpuno prestaju** — ugovor ne može ni primati ni slati EURe. Ovo bi pogodilo sve korisnike sustava.

**Preporuke za mitigaciju:**

- **Koristiti pull payment obrazac** umjesto push distribucije. Umjesto da ugovor automatski šalje EURe svim primateljima (gdje jedan blokirani primatelj može blokirati cijelu transakciju), primatelji sami povlače svoj udio. Ako je pojedinačni primatelj blokiran, samo njegov withdraw neće uspjeti, dok ostali nesmetano funkcioniraju
- **Razmisliti o modularnoj arhitekturi** — držati minimalna sredstva u payment splitter ugovoru, a glavninu u zasebnim Safe adresama
- **Implementirati try/catch** oko svih EURe transfer poziva za graceful handling revertova
- **Koristiti OpenZeppelin ReentrancyGuard** kao standardnu zaštitu, posebno ako se koristi ERC-677 `transferAndCall()`

### Rizik nadogradnje ugovora

EURe koristi UUPS proxy — **Monerium može nadograditi implementaciju ugovora**. Iako je malo vjerojatno da bi nadogradnja prekinula ERC-20 kompatibilnost (to bi poremetilo čitav ekosustav), ovo je pretpostavka povjerenja koju treba uzeti u obzir. Nadogradnju autorizira Owner (multi-sig), što dodaje određenu razinu sigurnosti.

### Rizik ovisnosti o Moneriumu

EURe je centralizirani stablecoin — **jedna tvrtka (Monerium EMI ehf.) kontrolira kovanje, spaljivanje i blacklisting**. Ako Monerium prestane s radom ili izgubi licencu, korisnici imaju pravo na otkup po nominalnoj vrijednosti (MiCA članak 49), ali funkcionalni tokeni mogu prestati raditi. Monerium trenutno ne naplaćuje naknade, ali može ih uvesti u budućnosti.

### V1/V2 migracija

Nakon nadogradnje na V2, oba ugovora (V1 i V2) prikazuju iste bilance. Pri korištenju V1, emitiraju se dva Transfer eventa. **Indekseri i monitoring sustavi trebaju koristiti isključivo V2 adresu** (`0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430`) od bloka **35656951** na Gnosis Chainu.

### Gas optimizacija

V2 donosi značajne uštede gasa: `transfer` je **71% jeftiniji** (91.908 → 26.245 gas), a `approve` je **64% jeftiniji** (68.586 → 24.858 gas). Na Gnosis Chainu s njegovim niskim gas cijenama, ovo čini payment splitter operacije praktički besplatnima.

---

## 5. Alternative: usporedba stablecoina na Gnosis Chainu

| Značajka | EURe (Monerium) | EURS (Stasis) | EURA (Angle) | cEUR (Celo) | USDC.e (Circle) | wxDAI |
|----------|-----------------|---------------|--------------|-------------|-----------------|-------|
| **Denominacija** | EUR | EUR | EUR | EUR | USD | USD |
| **Prisutnost na Gnosis** | ✅ Nativno | ⚠️ Zanemariva | ✅ Mala | ❌ Nema | ✅ Bridged | ✅ Nativni gas token |
| **Adresa ugovora** | `0x420CA0...3430` | Via Omnibridge | `0x4b1E2c...5984` | N/A | `0x2a22f9...76f0` | `0xe91D15...a97d` |
| **Likvidnost (Gnosis)** | ~18,3M EUR | ~0 | <100K EUR | N/A | ~13M USD | ~88M USD |
| **Izdavač** | Monerium EMI ehf. | STASIS (Malta) | Angle DAO | Mento/Celo | Circle (SAD) | MakerDAO/Sky |
| **MiCA status** | ✅ Usklađen (EMT) | "MiCA-ready" | ❌ Decentralizirano | ❌ Decentralizirano | Djelomično (EURC ima licencu) | ❌ Decentralizirano |
| **Može zamrznuti?** | ✅ Da (blacklist) | ✅ Da (globalni freeze) | ❌ Ne | ❌ Ne | ✅ Da (blacklist) | ❌ Ne |
| **Decentralizacija** | Centralizirano (regulirani EMI) | Centralizirano | Decentralizirano (DAO) | Polu-decentralizirano | Centralizirano | Decentralizirano |
| **Kolateral** | 1:1 EUR fiat (>100%) | 1:1 EUR fiat | Kripto prekolateriziran | Kripto (~393%) | 1:1 USD fiat | Kripto (MakerDAO) |
| **SEPA on/off-ramp** | ✅ Web3 IBAN | ✅ SEPA/SWIFT | Ograničeno | Ograničeno | Putem mjenjačnica | Via xDAI Bridge |
| **Gnosis Pay podrška** | ✅ Da | ❌ Ne | ❌ Ne | ❌ Ne | ✅ Da | ❌ Ne |

**EURe nema stvarnog EUR-denominiranog konkurenta na Gnosis Chainu.** EURS (Stasis) ima veću ukupnu tržišnu kapitalizaciju (~283M USD), ali **praktički nultu likvidnost na Gnosis Chainu**. EURA (Angle) je prisutna ali s minimalnom likvidnošću (<100K EUR) i postoji u Balancer stEUR/EURe poolu. cEUR uopće nije dostupan na Gnosis Chainu.

Kao USD-denominirani fallback, **wxDAI** dominira s ~73,5% udjela svih stablecoina na Gnosis Chainu (~88M USD), dok **USDC.e** raste (~13M USD) uz aktivne programe poticaja (GIP-103) i podršku u Gnosis Pay kartici te Aave V3.

DeFi protokoli s aktivnom EURe likvidnošću na Gnosis Chainu uključuju: **Oku Trade** (sDAI/EURE par, ~805K USD dnevno), **Curve**, **Balancer V2** (stEUR/EURe pool), **Aave V3** i **CoW Swap**.

---

## Zaključak: ključni uvidi za podcast tržište

**EURe je jedini praktični izbor za EUR-denominirani payment splitter na Gnosis Chainu.** Nijedan drugi EUR stablecoin nema dovoljnu likvidnost niti infrastrukturnu podršku. Ovo je istovremeno i prednost (duboka integracija s ekosustavom) i rizik (potpuna ovisnost o jednom izdavaču).

Tri ključna arhitekturna zaključka za projekt:

**Pull payment obrazac je obavezan, ne opcionalan.** Budući da blacklisting bilo koje adrese u lancu distribucije uzrokuje revert cijele transakcije u push modelu, payment splitter mora implementirati pull mehanizam gdje svaki primatelj samostalno povlači svoj udio. Ovo izolira rizik zamrzavanja pojedinačnih adresa.

**Nulte naknade su privremena pogodnost.** Monerium trenutno ne naplaćuje ništa za SEPA ↔ EURe konverziju i čak pokriva gas naknade za kovanje. Ovo čini ekonomiku projekta vrlo povoljnom, ali treba planirati za scenarij uvođenja naknada. Monerium zarađuje od investicijskog prihoda zaštićenih sredstava, što je održiv model, ali ne garantira trajno besplatnu uslugu.

**MiCA ne sadrži članak specifično naslovljen "zamrzavanje e-money tokena"**, ali **Članak 94(2)** eksplicitno daje nacionalnim nadležnim tijelima ovlast zahtijevati "zamrzavanje ili sekvestraciju imovine". Za hrvatski kontekst, HANFA kao NCA može zatražiti zamrzavanje putem suradnje s islandskim regulatorom ili izravno temeljem MiCA-e. Broj stvarnih zamrzavanja na Gnosis Chainu je nepoznat ali vjerojatno vrlo mali — jedini poznati slučaj (LCX, 611K EURe) dogodio se na Ethereumu i bio je vezan uz hakiranje, ne regulatornu akciju. Rizik zamrzavanja payment splitter adrese je nizak ali nije nula, te ga arhitektura sustava mora uzeti u obzir.