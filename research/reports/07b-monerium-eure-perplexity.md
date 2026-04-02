# Monerium EURe — Perplexity izvještaj

**Prompt:** [07-monerium-eure-gnosis.md](../prompts/07-monerium-eure-gnosis.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

# Monerium EURe na Gnosis lancu – tehničko, regulatorno i integracijsko istraživanje

## 1. Tehničke činjenice o EURe na Gnosisu

### 1.1. Adrese ugovora i standard

- Na Gnosis lancu (chain ID 100) povijesna V1 adresa EURe tokena je `0xcB444e90D8198415266c6a2724b7900fb12FC56E`, pod nazivom **Monerium EUR emoney (EURe)**.[^1][^2]
- S V2 nadogradnjom uvedena je nova implementacijska adresa EURe `0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430`; V1 i V2 zajedno održavaju isti balans, pri čemu V1 prosljeđuje pozive na V2 nakon određenog bloka.[^3][^4]
- Monerium navodi da se tokeni temelje na OpenZeppelin bibliotekama i da podržavaju standardni **ERC‑20** skup funkcija, uz dodatnu podršku za **ERC‑2612 Permit** (gasless approvals).[^5][^3]

### 1.2. Funkcije i proširenja (mint, burn, freeze, blacklist)

- Dizajn tokena eksplicitno navodi da je riječ o ERC‑20 tokenu s **multi‑entity minting i burning** funkcionalnošću, **adresno-specifičnim zamrzavanjem** (blacklist/freeze) i mogućnošću nadogradnje ugovora.[^2]
- Standardne ERC‑20 funkcije koje su implementirane su: `totalSupply`, `balanceOf`, `transfer`, `transferFrom`, `approve`, `allowance`.[^2][^5]
- Frontend ugovor (`TokenFrontend`) delegira većinu logike na kontroler (`SmartController` / `StandardController`), ali izlaže dodatne funkcije:
  - `transferAndCall` (ERC‑677 stil),
  - `mintTo` i `mint` (za mintanje, uključujući bridge use‑case),
  - `burnFrom` (uz potpis vlasnika adrese),
  - `recover` (recovery funkcija za slučaj izgubljenog private key‑a, uz korisnički potpis).[^5]
- U pozadini se koristi **BlacklistValidatorUpgradeable**; administratori mogu dodati adresu na blacklistu, čime se takvoj adresi onemogućuje slanje i/ili primanje tokena (efektivno zamrzavanje).[^2][^5]

### 1.3. Uloge i privilegirane funkcije

- Token dizajn definira tri razine uloga:[^5][^2]
  - **Owner** – jedna adresa (Gnosis Safe multisim) kontrolira dodavanje/uklanjanje `admin` i `system` adresa, postavljanje cap‑a za minting allowance te postavljanje validatora (blacklista).
  - **Admin** – može povećavati mint allowance za `system` adrese te **dodavati/uklanjati adrese na blacklistu**, tj. upravljati zamrzavanjem.[^2]
  - **System** – operativne adrese koje Monerium backend koristi za **mintanje i burnanje** (izdavanje i otkup e‑novca).[^2]
- Implementacija koristi OpenZeppelin **Ownable2StepUpgradeable** i **AccessControlUpgradeable**; vlasnik (`owner`) ima ekskluzivno pravo inicirati nadogradnje implementacijskog ugovora i dodjeljivati `admin`/`system` role.[^5]
- Logika tokena uključuje i **Pausable** mehanizam; vlasnik može `pause` i `unpause` prijenose na razini kontrolera, što omogućuje globalno zaustavljanje transfera u ekstremnim situacijama.[^5]

### 1.4. Nadogradivost i proxy arhitektura

- V2 arhitektura koristi **UUPS proxy pattern** (OpenZeppelin UUPS) u kombinaciji s **ERC1967Proxy** instancama; svaka emisija (npr. EURe) ima svoj proxy koji delegira pozive na zajednički implementacijski ugovor `Token.sol`.[^5]
- Sva četiri Monerium tokena (EURe, USDe, GBPe, ISKe) u jednom okruženju dijele zajednički **BlacklistValidatorUpgradeable** ugovor, što znači da je blacklist globalan za sve Monerium tokene na tom lancu.[^5]
- Nadogradnje se provode tako da se implementacija zamijeni novom verzijom, dok proxy i storage ostaju isti; pravo na nadogradnju ima samo `owner` adresa.[^5]
- Migracija na V2 na Gnosis lancu dovršena je 25. 8. 2024., uz smanjenje gas troškova za `transfer` i `approve` od preko 60% u odnosu na V1.[^4][^3]

### 1.5. Opticaj i dnevni volumen na Gnosisu

- GnosisScan token tracker za adresu `0xcB444e90...FC56E` (V1 frontend za EURe) prikazuje **ukupnu ponudu** od približno **6 019 657,90 EURe** na Gnosis lancu, s preko 10 tisuća holdera.[^1]
- Na istoj stranici vidljiv je 24‑satni DEX volumen za Balancer V2 EURe parove na Gnosisu od oko 155 tisuća + 118 tisuća + 42 tisuće USD, što daje približno **316 tisuća USD** dnevnog volumena na Gnosisu (samo na Balanceru).[^1]
- Ukupna cirkulirajuća ponuda EURe preko svih lanaca iznosi oko **23,35 milijuna tokena**, što implicira da je Gnosis značajan ali ne dominantan dio ukupne distribucije.[^6]
- Važno je napomenuti da je to vidljiva DEX likvidnost; ukupni on‑chain volumen (P2P plaćanja, interni prijenosi) na Gnosisu nije javno agregiran u jednom izvoru i morao bi se dobivati iz prilagođenih analitičkih upita.

## 2. Regulatorna situacija: MiCA, KYC i zamrzavanje

### 2.1. Status EURe kao e‑novca / EMT pod MiCA

- EURe je **e‑money token (EMT)** denominiran u eurima, izdan od strane **Monerium EMI ehf.**, institucije za elektronički novac licencirane u EGP‑u i nadzirane od strane Centralne banke Islanda.[^7][^8][^9]
- Kao EMT, EURe je nespekulativni instrument: svaki token predstavlja **pravno potraživanje za 1 EUR** u segregiranim, visoko likvidnim rezervama (bankovni depoziti i HQLA), a holderi nemaju pravo na kamate.[^8][^7]
- MiCA (Regulation (EU) 2023/1114) zahtijeva da EMT izdavatelji drže rezerve 1:1 i omogućuju pravo na otkup u bilo kojem trenutku po nominalnoj vrijednosti, uz stroge zahtjeve u pogledu zaštite korisnika i informiranja.[^10][^11]

### 2.2. KYC/AML postupak i trajanje

- Kao EMI i MiCA‑uskladjeni izdavatelj, Monerium mora provoditi **standardni KYC/AML** za sve korisnike (osobne i poslovne), uključujući identifikacijske dokumente, provjeru PEP/sanction lista i, po potrebi, dokaz o porijeklu sredstava.[^12][^7]
- Marketinški materijali i tutorijali prikazuju onboarding kao proces u kojem korisnik:
  1. otvori račun,
  2. poveže EVM wallet adresu,
  3. završi online identifikaciju (ID + selfie),
  4. dobije osobni IBAN vezan na tu adresu – često unutar jedne online sesije.[^13][^14]
- Iako Monerium navodi da je **prosječno vrijeme integracije za B2B partnere oko 4 tjedna**, za krajnje korisnike (osobe / manji biznisi) onboarding je projektiran da bude “u minutama”, ali stvarno trajanje može varirati ovisno o kompleksnosti profila i ručnim provjerama.[^14][^15]

### 2.3. SEPA → EURe i EURe → SEPA: pravni i operativni okvir

- Monerium osigurava **osobni IBAN** koji je eksplicitno mapiran na određenu EVM adresu (uključujući Safe) – uplata SEPA transfera na taj IBAN rezultira automatskim mintanjem EURe na povezanu adresu, dok slanje EURe nazad preko Monerium API‑ja dovodi do burnanja i isplate EUR na odredišni bankovni račun.[^16][^17][^15]
- Ove operacije se pravno tretiraju kao izdavanje i otkup e‑novca od strane EM‑institucije, a ne kao kripto‑kripto razmjena, što povlači primjenu pravila o e‑novcu i platnim uslugama (PSD2, MiCA, direktive o e‑novcu i AML okvir).[^7][^8]
- Pravo na otkup EMT‑a u fiat valutu kod izdavatelja zaštićeno je MiCA‑om; relevantne odredbe nalaze se u dijelu koji uređuje EMT izdavatelje (naslovi III i IV MiCA‑e), a dopunjene su pravilima o zaštiti sredstava i prioritetnom potraživanju u slučaju insolventnosti.[^18][^11][^10]

### 2.4. Pravo i obveza zamrzavanja (freeze / blacklist) u pravu i u ugovoru

- Smart‑contract arhitektura uključuje **Blacklist** komponentu: administratori mogu dodati određene adrese u zajednički blacklist validator, čime se onemogućava da te adrese šalju ili primaju Monerium tokene.[^2][^5]
- Moneriumovo priopćenje povodom incidenta s LCX‑om (kompromitirani hot‑wallet) jasno navodi da je EURe iz kompromitirane adrese „zamrznut“, a ekvivalentan iznos ponovno izdan na novu LCX adresu, dok se tokeni na staroj adresi **više ne priznaju kao e‑novac**.[^19]
- U istom priopćenju Monerium izričito navodi da zadržava pravo **blacklistati ili zamrznuti adrese** u slučaju:
  1. potencijalnog sigurnosnog incidenta ili prijetnje za Monerium ili korisnike;
  2. potrebe usklađivanja s **zakonom, regulacijom ili pravnim nalogom** od strane islandskih vlasti, sudova ili drugih nadležnih tijela.[^19]
- U uvjetima korištenja Monerium naglašava da se sredstva mogu zamrznuti ili mora predati čuvene (safeguarded) rezerve **na temelju pravomoćnog naloga nadležnog tijela**, uključujući slučajeve sumnje na pranje novca, financiranje terorizma ili druge kaznene radnje.[^20]

### 2.5. MiCA ovlasti nadzornih tijela za prekid aktivnosti

- MiCA u **Titulu VII** definira ovlasti nadležnih tijela; **članak 102** („Precautionary measures“) omogućuje nacionalnim regulatorima da izdavatelju ili pružatelju usluga **privremeno zabrane ili ograniče** izdavanje, ponudu ili pružanje usluga vezanih uz određene tokene kada postoji ozbiljna zabrinutost za zaštitu ulagatelja ili stabilnost tržišta.[^21][^22]
- **Članak 105** predviđa „product intervention“ ovlasti – nacionalna tijela, ESMA i EBA mogu **zabraniti ili ograničiti marketing, distribuciju ili prodaju** određenih asset‑referenced ili e‑money tokena, ili određenih aktivnosti povezanih s njima, radi zaštite ulagatelja ili očuvanja financijske stabilnosti.[^23]
- Iako MiCA ne specificira „zamrzavanje pojedine adrese“ na razini smart‑contracta, kombinacija ovlasti iz članaka 102 i 105, zajedno s postojećim AML i e‑money pravilima, u praksi znači da se od Moneriuma može **zahtijevati blokada određenih korisnika ili transakcija**, što se tehnički provodi kroz blacklist/freeze funkcije u ugovoru.[^20][^23][^21]
- Za hrvatske subjekte (npr. HANFA, hrvatski sud) takvi zahtjevi bi se tipično kanalizirali preko home‑regulatora (Centralna banka Islanda) i/ili europskih tijela (ESMA, EBA), a Monerium je obvezan postupiti po **pravosnažnim i nadležnim** nalozima.[^21][^7]

### 2.6. Transparentnost oko zamrzavanja

- Monerium **ne vodi centralno objavljenu listu svih zamrznutih adresa**; poznati slučajevi (poput LCX incidenta) objavljuju se kroz blog priopćenja, ali ne postoji javni registar svih freeze događaja po lancu.[^19]
- Budući da je blacklist implementiran on‑chain kroz posebni validator ugovor, teoretski se može izvesti analiza promjena na tom ugovoru da bi se identificirale blacklistane adrese, ali to zahtijeva custom on‑chain analitiku, a Monerium ne objavljuje službene agregirane brojke po mreži (uključujući Gnosis).[^19][^5]
- Zaključak: za sada nije moguće točno znati „koliko je zamrzavanja bilo na Gnosisu“ bez vlastite analize događaja na BlacklistValidator ugovoru.

## 3. Integracijske mogućnosti: API, SDK, Safe i Gnosis Pay

### 3.1. Monerium API: SEPA ↔ EURe programabilni tokovi

- Monerium izlaže REST API (v1/v2) putem baza URL‑ova `https://api.monerium.app` (production) i `https://api.monerium.dev` (sandbox), s OAuth2 autentikacijom i standardnim JSON payloadima.[^24][^25][^26]
- API podržava:
  - kreiranje i upravljanje korisničkim **profilima** (osoba / tvrtka),
  - **linkanje EVM adresa** uz IBAN putem „linkAddress“ flow‑a (potpis poruke iz walleta),
  - kreiranje **naloga za plaćanje** (orders) gdje izvor može biti EURe na određenoj adresi, a odredište IBAN, ili obratno preko SEPA push transfera.[^17][^26]
- U developer dokumentaciji eksplicitno se navodi **sandbox okruženje** za web i API (`monerium.dev` / `api.monerium.dev`), kao i podržane mreže: Ethereum Goerli, Polygon Mumbai, Gnosis Chiado u sandboxu te Ethereum, Polygon i Gnosis u produkciji.[^26]
- Time je jasno da **postoji API za programsku konverziju SEPA → EURe i EURe → SEPA**, uključujući mogućnost da backend vaše aplikacije automatski otkriva uplate na korisničke IBAN‑e i generira odgovarajuće tokove na Gnosis adresama.[^16][^17]

### 3.2. Mogućnost izravnog SEPA u EURe na Gnosis adresu

- Monerium u više izvora opisuje model „Web3 IBAN‑a“: korisnik dobiva jedinstveni IBAN koji je **mapiran na njegovu wallet adresu**; svaka SEPA uplata na taj IBAN automatski se tokenizira kao EURe na specificiranoj adresi, na odabranoj mreži (uključujući Gnosis).[^27][^28][^16]
- Case study s Gnosis Pay detaljno opisuje kako korisnik šalje EUR na svoj Monerium IBAN, a Monerium po primitku automatski mint­a EURe u njegov Gnosis Pay (Safe) walletu – potpuno bez potrebe za CEX‑om ili manualnim bridgeanjem.[^15]
- Isto vrijedi i za obične Monerium korisnike: nakon što je određena Gnosis adresa povezana i omogućena, SEPA uplata na pripadajući IBAN će rezultirati EURe mintanjem na toj adresi.[^26][^16]

### 3.3. Naknade za konverziju i korištenje

- Službena Monerium stranica o naknadama navodi da **trenutno ne naplaćuju**:
  - otvaranje i održavanje jednog IBAN‑a po korisniku,
  - **SEPA prijenose** između bankovnih računa i Web3 walleta korisnika,
  - transakcijske naknade za mintanje i burnanje EURe,
  - pristup **developer API‑ju**.[^29]
- Monerium zadržava pravo uvesti naknade u budućnosti, ali u trenutnom modelu osnovni SEPA ↔ EURe tokovi su **bez naknade s njihove strane**, uz naravno mrežne fee‑ove na Gnosisu (xDAI za gas) i eventualne bankarske troškove na strani korisnika.[^29][^15]

### 3.4. Monerium SDK za JavaScript/TypeScript

- Dostupan je službeni **@monerium/sdk** paket na npm‑u; dokumentacija navodi da je to „essential tools to interact with the Monerium API, an electronic money issuer“.[^30]
- SDK uključuje:
  - `MoneriumClient` klasu za upravljanje OAuth flow‑om (`getAccess`, `authorize`, `getProfiles`),
  - util funkcije za linkanje adresa (potpis standardizirane poruke),
  - tipove za profiles, orders i druge entitete.[^30][^26]
- SDK eksplicitno podržava okruženja `sandbox` i `production` te mreže Ethereum, Polygon i Gnosis; primjeri pokazuje inicijalizaciju s `environment: 'sandbox'` i rad s TS tipovima.[^26][^30]

### 3.5. Integracija sa Safe{Wallet} i Safe Appom

- Monerium ima **službenu Safe App integraciju**; Safe App se može otvoriti preko URL‑a `https://app.safe.global/apps/open?appUrl=https://monerium.app`, a manifest definira podršku za više mreža, uključujući Gnosis (chain id 100).[^31][^32]
- Moneriumov vodič „Getting started with Safe{Wallet}“ preporučuje upravo korištenje Safe Appa na desktopu; Safe automatski detektira vaš Safe wallet, nakon čega se nastavlja standardni Monerium onboarding i povezivanje IBAN‑a.[^31]
- Safe{Core} Account Abstraction SDK ima **Monerium pack** u onramp kitu, koji omogućuje developerima da implementiraju tokove u kojima se iz Safe računa programatski kreiraju SEPA nalozi (i obratno) koristeći EURe i Monerium IBAN.[^33]
- Safe i Monerium zajednički komuniciraju da ova integracija omogućuje 1‑klik onramp/offramp iz Safe smart računa u bankovni račun, bez potrebe za CEX‑om ili dodatnim kustodijalnim rješenjima.[^34][^35][^33]

### 3.6. Gnosis Pay i veza s EURe/Safe

- Gnosis Pay card wallets su Safe smart accounti koji su **direktno povezani s osobnim Monerium IBAN‑om** svakog korisnika.[^15]
- Tok „top‑upa“:
  - korisnik šalje EUR putem SEPA na svoj Monerium IBAN,
  - Monerium odmah i besplatno mint­a **EURe** u njegov Gnosis Pay Safe wallet,
  - sredstva su kolateralizirana 102% kroz bankovne depozite i HQLA.[^15]
- Kod trošenja karticom:
  - card issuer (Gnosis Pay) provjerava on‑chain EURe saldo u Safe walletu u realnom vremenu,
  - Safe omogućava issueru da inicira on‑chain transfer EURe s korisnikova Safe‑a na issuerov Safe za potrebe Visa namire,
  - issuer zatim koristi vlastiti Monerium IBAN za slanje EUR Visa schému.[^15]
- Ovo pokazuje da je kombinacija **EURe + Safe + Monerium API + Gnosis Pay** već battle‑tested za vrlo sličan use‑case kao što je vaše podcast tržište (on‑chain prihodi, off‑ramp u fiat, kartično trošenje), samo u B2C kontekstu.[^28][^15]

### 3.7. Kompatibilnost s pametnim ugovorima i Safe patternima

- Budući da je EURe standardni ERC‑20 (uz dodatne funkcije), radi **bez posebne konfiguracije** sa Safe{Wallet} – token će se pojaviti čim na Safe adresu stigne prvi EURe ili ga ručno dodate kroz custom token adresu.[^31][^2]
- ERC‑20 funkcije `approve` i `transferFrom` implementirane su kroz standardnu ERC‑20 biblioteku (`ERC20Lib`), pa su **kompatibilne s klasičnim patternom razdjelnika plaćanja** (spender ugovori koji troše allowance).[^2][^5]
- Dodatne točke pažnje:
  - zbog **blacklist/validator** logike, transfer će revertati ako je pošiljatelj ili primatelj na blacklisti, što može uzrokovati failanje cijele transakcije vašeg pametnog ugovora,[^19][^5]
  - postoji `recover` funkcija kojom Monerium može, uz odgovarajući korisnički potpis, premjestiti tokene s jedne adrese na drugu – to je bitno za model gubitka ključeva, ali i kao centralizirana intervencijska točka.[^2][^5]
- U javnim izvorima nema zabilježenih „kompatibilnosnih“ bugova EURe‑a s Safeom ili tipičnim DeFi ugovorima; LlamaRisk i Aave research fokusiraju se više na **likvidnost** i centralizaciju mint/burn procesa nego na tehničke nekompatibilnosti.[^36][^37]

## 4. Ključni rizici za podcast tržište na EURe

### 4.1. Rizik zamrzavanja adresa (freeze / blacklist)

- Monerium ima tehničku mogućnost i ugovorno/zakonsko pravo **zamrznuti pojedinačne adrese** putem blacklist validatora; takva adresa više ne može slati niti primati EURe.[^19][^5]
- U slučaju da vaš **razdjelnik plaćanja** ili treasury Safe bude dodan na blacklistu (npr. zbog sumnje na AML povredu, greškom ili temeljem naloga), sve transakcije koje uključuju tu adresu će revertati:
  - u praksi to znači da uplate više neće moći biti isplaćene dalje, a svi EURe koji završe na toj adresi ostali bi „zaglavljeni“ on‑chain (i najvjerojatnije nerefundabilni u fiat, jer ih Monerium ne bi priznao kao valjane e‑novčane tokene).[^20][^19][^5]
- Za tvoj use‑case to implicira da bi ključne adrese (npr. escrow/razdjelnik, treasury Safe) trebalo:
  - voditi na čisto KYC‑anoj tvrtki ili udruzi s konzervativnim compliance profilom,
  - razmotriti **segmentaciju rizika** (npr. zasebni splitter za „high‑risk“ izvore prihoda, dnevno pražnjenje u „čiste“ trezore).

### 4.2. Ovisnost o Moneriumu kao jedinom issueru

- Monerium je **jedini ovlašteni izdavatelj EURe**; Aave research eksplicitno bilježi da je supply dinamičan kroz mint/burn, a mint allowance je pod kontrolom admin role Moneriuma.[^37][^36]
- Ako Monerium izgubi licencu, bude predmet regulatorne zabrane ili jednostavno odluči ugasiti uslugu, on‑off ramp SEPA ↔ EURe bio bi ozbiljno narušen ili onemogućen, iako bi sam token i dalje postojao on‑chain.[^7][^21]
- MiCA i e‑money regulativa daju holderima **prioritetno potraživanje** na rezerve u slučaju insolventnosti, ali operativni prekid bi i dalje izazvao značajnu nelikvidnost i, moguće, diskont na sekundarnom tržištu.[^18][^8][^7]

### 4.3. Rizik promjene ugovora (upgradeability) i governance rizik

- Ugovor je nadogradiv (UUPS/Proxy); vlasnik (Gnosis Safe koji kontrolira Monerium) može:
  - deployati novu implementaciju s promijenjenim pravilima transfera, naknadama ili dodatnim kontrolama,
  - promijeniti blacklist validator ili parametre vezane uz validaciju transfera.
- Iako je Monerium prošao audit (Ackee Blockchain Security) i radi u reguliranom okviru, ova centralizirana kontrola znači da se **tokenova semantika može promijeniti** bez on‑chain glasanja korisnika, što može razbiti neke integracije ako pretpostavljaju „čist“ ERC‑20 behavior.[^3][^5]

### 4.4. Likvidnosni i tržišni rizik

- Ukupna ponuda EURe globalno je relativno skromna (deseci milijuna), a na Gnosisu oko 6 milijuna, s DEX volumenom reda stotina tisuća USD dnevno – dovoljno za B2C plaćanja i srednje use‑caseove, ali ograničeno za vrlo velike protoke ili leveraged DeFi strategije.[^6][^37][^1]
- LlamaRisk u analizi EURe na Linea mreži upozorava na **koncentriran supply** i plitku likvidnost na novim lancima; na Gnosisu je situacija bolja, ali i dalje daleko od likvidnosti USDC/DAI na Ethereum mainnetu.[^37]

### 4.5. Pravna i reputacijska izloženost

- Kao izdavatelj e‑novca, Monerium je dužan agresivno provoditi AML/CFT mjere, uključujući prijave FIU‑u i zamrzavanje sredstava; to može voditi do **„compliance‑overreach“ scenarija** u kojima se adrese blokiraju zbog konzervativnih procjena rizika.[^20][^19]
- Za projekt koji upravlja isplatama kreatora sadržaja (podcast tržište) to znači potrebu za:
  - jasnim **ToS‑om i KYC/AML politikom** vašeg protokola,
  - mogućnošću isključivanja problematičnih korisnika prije nego što dođu na radar Moneriuma,
  - pravnim savjetovanjem o statusu vašeg subjekta (npr. treba li vam lokalna licenca za platne usluge).

## 5. Alternative na Gnosis lancu i usporedba

### 5.1. Koji još EUR stabilni tokeni postoje na Gnosisu?

- **EURS (STASIS EURO)** – fiat‑backed euro stablecoin, izvorno na Ethereumu, dostupan na Gnosisu preko bridgea (xDAI/Omnibridge); STASIS navodi da određeni iznos EURS cirkulira na Gnosisu kao „3rd party bridged“ asset.[^38][^39]
- **EURA / agEUR (Angle Protocol)** – decentralizirani, over‑collateralized euro stablecoin, s primarnom implementacijom na Ethereumu/Polygonu; postoji **agEUR token instanca na Gnosisu** (Angle Protocol: agEUR Token), tipično bridgana, s ograničenom lokalnom likvidnošću.[^40][^41]
- **cEUR / CEUR (Celo Euro)** – Celo euro stablecoin; na Gnosisu se pojavljuje kao **Celo Euro (Wormhole) (CEUR)** na nekim DeFi adresama, ali s vrlo malim obujmom i bez šireg DeFi ekosustava.[^42][^43]
- Uz euro tokene, Gnosis ima bogat ekosustav **USD stablecoina**:
  - **USDC** (u varijantama `USDC.e` i „USDC on xDai (old USDC)“) preko Omnibridge‑a,[^44]
  - **DAI/USDS** (povezani s xDAI bridgeom; xDai je sam po sebi USD‑pegirani native token).[^45][^46]

### 5.2. Usporedna tablica: EURe vs alternativni tokeni na Gnosisu

| Token | Izdavatelj / tip | Regulatorni status | Približna prisutnost na Gnosisu | Mogućnost zamrzavanja (on‑chain) | Napomene |
|------|-------------------|--------------------|----------------------------------|-----------------------------------|----------|
| **EURe** | Monerium EMI, e‑money token (EMT) | EMI licenca + MiCA EMT, rezerve 1:1 (102% HQLA) | Visoka za EUR: nativni deployment, ~6M EURe supply, aktivni Balancer poolovi | **Da** – centralni blacklist validator, admin može blokirati adrese; moguć i recovery i globalni pause | Duboka integracija s SEPA, Safe, Gnosis Pay; centraliziran mint/burn i regulatorno snažno usklađen.[^1][^2][^5][^15] |
| **EURS** | STASIS, fiat‑backed stablecoin | MiCA‑uskladjeni euro stablecoin, redoviti audit rezervi [^38][^47] | Niska na Gnosisu: mali bridged supply prema STASIS transparency stranici | **Da** – kao i ostali centralizirani fiat stablecoini, STASIS može pauzirati/blacklistati adrese (centralizirani issuer) [^48][^49][^50] | Više globalne likvidnosti nego EURe, ali na Gnosisu znatno manje korišten; nema SEPA on/offrampe direktno na Gnosisu. |
| **EURA / agEUR** | Angle DAO, crypto‑collateralized | DeFi protokol, over‑collateralized, bez statusa e‑novca | Umjerena do niska: postoji agEUR instanca, ali bez masivne lokalne likvidnosti | **Ovisno o protokolu** – nema centraliziranog fiat custodiana; governance može mijenjati parametre, ali nema klasične blacklist funkcije kao kod fiat EMT‑ova [^40] | Decentraliziraniji dizajn, ali bez izravnog SEPA on‑offrampa; regulatorno manje „čist“ za B2B fiat use‑caseve. |
| **cEUR / CEUR (bridged)** | Celo Foundation / Mento | Stablecoin na Celo mreži, kripto‑kolateraliziran | Vrlo niska: maleni bridged balans na Gnosisu (Celo Euro (Wormhole)) [^42] | Nema centralnog fiat custodian‑a; rizik i kontrola definirani protokolom Mento | Praktično zanemariva opcija za ozbiljniji EUR volumen na Gnosisu. |
| **USDC (na Gnosisu)** | Circle / Centre, fiat‑backed USD stablecoin | Regulatorno usklađen, licenciran izdavatelj, rezerve 1:1 USD | Visoka za USD: raširena DeFi upotreba, više poolova i integracija | **Da** – USDC ima `freeze` / `blacklist` funkciju; adrese mogu biti blokirane na zahtjev regulatora (globalna blacklist) [^51][^52] | Vrlo likvidna USD opcija; dobar fallback za denominaciju u USD, ali s istim (ili većim) freeze rizikom kao EURe. |
| **DAI / USDS / xDAI** | MakerDAO / Gnosis | DAI kao decentralized stablecoin, xDai kao native chain token | Vrlo visoka za USD‑peg (xDAI kao gas, široko korišten) [^45][^46] | **DAI:** nema centralizirani freeze na korisničkim adresama (decentraliziran protokol); **xDAI:** kontrola nad bridgeom, ali ne nad pojedinačnim ERC‑20 saldima [^53][^45] | Dobar „ne‑freezable“ USD‑peg za DeFi logiku, ali nije euro denominiran; regulatorno manje pogodna baza za direktnu SEPA integraciju. |

### 5.3. Interpretacija za tvoj use‑case

- Ako ti je **prioritet MiCA‑uskladjenost, SEPA on/offramp i euro denominacija**, EURe ima jasnu prednost nad svim alternativama na Gnosisu: jedini je euro token s integriranim IBAN‑om, Gnosis Pay podrškom i dokumentiranim B2B use‑caseovima (treasuries, kartične sheme).[^28][^16][^15]
- Ako želiš **smanjiti zamrzivost** na razini smart‑contracta, decentralizirani euro tokeni poput agEUR nude bolju otpornost na centralni freeze, ali **bez pravnog okvira e‑novca** i bez izravne SEPA integracije; to komplificira isplate kreatorima u fiat banke.[^40][^18]
- Kao „rezervna“ opcija za skladištenje vrijednosti i likvidnost možeš kombinirati EURe s **USDC/DAI/xDAI**:
  - USDC nudi veću globalnu likvidnost, ali s identičnim freeze rizikom kao EURe,[^51][^54]
  - DAI/xDAI su otporniji na centralni freeze, ali nisu prirodno usklađeni s euro denominacijom i SEPA okvirom.[^53][^45]

## 6. Sažetak implikacija za dizajn podcast tržišta

- EURe na Gnosisu pruža **tehnički zreo, MiCA‑uskladjen i SEPA‑integriran** temelj za euro denominirano tržište (mint/burn, Permit, Safe integracija, Gnosis Pay), s dobrim ali ne enormnim likvidnosnim profilom.[^3][^1][^15][^5]
- Ključni kompromis je **centralizirani nadzor** (mint/burn, blacklist, upgrade), koji uvodi freeze rizik za adrese kroz pravne i compliance kanale; taj se rizik može djelomično mitigirati dizajnom arhitekture (segmentacija trezora, monitoriranje, pravni okvir), ali ga se ne može ukloniti.[^19][^2][^5]
- Alternativni euro stablecoini na Gnosisu (EURS, agEUR, CEUR) trenutno ne nude kombinaciju MiCA‑uskladjene e‑novce, duboke integracije sa Safeom i Gnosis Payem te snažne SEPA on‑off rampe; stoga djeluju više kao **sekundarne opcije za diversifikaciju**, a ne kao primarna valuta tvog sustava.[^39][^38][^40]

---

## References

1. [Monerium EUR emoney (EURe) Token Tracker | GnosisScan](https://gnosisscan.io/token/0xcB444e90D8198415266c6a2724b7900fb12FC56E) - Monerium EUR emoney (EURe) Token Tracker on GnosisScan shows the price of the Token $1.084, total su...

2. [smart-contracts/docs/tokendesign.md at main · monerium/smart ...](https://github.com/monerium/smart-contracts/blob/main/docs/tokendesign.md) - Monerium's token adheres to the ERC-20 standard, supporting multi-entity minting and burning, addres...

3. [Contracts V2 | Monerium](https://docs.monerium.com/contracts-v2/) - What is happening

4. [What you need to know about the V2 token contracts - Monerium](https://monerium.dev/docs/contracts-v2) - What you need to know about the V2 token contracts

5. [GitHub - monerium/smart-contracts: ERC20 compatible e-money deployed on Ethereum](http://github.com/monerium/smart-contracts) - ERC20 compatible e-money deployed on Ethereum. Contribute to monerium/smart-contracts development by...

6. [DefiLlama](https://defillama.com/stablecoin/monerium-eur-emoney) - DefiLlama is a DeFi TVL aggregator. It is committed to providing accurate data without ads or sponso...

7. [The Regulated Euro-Backed E-Money Token by Monerium - Bleap](https://www.bleap.finance/blog/all-about-eur-regulated-stablecoin-by-monerium) - EURe is a MiCA-regulated euro token issued by Monerium, fully backed 1:1 by euros. Learn how it work...

8. [EURe - Cryptocurrencies | IQ.wiki](https://iq.wiki/wiki/eure) - Monerium EUR emoney on CoinMarketCap. Dec 9, 2025. [3]. www.bleap.finance. All about EURe, regulated...

9. [All About EURe: The Regulated Euro-Backed E-Money Token by ...](https://www.bleap.finance/pt-pt/blog/all-about-eur-regulated-stablecoin-by-monerium) - EURe is a MiCA-regulated euro token issued by Monerium, fully backed 1:1 by euros. Learn how it work...

10. [Article 49 - mica.wtf](https://www.mica.wtf/mica/title-iv-e-money-tokens-art.-48-48/chapter-1/article-49) - Upon request by a holder of an e-money token, the issuer of that e-money token shall redeem it, at a...

11. [MiCA Regulation (EU 2023/1114) - Full Text](https://www.mica.info) - Read the full text of the Markets in Crypto-Assets Regulation. Browse all 149 articles of the EU cry...

12. [Monerium API v2 Documentation](https://monerium.dev/api-docs/v2) - For KYC Sharing Model: Complete KYC data including details, form and verifications is required. For ...

13. [Getting Started With Monerium: How To Get Your Web3 IBAN](https://www.youtube.com/watch?v=AVRJ8zAJ3xs) - Monerium lets you receive and send euros onchain with your own ... identity verification • Generate ...

14. [#eure | Monerium - LinkedIn](https://www.linkedin.com/posts/monerium_eure-activity-7359554243608899584-_aYO) - Running a Web3 business in Europe? This video shows how to open a Monerium ... Monerium inside Safe:...

15. [How Card Issuers like Gnosis Pay can settle with Visa using stablecoins](https://monerium.com/case-studies/gnosispay/) - This case study explores how Monerium enables Card Issuers like Gnosis Pay to streamline their oncha...

16. [Monerium Developers](https://monerium.dev) - Monerium is a fintech company with the mission of making digital currency accessible, secure, and si...

17. [Walkthrough - API | Monerium](https://monerium.dev/docs/walkthrough) - Walkthrough

18. [[PDF] Regulating Crypto-Assets: Stablecoins - cepInput](https://www.cep.eu/fileadmin/user_upload/cep.eu/Studien/cepInput_Stablecoins/cepInput_Regulating_Crypto-Assets_Stablecoins.pdf) - Similar precautionary measures are also inherent in the business of investment funds and, as investm...

19. [Statement concerning the issue of EURe to LCX AG - Monerium](https://monerium.com/blog/2022/blacklisted/) - The EURe 611,000.- previously at address 0x165402279f2c081c54b00f0e08812f3fd4560a05 are no longer re...

20. [Personal terms of service | Monerium](https://monerium.com/policies/personal-terms-of-service/) - 1. General

21. [Overview of the competent authorities under MiCAR](https://cms.law/en/che/publication/legal-experts-on-markets-in-crypto-assets-mica-regulation/overview-of-the-competent-authorities-under-micar) - Explore EU, EBA, and ESMA powers under MiCAR in our legal experts’ article. It simplifies the regula...

22. [Article 102 - Precautionary measures by competent authorities of ...](https://www.mica.info/article/102/) - Read Article 102 of the Markets in Crypto-Assets Regulation (EU 2023/1114).

23. [[PDF] 174768/EU XXVII. GP](https://www.parlament.gv.at/dokument/XXVII/EU/174768/imfname_11345907.pdf)

24. [Monerium API](https://docs.monerium.com/api/) - API for developers to integrate Monerium payments, wallets, and user onboarding functionalities into...

25. [Using the API - Monerium](https://monerium.dev/docs/api) - API consumers can authenticate themselves by providing an OAuth 2.0 access token according to the Th...

26. [Monerium SDK Documentation - GitHub Pages](https://monerium.github.io/sdk/) - Documentation for Monerium SDK

27. [Crypto iban powered by Monerium. : r/ethereum - Reddit](https://www.reddit.com/r/ethereum/comments/1jvbupy/crypto_iban_powered_by_monerium/) - Monerium then mint a ERC20 token called EURe, which represents that €1. You can now use this EURe to...

28. [EURe on Gnosis Chain](https://www.gnosis.io/blog/eure-on-gnosis-chain) - One of the unique features of the EURe issued by Monerium is the ability to send and receive euros d...

29. [Fee schedule | Monerium](https://monerium.com/fee-schedule/) - Currently, Monerium is not charging any fees. That includes: One IBAN per customer; SEPA transfers f...

30. [Monerium SDK Documentation - NPM](https://www.npmjs.com/package/@monerium/sdk) - Essential tools to interact with the Monerium API, an electronic money issuer.. Latest version: 3.4....

31. [Getting started with Safe{Wallet} - Monerium Knowledge Base](https://help.monerium.com/article/4-getting-started-with-safe-wallet) - Desktop: Recommended: Use the Safe App. Launch Monerium inside the Safe App for seamless integration...

32. [Update Monerium - Add networks/chains · Issue #450 · safe-global ...](https://github.com/safe-global/safe-apps-list/issues/450) - Manifest.json URL: https://monerium.app/manifest.json. Name: Monerium ... The app can be loaded as a...

33. [Connecting Safes to Euro IBAN accounts](https://safe.mirror.xyz/4pgiJAEQ2Jt0ij9Ezc8FSOSiRSfVY4Im8FZ0LuICx-8) - Safe{Core} account abstraction SDK adds the Monerium pack to enable the seamless onramp and offramp ...

34. [Monerium Joins Forces With Safe To Bridge Gap Between Crypto ...](https://uk.investing.com/news/cryptocurrency-news/monerium-joins-forces-with-safe-to-bridge-gap-between-crypto-and-european-banks-3075276) - The integration of Monerium's services would allow crypto-native organizations to reduce their relia...

35. [Safe Wallet Provider Teams Up with Monerium to Incorporate $60B Digital Assets into European Banking System](https://www.binance.com/en-ZA/square/post/809860) - According to a report published by CoinTime, smart wallet provider Safe has joined forces with Moner...

36. [[ARFC] Add EURe to Linea V3 Instance - Governance - Aave](https://governance.aave.com/t/arfc-add-eure-to-linea-v3-instance/21840) - Monerium is the sole issuer of EURe. Total supply varies based on a mint and burn mechanism, with a ...

37. [[ARFC] Add EURe to Linea V3 Instance](https://governance.aave.com/t/arfc-add-eure-to-linea-v3-instance/21840/3) - Summary LlamaRisk does not recommend onboarding EURe to Aave V3 Linea due to critical liquidity conc...

38. [STASIS EURO - Cryptocurrencies - IQ.wiki](https://iq.wiki/wiki/stasis-euro) - In 2021, EURS became available on Ethereum layer-2 chains like Arbitrum, Polygon, Gnosis Chain, Algo...

39. [Transparency and reserve verification - Stasis](https://stasis.net/transparency) - Gnosis Chain (xDAI). via 3rd party bridge*. € 357.850.00%. *Users have the option to bridge EURS to ...

40. [EURA (agEUR) Stablecoin Explained: How Angle's Euro ... - Bleap](https://www.bleap.finance/blog/eura-ageur-stablecoin-explained) - Fully on-chain, over-collateralized, and governed by Angle DAO. Used in DeFi, payments, and non-cust...

41. [Address: 0xeeD3BCA1...00D7446CC | GnosisScan](https://gnosisscan.io/address/0xeeD3BCA1358c2b677304bF6eE14998F00D7446CC) - Multichain Info · 0x630e98545bb5931a9e9d58e31910d43434c9443d6ea79b68d6401499268f7d74 · Angle Protoco...

42. [CoW Protocol: Settlement | Address: 0x9008D19f...10560ab41](https://gnosisscan.io/address/0x9008D19f58AAbD9eD0D60971565AA8510560ab41) - XDAI Balance. Gnosis Chain Logo 0 ... Celo Euro (Wormhole) (CEUR). <0.01%, $1.16, 123.1421, $142.84....

43. [Celo: cEUR Token | Address: 0xd8763cba...ed6d6ca73 | CeloScan](https://celoscan.io/address/0xd8763cba276a3738e6de85b4b3bf5fded6d6ca73) - Celo: cEUR Token. $0.000 CELO, 0.00142253, 25.001 ... Gnosis (0). 0 (0%). Linea (0). 0 (0%). Moonbea...

44. [Omnibridge | Gnosis Chain](https://docs.gnosischain.com/bridges/About%20Token%20Bridges/omnibridge) - Omnibridge a native token bridge that mints the canonical representations of bridged assets on Gnosi...

45. [xDai Bridge | Gnosis Chain](https://docs.gnosischain.com/bridges/About%20Token%20Bridges/xdai-bridge) - The xDai bridge is a native Dai bridge from Ethereum that is used to mint and burn xDai, the native ...

46. [xDai Token - Gnosis Chain](https://docs.gnosischain.com/about/tokens/xdai) - xDai tokens are transactional tokens on Gnosis and also used to pay for execution of smart contracts...

47. [EURS](https://stasis.net/eurs/) - EURS is the most transparent Europe-wide digital currency and the single most popular non-USD stable...

48. [Are Stablecoins Freezable? | Markose Chenthitta - LinkedIn](https://www.linkedin.com/posts/markosechenthitta_are-stablecoins-freezable-even-in-non-custodial-activity-7386997310003515392-58x-) - (STASIS) - EURCV (SocGen-Forge) - VNX Euro (VNX) - VNX GBP ... blacklist specific wallet addresses. ...

49. [Stablecoins 101: Behind crypto's most popular asset - Chainalysis](https://www.chainalysis.com/blog/stablecoins-most-popular-asset/) - Examples include the USD-pegged Tether (USDT) and USD Coin (USDC) and EUR-pegged Stasis Euro (EURS)....

50. [Fiat-backed stablecoins, explained - Kraken](https://www.kraken.com/learn/fiat-backed-stablecoins-explained) - Stasis Euro (EURS): EURS is the largest stablecoin pegged to the ... Companies issuing stablecoins c...

51. [Can USDC Be Frozen? How Blacklisting Works - USDC.org](https://usdc.org/glossary/freeze) - USDC addresses can be frozen for sanctions compliance. Learn how the freeze function works, when it'...

52. [CENTRE appears to have blacklisted an address holding USDC for ...](https://finance.yahoo.com/news/centre-appears-blacklisted-address-holding-194053348.html) - ... smart contracts interact with Ethereum, we can see the address named "blacklister" matches the o...

53. [Can DAI be blocked frozen? : r/MakerDAO - Reddit](https://www.reddit.com/r/MakerDAO/comments/l0pfjl/can_dai_be_blocked_frozen/) - Dai is completely decentralized and cannot be blocked or frozen (unlike other stablecoins like USDC)...

54. [USDC blacklists an address for the first time - CoinGeek](https://coingeek.com/usdc-blacklists-an-address-for-the-first-time/) - A blacklisted USDC address can no longer receive USDC, and all of the USDC controlled by the address...


