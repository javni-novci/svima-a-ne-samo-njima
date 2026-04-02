# 0xSplits i razdjelnici — Perplexity izvještaj

**Prompt:** [04-0xsplits-gnosis.md](../prompts/04-0xsplits-gnosis.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

# Razdjelnici plaćanja za EURe na Gnosis lancu

## Pregled i glavne preporuke

Ovo izvješće analizira 0xSplits (v1 i v2), alternativne protokole (Superfluid, Sablier, Drips), OpenZeppelin PaymentSplitter i mogućnost vlastite implementacije za scenarij: automatska raspodjela EURe (Monerium e‑money) na Gnosis lancu 90% kreatoru, 10% platformskom Safe trezoru.[^1][^2]
Zaključak je da je, uz EURe kao standardni ERC‑20, tehnički najzrelija opcija 0xSplits (v1/v2) ili jednostavni, revidirani PaymentSplitter/OZ‑bazirani razdjelnik, dok se streaming protokoli više isplate tek za pretplate ili kontinuirana plaćanja.[^3][^2]

## 0xSplits na Gnosis lancu

### Dostupnost i verzije

Splits tim navodi da uz Ethereum podržavaju još sedam glavnih mreža, uključujući Gnosis Chain, što jasno implicira da je protokol (barem v1) deployan i na chainId 100.[^4]
Njihov Warehouse ugovor – temelj v2 arhitekture – ima službenu listu mreža u dokumentaciji gdje je Gnosis (100) eksplicitno naveden među podržanim chainovima, što znači da je infrastruktura za v2 prisutna i na Gnosisu.[^5]

Subgraph repozitorij za Splits navodi Gnosis kao jednu od mreža za koju postoji Splits subgraph, što dodatno potvrđuje da Splits ugovori emitiraju događaje na Gnosisu i da ekosustav alata očekuje tu mrežu.[^6]
Na samom Gnosisscanu postoje transakcije označene kao "Create: SplitMain", što pokazuje da je v1 SplitMain ugovor stvarno deployan na Gnosis Chainu, iako dokumentacija ne objavljuje centralizirani popis svih adresa.[^7]

Trenutna javna dokumentacija ne sadrži zasebnu tablicu s točnim v2 adresama za Gnosis (Warehouse + SplitFactoryV2), ali v2 se deterministički deploya pomoću CreateX tvornice na isti skup adresa preko mreža, uz zahtjev da ciljani chain podržava odgovarajući EVM/OPCode skup – što Gnosis čini.[^8]
Praktično, integracija se u aplikacijama radi preko Splits SDK‑a (SplitV2Client/SplitsClient) gdje se zadani chainId postavlja na 100; SDK zatim koristi odgovarajuće adrese ugovora za tu mrežu.
[^9][^10]

### Podrška za ERC‑20 tokene i EURe

Službena dokumentacija za Split (v1) navodi da svaki Split može primati i distribuirati nativni ETH te ERC‑20 tokene; svi primljeni ERC‑20 tokeni akumuliraju se i raspodjeljuju po postocima.[^11]
Za SplitV2 se izričito kaže da svaki Split djeluje kao platni smart‑wallet koji prima ETH i bilo koji ERC‑20 token, pri čemu se sredstva čuvaju u balansu Splita (Push) ili u Warehouse ugovoru (Pull).[^12]

Ograničenja su prvenstveno za netransferabilne, fee‑on‑transfer i rebasing tokene, no Monerium EURe je standardni ERC‑20 bez takvih specifičnosti, pa se ponaša identično drugim ERC‑20 tokenima u Splits ekosustavu.[^13]
Stoga se EURe na Gnosisu može distribuirati preko Splitsa kao bilo koji drugi ERC‑20 – potreban je samo standardni approve/transferFrom tok za contract‑based tokove isplata.[^13][^11]

### Model distribucije: Pull i Push

V1 Splits koristi čist pull model: sredstva (ETH/ERC‑20) se šalju na Split proxy, zatim ih keeper/distributor pozivom `distributeETH`/`distributeERC20` premješta u SplitMain i zapisuje koliko svaki primatelj ima pravo povući; primatelji zatim sami zovu `withdraw` kako bi podigli sredstva.[^14][^11]
V2 uvodi dva tipa Splita izgrađena na Warehouseu: PullSplit, gdje se nakon distribucije sva sredstva nalaze u Warehouseu i primatelji ih povlače sami (pull flow), te PushSplit, gdje se sredstva pri distribuciji pokušavaju odmah gurnuti na adrese primatelja, uz fallback u Warehouse ako transfer ne uspije.[^12]

Za vaš slučaj „90% kreatoru, 10% Safe trezoru“ tipičan obrazac je:
- postaviti PullSplit s dvama primateljima (kreator i Safe) i odgovarajućim postotcima,
- integrirati se na razini platforme tako da sav EURe prihod ide na adresu Splita,
- periodički (ili po događaju) pozivati `distribute` (koje može pokretati keeper uz distributorFee), a kreator i Safe sami periodično povlače svoja sredstva iz Warehousea.[^1][^12]

### Potrošnja goriva (gas)

Za v1 SplitMain na Ethereum mainnetu nedavna CreateSplit transakcija potrošila je oko 1.195.135 gas jedinica (vjerojatno manji broj primatelja), dok je stariji CreateSplit s većim brojem primatelja dosegnuo oko 12.400.552 gas jedinica, što pokazuje da je trošak približno linearan u broju primatelja.[^15]
Audit izvješće za Splits v2 pokazuje da distribucija Splita s 1.100 primatelja može potrošiti preko 30 milijuna gas jedinica, te preporučuje da se Splits u praksi ograniče na ~250 primatelja kako bi se izbjeglo riskiranje dosezanja blok gas limita.[^16]

Za male konfiguracije poput vaše (dva primatelja) kreiranje i distribucija Splita su redovi veličine milijun gas jedinica ili manje, uz standardnu linearnu ovisnost o broju primatelja; stvarni trošak ovisit će o točnom code pathu (v1 vs v2, Push vs Pull) i broju distribuiranih tokena.[^16][^12]
Specifične brojke za Gnosis nisu zasebno dokumentirane, ali kao EVM‑kompatibilni chain koristi iste gas jedinične troškove za operacije, pa su gas jedinice praktički iste kao na Ethereumu, dok je sama cijena po gasu značajno niža.[^17]

### Sigurnost i revizije

Mirror pregled Splitsa navodi da su originalni (v1) Splits smart‑ugovori deployani na Ethereum mainnet i neovisno revidirani od strane Shipyard tima, što je bitna referenca za sigurnost prve verzije protokola.[^18]
Za Splits v2 postoji detaljno audit izvješće neovisnog istraživača Zacha Obronta (Lead researčer u Sherlocku i Spearbitu), koji je pregledao cijeli splits‑contracts‑monorepo, uključujući Warehouse, SplitWalletV2 i SplitFactoryV2, te identificirao nekoliko visokih i srednjih ranjivosti koje su ispravljene prije finalnog commita.[^16]

Audit naglašava specifične rizike (npr. prevelik broj primatelja može dovesti do zaključavanja sredstava zbog gas limita, rounding edge caseovi kod tokena s niskom decimalnom preciznošću, potencijalni problemi s incentivima u Warehouseu) te dokumentira mitigacije i ograničenja koja bi integratori trebali poštivati.[^16]
Sveukupno, i v1 i v2 imaju profesionalne revizije i javno objavljene izvještaje, što ih čini sigurnijim izborom od ad‑hoc vlastitih implementacija.

### SDK za JavaScript/TypeScript

Splits dokumentacija opisuje `@0xsplits/splits-sdk`, s klasama kao što su `SplitsClient`, `SplitV2Client`, `LiquidSplitClient` i Data Client, koje se inicijaliziraju s `chainId`, `publicClient` i opcionalnim `walletClient`‑om (viem), te API ključem za njihovu GraphQL API uslugu.[^10][^9]
SDK pruža write funkcije (`createSplit`, `updateSplit`, `createLiquidSplit`, razni template helperi) i read/data funkcije (`getSplitMetadata`, `getAccountMetadata`, itd.), što znatno pojednostavljuje integraciju u TypeScript/JavaScript backend ili frontend (npr. Next.js).[^19][^9]

### Sažetak za vaš use‑case (0xSplits)

- Gnosis podrška: Da; Splits službeno navodi Gnosis kao podržani mainnet, a Warehouse je deployan na chainId 100.[^5][^4]
- ERC‑20/EURe podrška: Da, sve dok je token standardni ERC‑20 (što EURe jest).[^13][^12]
- Model distribucije: Pull (v1 i PullSplit v2) i opcionalno Push (PushSplit v2) uz fallback u Warehouse.[^11][^12]
- Revizije: v1 revidirao Shipyard; v2 detaljno revidirao Zach Obront, s javnim izvješćem.[^18][^16]
- Integracija: Zreli TS/JS SDK, gotovi helperi, subgraph/data API.

## Alternativni razdjelnici na Gnosisu

### Superfluid (streaming plaćanja)

Superfluid je protokol za real‑time streaming tokena; službena objava navodi da je deployan na više mreža uključujući Polygon, Gnosis Chain, Arbitrum, Optimism, Avalanche i BSC.[^20][^21]
Superfluid streamovi su programabilni i kompozabilni, što omogućuje kontinuirane pretplate, plaće i druge tokove koji se mogu kombinirati sa smart‑ugovorima (npr. za automatsko redistribuiranje).
[^21][^20]

Za vaš slučaj jednokratne raspodjele po transakciji, klasični Splits/PaymentSplitter je jednostavniji, dok je Superfluid relevantniji ako želite:
- pretplate (mjesečne ili kontinuirane streamove prema kreatoru i platformi),
- payroll ili kontinuirani grantovi koji se u svakom trenutku mogu „pauzirati/otkazati“.
[^20][^21]
Superfluid podržava bilo koji ERC‑20 koji je „wrappan“ u njihov Super Token standard; EURe bi u tom slučaju trebalo wrapati u odgovarajući Super Token prije streaminga.
[^20]

### Sablier (streaming i vesting)

Sablier je DeFi protokol za real‑time tokove tokena (streaming), vesting i airdropove; službena dokumentacija navodi da je protokol deployan na 24+ mainneta, uključujući Gnosis (chainId 100, nativni token xDAI).[^22]
Streamovi u Sablieru rade nad ERC‑20 tokenima (ne samo nad nativnim tokenom), te su pogodniji za linearnu isplatu tijekom vremena nego za instantni split pojedinačne uplate.[^23][^22]

Za use‑case pretplate (npr. godišnje članstvo naplaćeno unaprijed, ali isplaćeno kreatoru i platformi linearno kroz vrijeme) Sablier može biti dobar izbor, no za jednostavan 90/10 split po transakciji to je prekompleksno rješenje.
Integracija zahtijeva pozive njihovih streaming ugovora, ali postoji dobro dokumentiran SDK i detaljan popis chainova na kojima su deployani, uključujući Gnosis.[^22]

### Drips

Drips je Ethereum‑bazirani protokol za streaming i splitting ERC‑20 tokena, s naglaskom na retroaktivno financiranje, ovisnosti open‑source projekata i složenije grafove financiranja.[^24][^25]
Službena dokumentacija za smart‑contract detalje objavljuje tablice s adresama za Ethereum mainnet i Sepolia, ali ne navodi Gnosis Chain među podržanim mrežama.[^26]

Nisu pronađeni javni podaci o deployevima Drips ugovora na Gnosis Chainu, pa se za sada ne može pretpostaviti prva klasa podrške za ovaj chain.
Za vaš slučaj, Drips je ionako kompleksniji nego što je potrebno, pa ga se može smatrati manje pogodnim dok ne postoji jasan Gnosis deployment.
[^24][^26]

### OpenZeppelin PaymentSplitter

OpenZeppelin PaymentSplitter je generički, open‑source i široko korišten ugovor za raspodjelu uplaćenih sredstava između grupe primatelja.[^2]
Dokumentacija navodi da podržava i Ether i ERC‑20 tokene, da se udjeli definiraju putem „shares“ vrijednosti po adresi, te da koristi pull model: sredstva ostaju u ugovoru dok primatelj ne pozove `release` (za ETH ili određeni ERC‑20 token).[^27][^2]

PaymentSplitter je dio OpenZeppelin Contracts paketa koji je općenito smatran industrijskim standardom i prolazi stalnu sigurnosnu provjeru i battle‑testing kroz široku upotrebu, iako se ne objavljuje zasebno audit izvješće isključivo za ovaj ugovor.[^28]
Ograničenje je što se popis primatelja i njihovih shares može definirati samo u konstruktoru – nema mogućnosti kasnijeg mijenjanja raspodjele bez novog deploya ugovora.[^2]

Za vaš 90/10 scenarij PaymentSplitter je funkcionalno dovoljan: deploya se jedan ugovor s dva primatelja (kreator i Safe) i odgovarajućim shares (npr. 90 i 10), a svaki EURe transfer na ugovor može se kasnije raspodijeliti tako da kreator i Safe pozivom `release(token, account)` povuku svoje dijelove.[^2]
Gas trošak je linearan u broju primatelja i broju tokena s kojih se povlači, ali sam ugovor je relativno jednostavan, pa je ukupna kompleksnost i overhead manja nego kod Splitsa.
[^29][^2]

### Ostale opcije

Postoje i drugi specijalizirani fee‑splitter ugovori (npr. za specifične protokole poput Curvea ili Obol Networka), no oni su usko vezani uz te ekosustave i nisu generički rješenja za vaše potrebe.[^30][^31]
Većina projekata koji implementiraju on‑chain podjelu prihoda bez dodatnih značajki (waterfall, vesting) koristi ili vlastitu minimalnu logiku sličnu PaymentSplitter‑u, ili direktno 0xSplits.
[^32][^33]

## EURe (Monerium) specifičnosti

### Standard, funkcije i adrese

Moneriumova dokumentacija za token dizajn eksplicitno navodi da Monerium e‑money tokeni (uključujući EURe) u potpunosti slijede ERC‑20 standard, s implementiranim standardnim metodama `totalSupply`, `balanceOf`, `transfer`, `transferFrom`, `approve` i `allowance`.[^13]
Uz standardne ERC‑20 funkcije, tokeni podržavaju multi‑entity minting i burning, adresno specifično zamrzavanje (freezing) i mogućnost nadogradnje ugovora za buduća proširenja.[^13]

Moneriumova Contracts V2 stranica navodi da je na Gnosisu izvršen upgrade EURe ugovora, pri čemu:
- V1 EURe (Monerium EUR emoney) ima adresu `0xcB444e90D8198415266c6a2724b7900fb12FC56E`,
- V2 Monerium EURe ima adresu `0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430`,
- V1 adresu sada „pokriva“ V2 logika iza kulisa, pa obje adrese odražavaju isti balans, ali novi ERC‑20 state živi na V2 ugovoru.[^34]

Token tracker na GnosisScanu za V1 adresu potvrđuje da je u pitanju ERC‑20 token s 18 decimala i opisom „Monerium EUR emoney (EURe)“, uz tisuće holdera i aktivan promet na Gnosis Chainu.[^35]
Moneriumova opća tokens dokumentacija navodi da je Gnosis (chainId 100) jedna od produkcijskih mreža na kojoj su e‑money tokeni živi, sa 18 decimala, što se poklapa s gore navedenim adresama.[^36]

### Mint/burn/freeze i upravljačke uloge

Token dizajn dokument opisuje tri uloge:
- Owner (multi‑sig Gnosis Safe) koji dodaje/uklanja admin i system adrese, postavlja cap za mint allowance i dodaje/miče validatore,
- Admin koji između ostalog povećava mint allowance za `system` adrese i dodaje/uklanja adrese s blacklist‑e,
- System adrese koje Monerium infrastruktura koristi za mint/burn (izdavanje i otkup e‑novca na temelju fiat priljeva/odljeva preko IBAN‑a).[^13]

Dokument potvrđuje da token podržava address‑specific freezing i blacklistanje, pri čemu admin adrese dodaju/miču adrese s blacklist‑e; to omogućuje regulatorno nužno zamrzavanje/ograničavanje računa u skladu s AML/KYC i e‑money regulativom.[^13]
Također opisuje „fund recovery process“ kojim se, uz inicijalni potpis pri povezivanju IBAN‑a, sredstva mogu preseliti s izgubljenog walleta na novi, putem posebne `recover` funkcije koju upravlja Monerium.
[^13]

U praksi to znači da Monerium ima tehničku mogućnost zamrzavanja sredstava na pojedinim adresama (putem blacklist‑e) i premještanja sredstava s izgubljenih walleta, pod uvjetima definiranima njihovom reguliranom poslovnom politikom (npr. sudski nalozi, AML sumnje, rješavanje izgubljenih računa); tehnička dokumentacija ne ulazi u pravne detalje, ali jasno ukazuje na postojanje tih mehanizama.
[^13]

### Kompatibilnost s pametnim ugovorima

Budući da su Monerium tokeni standardni ERC‑20, s jasno specificiranim sučeljem i bez netipičnog ponašanja poput rebasinga ili fee‑on‑transfer mehanizama, oni su dizajnirani tako da budu kompatibilni s tipičnim DeFi i dApp ugovorima koji očekuju standardno ERC‑20 ponašanje.[^13]
Monerium izričito navodi da su tokeni dostupni na Ethereum, Gnosis i Polygon mrežama, što se u praksi odražava kroz već postojeću integraciju sa Safe walletovima i on‑/off‑ramp tokove putem IBAN‑a.[^3][^36]

Moneriumov blog o EURe na Gnosis Chainu opisuje tok SEPA → EURe → Safe wallet: korisnik kreira Monerium račun, poveže Safe na Gnosis Chainu, dobije jedinstveni IBAN, uplati eure i zatim dobije EURe tokene direktno u svoj Safe na Gnosisu; isti put postoji i u obrnutom smjeru (EURe → SEPA bankovni račun).[^3]
Nigdje se ne navode posebna ograničenja za korištenje EURe u pametnim ugovorima izvan standardnih regulatornih aspekata (npr. mogućnost zamrzavanja), pa se EURe može tretirati kao standardni ERC‑20 u smislu integracije s 0xSplits, PaymentSplitterom i vlastitim razdjelnicima.
[^3][^13]

## Vlastita implementacija razdjelnika

### Minimalni funkcionalni zahtjevi

Za najjednostavniji slučaj „90% kreatoru, 10% platformskom Safeu“ minimalan razdjelnik za jedan ERC‑20 token, poput EURe, treba:
- primati EURe (bilo izravnim transferom na ugovor ili putem `transferFrom` nakon `approve`),
- zapisivati udjele primatelja (npr. u postocima ili apsolutnim „shares“ vrijednostima),
- omogućiti ili automatsko slanje (push) na oba primatelja pri svakoj uplati, ili akumulaciju i kasnije povlačenje (pull) kroz funkciju `withdraw`/`release` po korisniku.
[^37][^2]

Na tehničkoj razini to znači implementirati:
- spremanje liste primatelja i njihovih udjela (npr. mapping address → shares i ukupni totalShares),
- računanje raspodjele po uplati prema formuli $$ amount_i = totalReceived * shares_i / totalShares $$ (uz pažnju na zaokruživanje),
- funkcije za povlačenje pojedinog primatelja ili automatsku isplatu prilikom primitka sredstava.
[^37][^2]

### Sigurnosna razmatranja

Analize minimalnih payment splitter ugovora kao i OpenZeppelinov PaymentSplitter ukazuju na nekoliko uobičajenih sigurnosnih točaka:
- koristiti pull umjesto push isplata kako bi se izbjegle reentrancy napade i situacije u kojima primateljski ugovor revertom blokira raspodjelu svim ostalima,
- koristiti provjerene biblioteke poput OpenZeppelin `SafeERC20` za sigurne transferFrom/transfer pozive na ERC‑20 tokenima,
- paziti na overflow/underflow pri računanju raspodjele i na pravilno rukovanje zaokruživanjem (ostatak se može ostaviti u ugovoru ili pridodati određenom primatelju),
- limitirati broj primatelja po razdjelniku kako gas trošak distribucije ne bi premašio blok gas limit, slično preporukama iz Splits v2 audita.
[^2][^16]

Za EURe kao regulirani token dodatno treba računati na mogućnost da Monerium zamrzne adresu razdjelnika ili nekog primatelja; implementacija treba biti robusna na scenarij da određeni primatelj trajno ili privremeno ne može primiti sredstva (npr. kroz pull model gdje sredstva ostaju u ugovoru dok se situacija ne riješi).
[^13]

### Korištenje revidiranih minimalnih implementacija kao temelja

Umjesto pisanja vlastitog razdjelnika od nule, razumno je:
- koristiti OpenZeppelin PaymentSplitter kao bazu, budući da je standardno korišten, dobro pregledan i već implementira sve osnovne sigurnosne pattern‑e (pull model, shares, podrška za ERC‑20),
- ili, ako je potreban napredniji feature‑set (mutable raspodjele, incentivizirani distributori, waterfalli), osloniti se na već revidirane 0xSplits v2 ugovore (PullSplit/PushSplit + Warehouse) umjesto vlastite složene logike.
[^16][^2]

Postoje i javni primjeri minimalnih payment splitter ugovora u raznim člancima i repozitorijima (Conflux Payment Splitter, razni „SplitPayment“ hackathon projekti), ali oni rijetko imaju formalne revizije i uglavnom služe edukativnim svrhama; u produkcijskom kontekstu sigurnije je osloniti se na OpenZeppelin/0xSplits.[^38][^37]

## Usporedna tablica opcija

| Opcija | Gnosis podrška | ERC‑20 podrška | Revidirana | Gorivo (kreiranje) | Gorivo (distribucija) | Složenost integracije |
|--------|----------------|----------------|------------|--------------------|-----------------------|------------------------|
| **0xSplits v1/v2** | Da – Splits navodi Gnosis među podržanim mainnetima; Warehouse ima deployment na chainId 100.[^4][^5] | Da – ETH i standardni ERC‑20; ograničenja samo za netransferabilne, fee‑on‑transfer i rebasing tokene (EURe je standardni ERC‑20).[^12][^13] | Da – v1 auditirao Shipyard; v2 detaljno auditirao Zach Obront.[^18][^16] | Primjer CreateSplit na Eth mainnetu troši ≈1.2M gas za mali broj primatelja; veći Splits mogu doseći >10M gas.[^15] | Linearna u broju primatelja; v2 audit navodi da 1.100 primatelja može trošiti >30M gas, pa se preporučuje ≤~250 primatelja.[^16] | Srednja – potrebna integracija sa Splits SDK‑om (TS/JS) ili izravni pozivi na Split/Factory/Warehouse ugovore; dobijate napredne featurе (mutable splits, distributori, warehouse).[^9][^12] |
| **Superfluid** | Da – dokumenti navode implementacije na Polygonu, Gnosis Chainu, Arbitrumu, Optimismu, Avalancheu i BSC‑u.[^20][^21] | Da – bilo koji ERC‑20 koji je wrapan u Super Token; integracija zahtijeva konverziju EURe → SuperToken.[^20] | Da – protokol je dugo u produkciji, ali pojedina audit izvješća nisu centralizirano navedena u istom obliku kao kod Splitsa; treba provjeriti per‑release.[^20] | N/A – streamovi se kreiraju putem njihovih ugovora; gas trošak je usporediv s drugim konfigurabilnim ugovorima (ovisno o broju streamova i parametara). | Kontinuirani streaming troši gas pri kreiranju/izmjeni/gašenju streama; nema masovnih petlji po primateljima u jednoj transakciji, ali svaki stream je poseban zapis.[^20] | Srednja/visoka – idealno za pretplate i tokove, ne za jednostavne jednokratne splite; zahtijeva integraciju s njihovim Super Token standardom i dashboardom/SDK‑om.[^20][^21] |
| **Sablier** | Da – službeni popis chainova navodi Gnosis (chainId 100, xDAI) kao podržan mainnet.[^22] | Da – radi s ERC‑20 tokenima za streaming, vesting i slične scenarije.[^22][^23] | Da – Sablier je više puta auditiran (ChainSecurity, dr.); dokumentacija navodi opsežnu sigurnosnu praksu, iako pojedina izvješća treba provjeriti po verziji.[^23] | N/A – kreiranje streama je jedna transakcija s gasom proporcionalnim kompleksnosti parametara, ali bez petlji po velikom broju primatelja. | Distribucija je implicitna kroz vrijeme; gas se plaća pri kreiranju, izmjeni ili prekidu streama, ne pri svakom „claimu“ kao kod pull modela; korisnici povlače sredstva prema potrebi.[^23] | Srednja – dobar izbor za pretplate, payroll i vesting; za jednostavni 90/10 split po jednokratnoj uplati nepotrebno kompleksan.[^22][^23] |
| **Drips** | Nejasno – službena tablica smart‑contract adresa pokriva Ethereum mainnet i testnete, bez spomena Gnosis Chaina.[^26] | Da – protokol je dizajniran za streaming i splitting bilo kojeg ERC‑20 tokena na podržanim mrežama.[^24][^25] | Da – postoje javna audit izvješća (npr. Code4rena/Cantina) za Drips ugovore, no fokus je primarno na Ethereum deploymentu.[^25][^39] | N/A na Gnosisu – bez potvrđenog deploya na chainId 100; na Eth‑u kreiranje konfiguracija troši gas proporcionalno složenosti (broj streamova/splitova).[^26] | Distribucija i promjene konfiguracija su gas‑intenzivne ovisno o broju primatelja i složenosti grafova; protokol je zamišljen za složenije funding grafove, ne jednostavne splite.[^25] | Visoka – kompleksan model (DripsHub, drivers, immutable splits, retroPGF); bez jasnog Gnosis deploya nije preporučljivo kao primarno rješenje za vaš slučaj.[^25][^26] |
| **OpenZeppelin PaymentSplitter** | Da – library je blockchain‑agnostičan i radi na bilo kojem EVM chainu uključujući Gnosis; ne zahtijeva posebne deployeve izvan samog ugovora.[^2][^17] | Da – podržava i Ether i ERC‑20 tokene putem istog `release` mehanizma; dokumentacija navodi da očekuje „standardno“ ERC‑20 ponašanje.[^2] | Neformalno da – PaymentSplitter je dio OpenZeppelin Contracts paketa koji je industrijski standard i općenito battle‑tested, iako nema zasebno brandirano audit izvješće samo za ovaj ugovor.[^28] | Deploy jednostavnog PaymentSplittera s nekoliko primatelja je jeftin (jednostavan konstruktor + par struktura podataka); gas uglavnom ovisi o broju primatelja u konstruktoru.[^2] | Distribucija je pull‑model: svaki primatelj poziva `release` za određeni token; gas je linearan u broju primatelja i broju različitih tokena, ali bez dodatnih slojeva poput Warehousea.[^2][^27] | Niska/srednja – API je vrlo jednostavan (constructor + `release` funkcije); nema naprednih značajki (mutable splits, distributori), ali je idealan za fiksne udjele poput 90/10.[^2] |
| **Vlastiti minimalni razdjelnik** | Da – možete ga deployati izravno na Gnosis Chain (EVM kompatibilan).[^17] | Da – pod uvjetom da koristite standardne ERC‑20 operacije (`transfer`, `transferFrom`, `approve`), što je slučaj s EURe.[^13] | Ne, osim ako ga ne pošaljete na formalni audit; bez toga je sigurnosni profil slabiji od 0xSplits/OZ rješenja.[^16][^28] | Ovisi o implementaciji; minimalni splitter s mapiranjem shares i pull isplatama može biti relativno jeftin, ali je lako pogriješiti u dizajnu i sigurnosti.[^37] | Ovisi o izboru push vs pull modela i o broju primatelja; bez pažljivog dizajna može patiti od istih problema koje Splits v2 audit ističe (gas limit, rounding, DoS scenariji).[^16] | Srednja/visoka – zahtijeva dizajn, implementaciju, testiranje i (po mogućnosti) audit; dugoročno ima veći održavateljski teret od korištenja postojećih battle‑tested rješenja.[^37][^16] |

---

## References

1. [Splits v2 is live](https://splits.org/blog/v2-launch/) - Splits v2 has a new flexible architecture, empowering developers to offer custom user experiences ta...

2. [Finance - OpenZeppelin Docs](https://docs.openzeppelin.com/contracts/4.x/api/finance) - Smart contract finance utilities and implementations

3. [EURe on Gnosis Chain](https://www.gnosis.io/blog/eure-on-gnosis-chain) - A new generation of collectively-owned financial products that are more rewarding because you are an...

4. [Happy birthday Splits - Splits.org](https://splits.org/blog/splits-2-year/) - Although Ethereum remains our home, Splits supports seven additional mainnets: Base, Zora, Gnosis, A...

5. [Warehouse](https://docs.splits.org/core/warehouse) - Software to manage onchain earnings

6. [0xSplits/splits-subgraph - GitHub](https://github.com/0xSplits/splits-subgraph) - To access Splits subgraph data post sunset, please use the sdk. Ethereum. Mainnet · Goerli · Sepolia...

7. [Address: 0x9ebc8e61...f856f24e2 | GnosisScan](https://gnosisscan.io/address/0x9ebc8e61f87a301ff25a606d7c06150f856f24e2) - Create: SplitMain. $0.000 XDAI, 0.0125363, 4.16486002. Latest 1 internal transaction. Download Page ...

8. [splits-contracts-monorepo/packages/splits-v2/README.md at main ...](https://github.com/0xSplits/splits-contracts-monorepo/blob/main/packages/splits-v2/README.md) - Splits - v2. Architecture ... Mainnet deployment transactions for gas cost: Warehouse ... This will ...

9. [Splits V2 - Docs](https://docs.splits.org/sdk/splits-v2) - Software to manage onchain earnings

10. [Splits V1 - Docs](https://docs.splits.org/sdk/splits-v1) - import { SplitsClient } from '@0xsplits/splits-sdk' const splitsClient = new SplitsClient({ chainId ...

11. [Split](https://docs.splits.org/core/split) - Software to manage onchain earnings

12. [SplitV2 | Splits - Docs](https://docs.splits.org/core/split-v2) - Each Split gets its own address and proxy for maximum composability with other contracts onchain. .....

13. [smart-contracts/docs/tokendesign.md at main · monerium/smart-contracts](https://github.com/monerium/smart-contracts/blob/main/docs/tokendesign.md) - ERC20 compatible e-money deployed on Ethereum. Contribute to monerium/smart-contracts development by...

14. [0xSplits - protocol breakdown - solidnoob](https://www.solidnoob.com/blog/0xSplits) - Protocol for splitting on-chain income

15. [0xSplits: Split Main | Address: 0x2ed6c4b5...2feb694ee | Etherscan](https://etherscan.io/address/0x2ed6c4b5da6378c7897ac67ba9e43102feb694ee) - ... (address(splitMain), amount); } }. File 4 of 6 : Clones.sol. Outline ... Gnosis (3). $3,940 (<1%...

16. [splits-contracts-monorepo/audits/splits-v2.md at main · 0xSplits/splits-contracts-monorepo](https://github.com/0xSplits/splits-contracts-monorepo/blob/main/audits/splits-v2.md) - Contribute to 0xSplits/splits-contracts-monorepo development by creating an account on GitHub.

17. [Overview | Gnosis Chain](https://docs.gnosischain.com/developers/Overview) - Gnosis Chain is a community-owned EVM-based network that is operated by a diverse set of validators ...

18. [0xSplits](https://review.mirror.xyz/e80ijHgothbSsI-JXj0fTMzVPBKifu7-aDCpfhmjHkU) - 0xSplits is a trustless protocol for splitting on-chain income.

19. [Liquid Splits - Docs](https://docs.splits.org/sdk/liquid) - import { LiquidSplitClient } from '@0xsplits/splits-sdk' const liquidSplitClient = new LiquidSplitCl...

20. [Superfluid is Now Available on Ethereum Mainnet in Early ...](https://superfluid.org/post/superfluid-is-now-available-on-ethereum-mainnet-in-early-access) - Over the past two years of development, Superfluid has been deployed by our community to six…

21. [Superfluid - Gitcoin](https://gitcoin.co/apps/superfluid) - Multichain deployment: Deployed across multiple EVM-compatible networks including Ethereum, Polygon,...

22. [Supported Chains - Sablier Docs](https://docs.sablier.com/concepts/chains) - The Sablier Protocol is deployed on 27 mainnets and 5 testnet EVM chains, although not all of these ...

23. [Sablier Case Study: Multichain Token Distribution | Envio](https://docs.envio.dev/blog/case-study-sablier) - Discover how Sablier streamlined token streaming across 11+ chains using Envio's multichain indexing...

24. [Programmable cashflow - Drips Network](https://www.drips.network/solutions/programmable-cashflow) - Drips enables flexible and extensible streaming and splitting of any ERC-20, to any Ethereum address...

25. [code-423n4/2023-01-drips - GitHub](https://github.com/code-423n4/2023-01-drips) - Drips protocol is 100% on-chain and self-contained, except ERC-20 tokens it works with and drivers. ...

26. [Smart Contract details – Drips Docs](https://docs.drips.network/the-protocol/smart-contract-details/) - Ethereum Mainnet ; Drips cycle seconds, 604800 ; Drips logic, 0xb0C9B6D67608bE300398d0e4FB0cCa3891E1...

27. [ConditionalEscrow](https://docs.openzeppelin.com/contracts/3.x/api/payment) - The official documentation for OpenZeppelin Libraries and Tools

28. [Smart Contract Security Audits - OpenZeppelin](https://www.openzeppelin.com/security-audits) - We conduct a comprehensive review of your system's architecture and codebase, with each line of code...

29. [Michael.W基于Foundry精读Openzeppelin第55期——PaymentSplitter.sol](https://blog.csdn.net/michael_wgy_/article/details/139343434) - 文章浏览阅读2.5k次，点赞10次，收藏26次。PaymentSplitter库可以在一组领取地址无感知的情况下，将定量eth或某ERC20 token按照shares占比释放给该组中的某地址。当et...

30. [Smart Contract Audit | v1.3 - Obol Docs](https://docs.obol.org/docs/advanced-and-troubleshooting/security/smart-contract-audit) - Fees are planned to be implemented on the rewardRecipient splitter by updating to a new fee structur...

31. [Curve Fee Splitter - Smart Contract Audit - ChainSecurity](https://www.chainsecurity.com/security-audit/curve-fee-splitter) - Curve implements fee splitter to distribute fees (in crvUSD token) from the crvUSD stablecoin market...

32. [0xSplits - Web3 Wallet Tools - Alchemy](https://www.alchemy.com/dapps/splits) - 0xSplits offers composable, efficient, onchain splits for ETH & ERC20s tokens. Discover 0xSplits and...

33. [Alternative options](https://splits.org/changelog/alternative-options/) - Changelog Jul 21, 2023

34. [Contracts V2 | Monerium](https://docs.monerium.com/contracts-v2/) - What is happening

35. [Monerium EUR emoney (EURe) Token Tracker | GnosisScan](https://gnosisscan.io/token/0xcB444e90D8198415266c6a2724b7900fb12FC56E) - Monerium EUR emoney (EURe) Token Tracker on GnosisScan shows the price of the Token $1.084, total su...

36. [Tokens](https://docs.monerium.com/tokens/)

37. [Building a Dynamic Payment Splitter on Conflux Network ...](https://forum.conflux.fun/t/building-a-dynamic-payment-splitter-on-conflux-network/23109) - Pull Payments: Recipients pay gas for their own withdrawals; Minimal Storage: Only stores essential ...

38. [Payment Splitter - HackQuest](https://www.hackquest.io/projects/Payment-Splitter-4zo34i) - Multi-Party Payment Splitter is a blockchain-based smart contract designed to efficiently and automa...

39. [Ethereum Smart Contract Audit for Drips - Cantina.xyz](https://cantina.xyz/portfolio/47ff4361-233a-42c4-bb83-6a0beab255c8) - Drips is an Ethereum-based protocol for streaming and splitting ... chain ID's is kept up to date an...


