# GDPR i hashirani identifikatori — Perplexity izvještaj

**Prompt:** [06-gdpr-hash-identifikatori.md](../prompts/06-gdpr-hash-identifikatori.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

# GDPR, hashirani identifikatori i blockchain: pravni status i dizajnerske preporuke

## 1. Uvod i okvir problema

Projekt razmatra javno zapisivanje `keccak256(hash(OIB))` ili sličnog hashiranog osobnog identifikatora u pametni ugovor na javnoj, nepromjenjivoj Gnosis blockchain mreži kao mehanizam protiv Sybil napada (jedna fizička osoba = jedan identitet).
Takva obrada nužno se promatra kroz Opću uredbu o zaštiti podataka (GDPR) i hrvatski Zakon o provedbi Opće uredbe o zaštiti podataka (NN 42/18), pri čemu je ključno: (a) je li hash osobni podatak, (b) kako se na nepromjenjivi blockchain primjenjuje pravo na brisanje, (c) trebate li DPIA, (d) koja je odgovarajuća pravna osnova i (e) je li ZK‑nullifier pristup povoljniji.
[^1][^2][^3]


## 2. Hashirani identifikatori kao osobni podaci

### 2.1. Definicija osobnog podatka i pseudonimizacije

GDPR u članku 4(1) definira osobni podatak kao svaku informaciju koja se odnosi na identificiranu ili **identificirljivu** fizičku osobu, uključujući identifikacijski broj ili mrežni identifikator.[^2][^4]
Pseudonimizacija je definirana u članku 4(5) kao obrada pri kojoj se osobni podaci više ne mogu pripisati određenom ispitaniku bez dodatnih informacija, pod uvjetom da se te dodatne informacije drže odvojeno i pod tehničkim i organizacijskim mjerama.[^5][^2]
Recital 26 dodatno pojašnjava da se **pseudonimizirani podaci koji se mogu pripisati osobi korištenjem dodatnih informacija i dalje smatraju osobnim podacima**.[^6]


### 2.2. Presuda CJEU u predmetu Breyer (C‑582/14)

U predmetu *Breyer* Sud EU je utvrdio da dinamička IP adresa može biti osobni podatak za pružatelja online usluge ako on, uz pomoć treće strane (ISP‑a), ima pravnu i praktičnu mogućnost identificirati osobu korištenjem „sredstava koja se **razumno vjerojatno mogu upotrijebiti**“.[^7][^8][^9]
Sud je time potvrdio **relativni test identifikabilnosti**: gleda se koje su mogućnosti za identifikaciju s gledišta konkretnog voditelja obrade, uključujući dostupne treće strane, trošak i vrijeme potrebne identifikacije.[^8][^7]
Ovo je presudno za hashirane identifikatore: ako ih se s razumnim naporom može povezati s OIB‑om ili osobom, ostaju osobni podaci.


### 2.3. EDPB i WP29 o hashiranju i anonimizaciji

Radna skupina članka 29 (danas EDPB) u Mišljenju 05/2014 o tehnikama anonimizacije naglašava da **hashiranje izravnog identifikatora obično predstavlja pseudonimizaciju, a ne stvarnu anonimizaciju**, zbog rizika napada rječnikom, brute‑force napada i spajanja s drugim skupovima podataka.[^10][^11][^12]
Recital 26 postavlja vrlo visok prag za anonimizaciju: moraju se uzeti u obzir sva sredstva koja se razumno vjerojatno mogu upotrijebiti za identifikaciju, uzimajući u obzir trošak, vrijeme i raspoloživu tehnologiju, a osobni podaci moraju biti „učinjeni anonimnima tako da ispitanik **više nije identificiran**“.[^13][^6]
EDPB u novim Smjernicama 01/2025 o pseudonimizaciji potvrđuje da je pseudonimizacija **zaštitna mjera unutar GDPR‑a**, ali da pseudonimizirani podaci i dalje jesu osobni podaci za subjekt koji posjeduje dodatne informacije.[^14][^15][^16]


### 2.4. EDPS/AEPD dokument o hash funkciji

Zajednički dokument španjolskog nadzornog tijela (AEPD) i Europskog nadzornika za zaštitu podataka (EDPS) o hash funkciji naglašava da je hash tipična tehnika **pseudonimizacije**, a ne automatske anonimizacije.[^17][^18]
Dokument traži **objektivnu analizu rizika ponovne identifikacije**, uključujući entropiju originalne poruke (npr. veličina prostora OIB‑ova), znanje o formatu podataka i postojanje popisa ili rječnika koji omogućuju pretraživanje po hashu.[^19][^18]
Preporučuje korištenje soli ili ključa te drugih mjera (npr. enkripcija prije hashiranja) ako se želi približiti anonimizaciji, ali uz upozorenje da to ovisi o konkretnoj implementaciji i kontekstu.[^19][^17]


### 2.5. Stajališta nadzornih tijela (ICO, CNIL, AZOP)

ICO (UK nadzorno tijelo) u svojim smjernicama o anonimizaciji i pseudonimizaciji izričito navodi da je **hashiranje osobnih podataka tipična tehnika pseudonimizacije** te da takvi podaci ostaju u dosegu zakonodavstva o zaštiti podataka za onoga tko ima razumnu mogućnost ponovne identifikacije.[^20][^21][^22][^23]
CNIL u francuskim smjernicama o blockchainu navodi da se na lancu treba čuvati samo kriptografski otisak (hash, kriptografski commitment), ali ističe da i takvi otisci mogu predstavljati osobne podatke ako omogućuju povezivanje s osobom.[^24][^25][^26]
AZOP u svojim objašnjenjima definira osobni podatak u skladu s GDPR‑om i navodi OIB i identifikacijsku oznaku građana kao tipične primjere osobnih podataka; time implicitno potvrđuje da je bilo kakva obrada OIB‑a (pa i hashiranje) obrada osobnih podataka.[^27][^28][^29]


### 2.6. Hashirani OIB na javnom blockchainu

U kontekstu vašeg dizajna, nekoliko je specifičnih elemenata:

- **Prostor vrijednosti OIB‑a je mali i dobro definiran**, pa se `keccak256(OIB)` ili `keccak256(hash(OIB))` bez tajne soli može napasti rječnikom ili brute‑force metodom uz razumno mali trošak.
- Razumno je pretpostaviti da voditelj obrade (npr. platforma koja već zna OIB od KYC‑a) ili treće strane mogu s relativno malim naporom generirati istu hash tablicu i povezati hashirane zapise s konkretnim osobama.
- EDPB u Smjernicama 02/2025 o blockchainu izričito navodi da kada se osobni podaci čuvaju off‑chain, a na lancu se pohranjuje samo hash, **„hash će se također smatrati osobnim podatkom, kao i svaki drugi identifikator koji može postojati“** u odnosu na onoga tko ima pristup izvoru i/ili ključu.[^30][^3]

S obzirom na ovo, `keccak256(hash(OIB))` pohranjen na javnom lancu **jest osobni podatak za voditelja obrade i sve aktere koji s razumnim sredstvima mogu povezati hash s osobom**, te predstavlja pseudonimizirane osobne podatke u smislu članka 4(5) GDPR‑a.
[^3][^18][^10]


### 2.7. Može li jaki jednosmjerni hash biti anonimizacija?

GDPR ne priznaje čisto tehničko svojstvo „jednosmjernosti“ kao dovoljno za anonimizaciju; ključan je **kontekst identifikabilnosti** prema Recitalu 26 (sva sredstva razumno vjerojatno dostupna voditelju ili drugima).[^6][^13]
Čak i kada se koristi snažan hash algoritam (npr. Keccak‑256), ako je ulazni prostor mali i poznat (OIB, matični broj) ili ako voditelj obrade ima ili može dobiti originalne vrijednosti, podaci ostaju pseudonimizirani, ne anonimni.[^18][^10]
Smjernice AEPD/EDPS i WP29 navode da se hash može približiti anonimizaciji tek ako se: (a) koristi dovoljno jaka, tajna i jedinstvena sol/ključ, (b) svi ključevi i pomoćni podaci nepovratno unište i (c) čak i uz spajanje s drugim skupovima podataka više ne postoji realističan put do identifikacije.[^17][^13][^19]


## 3. Blockchain i pravo na brisanje (članak 17)

### 3.1. Nemogućnost brisanja na javnim lancima

Članak 17 GDPR‑a propisuje pravo ispitanika na brisanje („pravo na zaborav“) u nizu slučajeva (npr. povlačenje privole, prestanak potrebe za obradom, nezakonita obrada), a voditelj obrade tada ima obvezu **faktički izbrisati** osobne podatke bez nepotrebnog odgađanja.[^31][^32]
Javni blockchaini su po dizajnu **nepromjenjivi i distribuirani**; podaci replicirani na tisuće čvorova ne mogu se selektivno ukloniti bez rušenja konsenzusa i povijesti lanca, što pravna i tehnička literatura prepoznaje kao izravan sukob s člankom 17.[^33][^34][^35]
EDPB u Smjernicama 02/2025 o blockchainu naglašava da tehnička nemogućnost brisanja **ne oslobađa** voditelja obrade obveze usklađivanja s GDPR‑om: ako zbog arhitekture lanca ne može ispuniti prava ispitanika, takvu arhitekturu ne bi trebao koristiti za obradu osobnih podataka.[^36][^37][^3]


### 3.2. CNIL smjernice o blockchainu i pravu na brisanje

CNIL u smjernicama „Blockchain and the GDPR: Solutions for a responsible use…“ eksplicitno navodi da je **tehnički nemoguće ispuniti zahtjev za brisanjem podataka koji su jednom upisani u blockchain**.[^25][^26]
Kao djelomična rješenja CNIL predlaže: (a) pohranu osobnih podataka izvan lanca (off‑chain), (b) pohranu samo kriptografskih otisaka (hash, commitment) na lancu i (c) korištenje kriptografskih tehnika (enkripcija, hash s ključem) uz naknadno **uništenje ključa** kao funkcionalni ekvivalent brisanja.[^26][^24]
CNIL ističe da takva rješenja **ne rezultiraju potpunim brisanjem** u smislu fizičkog uklanjanja podataka iz lanca, ali mogu ispitanika dovesti „bliže učinkovitom ostvarivanju prava na brisanje“ blokiranjem pristupa ili de‑faktom onemogućavanjem čitljivosti podataka.[^25][^26]


### 3.3. EDPB Smjernice 02/2025 o hashiranim podacima na lancu

EDPB u Smjernicama 02/2025 o obradi osobnih podataka putem blockchain tehnologija preporučuje da se **osobni podaci, gdje god je moguće, uopće ne pohranjuju on‑chain**, već da se koriste off‑chain rješenja i on‑chain reference.[^38][^37][^3]
Smjernice navode da, kada se osobni podaci čuvaju off‑chain, a na lancu se čuva samo hash, sam hash **i dalje predstavlja osobni podatak** za one koji imaju pristup izvornim podacima ili ključu.[^3]
Ističe se i da, nakon brisanja off‑chain podataka i povezanog ključa/soli, hash **može** postati anoniman, ali **samo ako su ispunjeni strogi uvjeti anonimizacije iz Recitala 26**, što zahtijeva zasebnu tehničku i pravnu procjenu.[^15][^3][^6]


### 3.4. „Soljeni hash“ i brisanje

Korištenje „soljenog hasha“ (`hash(salt || OIB)`) gdje je sol tajna koju poznaje samo korisnik značajno otežava napad rječnikom, ali pravni učinak ovisi o tome tko ima kakav pristup:

- Ako voditelj obrade **nikada nema pristup soli ni OIB‑u**, a na lancu postoji samo hash iz kojeg nitko razumno ne može rekonstruirati identitet, za tog voditelja podaci mogu biti izvan dosega GDPR‑a (anonimni).[^39][^6]
- Ako voditelj ili partneri **u nekom trenutku obrađuju OIB i/ili sol** (npr. tijekom registracije i izdavanja SBT‑a) i mogu kasnije povezati hash s osobom (čak i samo kroz vlastite logove/protokole), hash će za njih ostati pseudonimizirani osobni podatak.[^10][^18]
- Ako jedino korisnik posjeduje sol, a odluči je nepovratno uništiti, tada se s gledišta voditelja obrade može argumentirati da više ne postoje razumno vjerojatna sredstva za re‑identifikaciju i da preostali hash više nije osobni podatak, ali takav zaključak zahtijeva konzervativan rizik‑assessment i vjerojatno potvrdu nadzornog tijela.[^13][^3][^19]

Zaključno, salting i uništenje soli **mogu pomoći** u približavanju zahtjevima brisanja, ali ne mijenjaju činjenicu da je do trenutka uništenja soli obrada bila obrada osobnih podataka i da arhitektura mora omogućiti prekid daljnjeg korištenja hash‑zapisa (npr. onemogućavanjem verifikacije povezanih dokaza).
[^32][^3]


## 4. DPIA (procjena utjecaja na zaštitu podataka)

### 4.1. Kada je DPIA obvezan

Članak 35 GDPR‑a propisuje da je DPIA obvezan kada je neka vrsta obrade, osobito uz primjenu novih tehnologija, **vjerojatno rizična po prava i slobode ispitanika** s obzirom na njezinu prirodu, opseg, kontekst i svrhe.[^40][^41][^42]
Uobičajeni primjeri obvezne DPIA‑e uključuju: sustavno i opsežno profiliranje s pravnim ili sličnim učincima, obradu posebnih kategorija podataka u velikom opsegu i sustavno praćenje javno dostupnih područja u velikom opsegu.[^41][^43]
Radne skupine i nadzorna tijela dodatno su definirali kriterije za „vjerojatno visok rizik“ (npr. inovativna uporaba tehnologije, obrada identifikatora u velikom opsegu, onemogućavanje ostvarivanja prava), a DPIA je preporučena ako je zadovoljena kombinacija dvaju ili više takvih kriterija.[^43][^44]


### 4.2. EDPB i blockchain: DPIA kao de‑facto obveza

EDPB u Smjernicama 02/2025 o blockchainu izričito naglašava da se **DPIA treba provesti prije bilo kakvog blockchain projekta koji uključuje osobne podate**, zbog same prirode tehnologije (immutabilnost, javna dostupnost, distribucija po čvorovima) i potencijalno visokog rizika za prava ispitanika.[^35][^36][^3]
Smjernice sugeriraju da sama uporaba blockchaina u kombinaciji s identifikatorima, osobito na javnim lancima, vrlo često zadovoljava više kriterija visokog rizika (inovativna tehnologija, potencijalno veliki opseg, otežano ostvarivanje prava), čime DPIA u praksi postaje obvezna.
[^37][^3]


### 4.3. Je li vaš sustav kandidat za obvezni DPIA?

Opisani sustav uključuje:

- Korištenje **nove tehnologije** (javni blockchain, pametni ugovori, SBT‑ovi).
- Obradu **pseudonimiziranog identifikatora** koji potječe iz zakonskog identifikacijskog broja (OIB) u potencijalno velikom opsegu.
- Dugotrajnu, praktično trajnu pohranu zapisa na javno dostupnoj mreži.
- Tehničku nemogućnost potpunog brisanja ili ispravljanja zapisa.

Ovi elementi vrlo vjerojatno čine obradu „vjerojatno visokog rizika“, pa je DPIA razumno smatrati **obveznim** prije produkcijskog lansiranja sustava.[^40][^43][^3]


### 4.4. Što DPIA treba sadržavati za ovaj slučaj

Prema članku 35(7) GDPR‑a i smjernicama nadzornih tijela, DPIA za vaš slučaj treba barem:[^42][^41][^40]

- **Opis obrade**: arhitektura (on‑chain/off‑chain), vrste podataka (OIB, hash, adresa novčanika), akteri (voditelj, izvršitelji, čvorovi, korisnici), protok podataka.
- **Svrhe i pravna osnova**: sprječavanje Sybil napada i osiguravanje jedinstvenosti identiteta; obrazloženje odabranog pravnog temelja (ugovor, legitimni interes, privola).
- **Procjenu nužnosti i proporcionalnosti**: zašto je javni blockchain potreban u odnosu na alternativne, manje intruzivne tehnologije (centralizirana baza, permissioned blockchain, ZK‑nullifier sustav).
- **Procjenu rizika za ispitanike**: rizik ponovne identifikacije iz hasha, rizik povezan s javnom trajnom pohranom, mogućnost profiliranja preko adresa, nemogućnost potpunog brisanja.
- **Mjere ublažavanja**: pseudonimizacija (hash + sol), off‑chain pohrana, ZK‑pristup, ograničavanje adresa, upravljanje ključevima, governance, ugovorni odnosi s čvorovima, procesi za odgovaranje na zahtjeve ispitanika.

Brojni izvori (npr. ICO i GDPR.eu) nude generičke predloške DPIA‑a koji se mogu prilagoditi blockchain slučaju.[^41][^42]


## 5. Pravna osnova obrade (članak 6)

### 5.1. Opcije: privola, ugovor, legitimni interes

GDPR predviđa šest pravnih osnova; za vaš slučaj realne su tri:[^45][^46][^47]

- **Privola (čl. 6(1)(a))** – ispitanik slobodno, informirano i nedvosmisleno pristaje na obradu (npr. kovanje SBT‑a temeljem OIB‑a).
- **Izvršenje ugovora (čl. 6(1)(b))** – obrada je nužna radi ispunjenja ugovora u kojem je ispitanik stranka (npr. pristup platformi uz uvjet jedinstvenog identiteta).
- **Legitimni interes (čl. 6(1)(f))** – obrada je nužna za legitimne interese voditelja ili treće strane (npr. zaštita od prijevara i Sybil napada), pod uvjetom da ne nadvladavaju prava ispitanika.[^48][^49]

AZOP u svojim materijalima naglašava da voditelj obrade mora za svaku obradu jasno definirati **odgovarajući pravni temelj** iz članka 6 i ga može opravdati (posebno kod legitimnog interesa kroz test razmjernosti).[^50][^51][^52]


### 5.2. Problemi s privolom na neizbrisivi zapis

Europska i hrvatska praksa tumače privolu kao opozivo pravno utemeljenje: ispitanik je u svakom trenutku ovlašten **povući privolu**, a povlačenje ne utječe na zakonitost obrade prije povlačenja.[^53][^54][^55]
Ako se temeljite isključivo na privoli, a u isto vrijeme znate da **tehnički ne možete izbrisati** hash s javnog lanca, nadzorna tijela mogu zaključiti da privola nije dovoljno „slobodna i informirana“, jer ispitanik zapravo nema stvarnu kontrolu (povlačenje ne proizvodi očekivani učinak potpune obustave obrade).[^32][^3]
Zbog toga nadzorna tijela i stručna literatura često preporučuju da se za neizbrisive zapise (npr. zapise u knjigama poslovnih događaja) oslanja na druge pravne osnove (zakonska obveza, legitimni interes, ugovor), a ne na privolu.
[^55][^56]


### 5.3. Izvršenje ugovora

Ako je jedinstveni on‑chain identitet nužan za pružanje usluge (npr. platforma funkcionira isključivo ako svaki korisnik ima jedinstveni SBT), može se argumentirati da je obrada hashiranog OIB‑a **nužna za izvršenje ugovora** ili za poduzimanje radnji na zahtjev ispitanika prije sklapanja ugovora.[^56][^51][^45]
Ipak, i kod ove osnove ostaje problem članka 17: po prestanku ugovora ili na prigovor ispitanika, voditelj mora moći prestati obrađivati podatke i, ako je primjenjivo, ih izbrisati ili barem učiniti nedostupnima.
Za javni blockchain to u praksi znači da ugovor mora omogućiti **prekid daljnje funkcionalne uporabe** tog identifikatora (npr. onemogućiti daljnju verifikaciju ili upotrebu SBT‑a nakon raskida ugovora).
[^31][^3]


### 5.4. Legitimni interes

Legitimni interes (Sprječavanje prijevara, zaštita od Sybil napada, sigurnost i integritet sustava) često je relevantna osnova, osobito u sigurnosnim kontekstima.[^46][^48]
Da bi se na njega oslonilo, potrebno je provesti **test legitimnog interesa**: (1) jasno definirati interes (sprječavanje više registracija iste osobe), (2) dokazati nužnost takvog oblika obrade (nema jednako učinkovitog, manje intruzivnog rješenja) i (3) pokazati da interes nije nadjačan pravima i slobodama ispitanika.[^57][^49][^58]
S obzirom na trajnu javnu pohranu identifikatora, visoku osjetljivost OIB‑a i mogućnost budućih napada povezivanja, prag za dokazivanje da ovaj interes **nadvladava** prava ispitanika može biti relativno visok, što opet upućuje na to da treba razmotriti dizajne koji uopće ne pohranjuju pseudonimne identifikatore na javnom lancu.
[^59][^35]


## 6. Nullifier i ZK‑pristupi kao alternativa

### 6.1. Koncept ZK‑nullifiera

Protokoli poput Semaphore implementiraju mehanizam „identity commitment“ i „nullifiera“: korisnik generira tajni identitet, na lancu se pohranjuje samo commitment ili se uopće ne pohranjuje izravno, a dokaz članstva i jedinstvenosti (npr. da je glas dao samo jednom) tajno se generira off‑chain i provjerava kao ZK‑dokaz.[^60][^61][^62]
Na lancu se pojavljuje samo kriptografski dokaz (i eventualno nullifier hash korišten za sprječavanje dvostruke upotrebe), koji **nije izravno vezan uz osobni identifikator** kao što je OIB i ne omogućuje praktičnu re‑identifikaciju bez dodatnih informacija koje tipično ne postoje.[^62][^60]


### 6.2. GDPR status ZK‑dokaza i nullifiera

Nema specifičnih presuda CJEU‑a o ZK‑dokazima, ali se na njih primjenjuju opća pravila identifikabilnosti: ako se iz ZK‑dokaza ili nullifiera **realno ne može** identificirati osoba, niti uz razumno vjerojatna sredstva, ti elementi mogu biti anonimizirani podaci i izvan dosega GDPR‑a.[^63][^6]
EDPB u Smjernicama o pseudonimizaciji naglašava da pseudonimizirani podaci ostaju osobni ako ih voditelj obrade može ponovno povezati s osobom, ali prihvaća da za treće strane bez pristupa ključu ili dodatnim informacijama ti isti podaci mogu biti anonimni.[^15][^39]
Ako se arhitektura dizajnira tako da **niti voditelj obrade nema pristup stvarnom identifikatoru**, već samo provjerava ZK‑dokaz jedinstvenosti/članstva, tada se može doći do modela u kojem voditelj uopće ne obrađuje osobne podatke (ili ih obrađuje samo vrlo kratko u KYC fazi kod treće strane koja nije on‑chain).[^61][^35]


### 6.3. Worldcoin / World ID i regulatorna praksa

Worldcoin i slični projekti (npr. World ID) koriste biometriju za dokaz jedinstvenosti, što je dovelo do brojnih regulatornih intervencija (BayLDA u Njemačkoj, CNIL u Francuskoj, PCPD u Hong Kongu). Iako dio arhitekture koristi ZK‑dokaze, nadzorna tijela su fokusirana na izvornu zbirku biometrijskih podataka i transparentnost, a ne na sam ZK sloj.[^64][^65][^66]
Nijedno od tih tijela ne dovodi u pitanje ZK‑tehnologiju kao takvu, ali jasno ističu da, **ako voditelj obrađuje biometriju ili drugi snažan identifikator**, sama uporaba ZK‑tehnike ga ne izuzima iz GDPR‑a; presudno je koji se osobni podaci prikupljaju i kako se njima upravlja.
[^65][^66]


### 6.4. Je li ZK‑nullifier pristup povoljniji?

U usporedbi s pohranom `keccak256(hash(OIB))` na javnom lancu, ZK‑nullifier pristup ima nekoliko jasnih prednosti:

- Omogućuje **dokaz jedinstvenosti bez izlaganja identifikatora** na javnom lancu.
- Lakše se usklađuje s načelom smanjenja količine podataka i ograničenja pohrane (čl. 5 GDPR‑a), jer on‑chain ostaju samo minimalni, teško povezivi podaci.[^34][^3]
- Pruža bolju fleksibilnost za ostvarivanje prava na brisanje i ograničenje obrade, jer se funkcionalna veza može prekinuti deaktivacijom verifikacijskog mehanizma ili uništenjem tajnog identiteta, bez potrebe za „brisanje“ podataka iz blokova.

Stoga se ZK‑nullifier može smatrati **bitno povoljnijim** s aspekta GDPR‑a, pod uvjetom da je cijela arhitektura (uključujući onboarding i eventualni KYC) dizajnirana tako da voditelj obrade ne pohranjuje izravne identifikatore na javnom lancu i da su svi pomoćni sustavi usklađeni.
[^35][^59]


## 7. Hrvatski kontekst

### 7.1. Zakon o provedbi Opće uredbe i nadležnost AZOP‑a

Zakon o provedbi Opće uredbe o zaštiti podataka (NN 42/18) implementira i operacionalizira GDPR u hrvatskom pravu, ali **ne uvodi posebna pravila za blockchain**; ključne definicije i načela preuzimaju se izravno iz GDPR‑a.[^67][^68][^1]
AZOP je nadležno tijelo za nadzor primjene GDPR‑a u Hrvatskoj i u svojim materijalima (česta pitanja, mišljenja, obrasci) slijedi iste principe identifikabilnosti, pseudonimizacije i pravnih osnova kao i EDPB.[^69][^29][^27]


### 7.2. AZOP i blockchain

Godišnja izvješća AZOP‑a navode da je izrađena i usvojena „Blockchain smjernica“ u okviru europskih radnih skupina, no sam tekst smjernice nije lako dostupan; iz konteksta izvješća i međunarodnih smjernica (EDPB, CNIL) razvidno je da AZOP dijeli opću liniju: **izbjegavati pohranu osobnih podataka on‑chain te koristiti off‑chain i pseudonimizacijske tehnike**.[^70][^71][^59]
Domaća stručna literatura (npr. pravni pregledi blockchaina i GDPR‑a na hrvatskom) također naglašava da su pseudonimizirani podaci i dalje osobni podaci, i da hashing **ne rješava** problem primjenjivosti GDPR‑a pri korištenju blockchaina.[^72][^59]


### 7.3. Hrvatski projekti i hashirani identifikatori

Javni izvori ne navode detaljne primjere hrvatskih projekata koji trajno pohranjuju hashirane OIB‑ove na javne lance; postoje projekti u financijskom sektoru i istraživanjima koji koriste blockchain, ali uglavnom u permissioned okruženjima ili s off‑chain pohranom osjetljivih podataka.[^73][^72]
S obzirom na visoku regulatornu osjetljivost na curenje OIB‑a i drugih identifikatora (primjeri kazni B2 Kapitalu i EOS Matrixu zbog povreda osobnih podataka), razumno je očekivati da bi AZOP zauzeo konzervativan stav i prema hashiranim OIB‑ovima na javnim lancima.
[^74][^75]


## 8. Sažetak relevantnih presuda i smjernica

### 8.1. Tablica ključnih izvora

| Instrument / dokument | Datum | Ključni zaključak |
|-----------------------|-------|-------------------|
| CJEU, *Breyer* (C‑582/14) | 2016 | Dinamička IP adresa može biti osobni podatak ako voditelj obrade, uz pomoć treće strane, razumno može identificirati osobu; potvrđen relativni test identifikabilnosti. [^7][^9] |
| GDPR, čl. 4 i Recital 26 | 2016/2018 | Definira osobni podatak i pseudonimizaciju; pseudonimizirani podaci su i dalje osobni podaci ako se mogu pripisati osobi dodatnim informacijama; anonimizacija traži da ispitanik više nije identificiran, uzimajući u obzir sva razumno vjerojatna sredstva identifikacije. [^2][^6] |
| WP29 Opinion 05/2014 (WP216) | 2014 | Hashiranje izravnih identifikatora u pravilu je pseudonimizacija, ne anonimizacija, zbog mogućih napada i spajanja skupova podataka; anonimizacija je kontekstualan i strogo definiran cilj. [^10][^11] |
| AEPD/EDPS „Introduction to the hash function as a personal data pseudonymisation technique“ | 2019 | Hash je prije svega tehnika pseudonimizacije; potreban je objektivan rizik‑assessment ponovne identifikacije, uz osobitu pažnju na entropiju poruke, dostupnost rječnika i pomoćnih podataka; salting/ključ mogu smanjiti rizik, ali ne garantiraju anonimizaciju. [^18][^17] |
| EDPB Guidelines 01/2025 on Pseudonymisation | 2025 | Pseudonimizacija je ključna zaštitna mjera, ali pseudonimizirani podaci ostaju osobni za subjekte koji drže dodatne informacije; ista obrada može biti osobni podatak za jednog sudionika, a anonimiziran za drugog bez pristupa ključu. [^14][^15][^16] |
| EDPB Guidelines 02/2025 on processing of personal data through blockchain technologies | 2025 | Blockchain ne dobiva izuzeće od GDPR‑a; preporučuje se izbjegavanje pohrane osobnih podataka on‑chain; hash off‑chain podataka na lancu je osobni podatak za subjekte s pristupom izvoru/ključeva; DPIA je snažno preporučena prije blockchain projekata s osobnim podacima. [^38][^3][^37] |
| CNIL „Blockchain and the GDPR“ | 2018 | Tehnički je nemoguće potpuno obrisati podatke upisane u blockchain; preporučuje se off‑chain pohrana osobnih podataka i on‑chain hash/commitment; brisanje se može funkcionalno približiti uništenjem ključa i blokiranjem pristupa. [^24][^25][^26] |
| ICO Guidance on anonymisation & pseudonymisation | 2022–2025 | Hashiranje je tipična tehnika pseudonimizacije; rezultati i dalje predstavljaju osobne podatke za subjekte koji ih mogu povezati s osobama; primjenjuje se test „razumno vjerojatne“ re‑identifikacije i „motiviranog uljeza“. [^20][^21][^22][^23] |
| GDPR, čl. 17 (pravo na brisanje) | 2016/2018 | Propisuje okolnosti u kojima ispitanik može tražiti brisanje, a voditelj mora bez odgode izbrisati osobne podate; dodatno ograničeno izuzecima (pravne obveze, javni interes, pravo na slobodu izražavanja). [^31][^32] |
| GDPR, čl. 35 (DPIA) i prateće smjernice | 2016/2018 | DPIA je obvezna za obrade koje vjerojatno rezultiraju visokim rizikom, osobito uz nove tehnologije; blockchain projekti s osobnim podacima tipično spadaju u ovu kategoriju. [^40][^41][^43] |
| Zakon o provedbi Opće uredbe o zaštiti podataka (NN 42/18) | 2018 | Nacionalni zakon kojim se provodi GDPR u RH; ne uvodi posebna pravila za blockchain, već potvrđuje primjenu općih načela i definicija GDPR‑a. [^1][^67][^68] |
| AZOP materijali (česta pitanja, privola, prava ispitanika) | 2018–2025 | Potvrđuju da je OIB osobni podatak; naglašavaju opozivost privole i potrebu jasnog pravnog temelja i načela smanjenja količine podataka za svaku obradu, uključujući inovativne tehnologije. [^27][^28][^55] |


## 9. Preporuke za dizajn vašeg projekta

Na temelju navedenog pravnog i regulatornog okvira, za opisani sustav (Sybil zaštita na Gnosis lancu uz hashirani OIB) proizlaze sljedeće praktične preporuke:

### 9.1. Polazne pretpostavke

1. **Tretirajte `keccak256(hash(OIB))` kao osobni podatak (pseudonimizirani)** za potrebe GDPR‑a i hrvatskog prava; sukladno tome, projekt je obrada osobnih podataka, s odgovarajućim obvezama voditelja obrade.
2. **Polazite od toga da je DPIA obvezna**, a ne samo preporučena, s obzirom na novu tehnologiju, opseg i neizbrisivu prirodu zapisa.
3. **Javni, permissionless blockchain** je s aspekta GDPR‑a najproblematičnija arhitektura; treba obrazložiti zašto je nužna i razmotriti mogućnosti permissioned ili hibridnog rješenja gdje je to izvedivo.


### 9.2. Tehničke preporuke

1. **Razmotrite ZK‑nullifier arhitekturu** (npr. Semaphore‑like dizajn) umjesto direktne pohrane hashiranog identifikatora na lancu:
   - On‑chain čuvajte samo kriptografske dokaze i nullifiere koji nisu izvedeni iz OIB‑a u trivijalnoj formi.
   - Off‑chain ili kod treće strane vodite eventualni KYC (OIB) u klasičnim, revokabilnim sustavima s mogućnošću brisanja.

2. Ako se hash ipak pohranjuje na lancu:
   - Koristite **tajnu sol** ili ključ **koji nije poznat voditelju obrade** (npr. user‑held salt) kako biste drastično smanjili rizik rječničkih napada.
   - Projektirajte sustav tako da se **funkcionalna ovisnost o tom hashu može prekinuti** (npr. pametni ugovor prestaje prihvaćati dokaze vezane uz određeni hash nakon povlačenja privole ili raskida ugovora).
   - Dokumentirajte uvjete pod kojima uništenje soli/ključa dovodi do toga da preostali hash više ne bude osobni podatak, uključujući detaljan rizik‑assessment i, po mogućnosti, prethodnu konzultaciju s AZOP‑om.

3. **Minimizirajte druge on‑chain identifikatore**:
   - Izbjegavajte izravnu pohranu adresa koje se mogu povezati s realnim identitetom (npr. reuse adresa, povezivanje s KYC burzama) gdje je to moguće.
   - Razmislite o korištenju session‑adresa, relayera ili drugih privacy‑enhancing tehnika kako biste otežali spajanje adresa s osobama.

4. **Off‑chain skladište kao primarni izvor istine**:
   - Čuvajte OIB i druge osobne podatke isključivo u off‑chain sustavima pod vašom kontrolom, s klasičnim mehanizmima brisanja, ograničenja pohrane i pristupa.
   - Na lancu čuvajte samo minimalne, dobro analizirane kriptografske artefakte (hash, commitment, ZK‑dokaz) potrebne za funkcionalnost.


### 9.3. Organizacijske i pravne preporuke

1. **Provedite formalni DPIA** prije produkcije:
   - Uključite tehničke i pravne stručnjake za blockchain i zaštitu podataka.
   - Jasno dokumentirajte zašto je blockchain nužan (npr. potreba za globalno provjerljivim Sybil‑otporom) i zašto drugi, manje intruzivni pristupi nisu dovoljni.

2. **Definirajte voditelja obrade i izvršitelje**:
   - Utvrdite tko je voditelj obrade (najčešće pravna osoba koja upravlja platformom) i tko su izvršitelji (npr. pružatelji node‑infrastrukture, KYC pružatelji).
   - Ako je arhitektura DAO‑like i decentralizirana, razmotrite koncept zajedničkih voditelja obrade i ugovorne aranžmane u skladu s člankom 26 GDPR‑a.

3. **Odabir pravne osnove**:
   - Preferirajte kombinaciju **izvršenja ugovora** (za osnovnu obradu nužnu za pružanje usluge) i/ili **legitimnog interesa** (sprječavanje prijevara), uz jasan test razmjernosti.
   - Privolu koristite samo za dodatne funkcionalnosti, ne kao jedini temelj za neizbrisive on‑chain zapise.
   - Jasno informirajte korisnike o **nepromjenjivoj prirodi** on‑chain zapisa i ograničenjima prava na brisanje.

4. **Prava ispitanika i procedura za brisanje/ograničenje**:
   - U politiku privatnosti i korisničke uvjete uključite detaljan opis kako se ostvaruju prava na pristup, ispravak, brisanje i ograničenje obrade u blockchain kontekstu (npr. funkcionalno blokiranje identiteta, uništenje soli, prestanak prihvaćanja dokaza).
   - Uspostavite procese za reagiranje na zahtjeve ispitanika u zakonskim rokovima, uključujući komunikaciju o tehničkim ograničenjima i mjerama kompenzacije (npr. deaktivacija računa, prestanak on‑chain korištenja identiteta).

5. **Sigurnost i governance**:
   - Osigurajte jake tehničke mjere (enkripcija, kontrola pristupa, logging, revizije) za sve off‑chain sustave koji obrađuju OIB i povezane podatke.
   - Dokumentirajte politike rotacije i uništenja ključeva/soli, upravljanje incidentima i obveznu prijavu povreda osobnih podataka AZOP‑u i ispitanicima gdje je to primjenjivo.

6. **Komunikacija s AZOP‑om**:
   - S obzirom na inovativnost i visoki rizik, razmotrite mogućnost **prethodnog savjetovanja** s AZOP‑om nakon izrade DPIA‑e, osobito ako DPIA pokaže da i nakon mjera ostaje visoki rezidualni rizik.


## 10. Zaključna pravna ocjena

Na temelju važećih presuda CJEU‑a, smjernica EDPB‑a, CNIL‑a, ICO‑a, AEPD/EDPS‑a i hrvatskih propisa, hashiran OIB pohranjen na javnom blockchainu – u obliku `keccak256(hash(OIB))` bez adekvatne, tajne soli poznate isključivo korisniku – **jest osobni podatak u smislu GDPR‑a**, jer se s razumno vjerojatnim sredstvima može povezati s identificiranom ili identificirljivom osobom.
[^2][^18][^10][^3]
Takav hash treba tretirati kao **pseudonimizirani osobni podatak**, a ne anonimizirani; stoga su na njega u potpunosti primjenjive obveze iz GDPR‑a (pravna osnova, DPIA, prava ispitanika, ograničenje pohrane, sigurnost obrade).
[^15][^10][^6]
S obzirom na inherentnu nekompatibilnost javnih blockchaina s pravom na brisanje, regulatorno najrobustniji pristup je **dizajn koji uopće ne zahtijeva pohranu pseudonimnih identifikatora na javnom lancu**, već koristi ZK‑nullifier ili slične mehanizme uz off‑chain obradu identiteta u kontroliranim, revokabilnim sustavima.

---

## References

1. [Zakon o provedbi Opće uredbe o zaštiti podataka - Narodne novine](https://narodne-novine.nn.hr/clanci/sluzbeni/2018_05_42_805.html) - (1) Ovim Zakonom osigurava se provedba Uredbe (EU) 2016/679 Europskog parlamenta i Vijeća od 27. tra...

2. [EU General Data Protection Regulation (EU GDPR) Article 4 - IITR](https://www.iitr.us/gdpr/4) - EU General Data Protection Regulation (EU GDPR) Article 4. Definitions. For the purposes of this Reg...

3. [[PDF] Guidelines 02/2025 on processing of personal data through ...](https://www.edpb.europa.eu/system/files/2025-04/edpb_guidelines_202502_blockchain_en.pdf) - The distributed nature of blockchain and the complex mathematical concepts involved imply a high deg...

4. [Art. 4 GDPR - Definitions](https://gdpr.eu/article-4-definitions/) - Art. 4 GDPRDefinitions For the purposes of this Regulation: ‘personal data’ means any information re...

5. [Art. 4 GDPR – Definitions - General Data Protection ...](https://gdpr-info.eu/art-4-gdpr/) - For the purposes of this Regulation: ‘personal data’ means any information relating to an identified...

6. [Recital 26 - Not Applicable to Anonymous Data - GDPR](https://gdpr-info.eu/recitals/no-26/) - The principles of data protection should therefore not apply to anonymous information, namely inform...

7. [CJEU decision on dynamic IP addresses touches fundamental DP ...](https://www.twobirds.com/en/insights/2016/global/cjeu-decision-on-dynamic-ip-addresses-touches-fundamental-dp-law-questions) - The plaintiff Mr. Breyer asserted that dynamic IP addresses qualify as personal data and are therefo...

8. [Clouds outside of the scope of GDPR? (identifiability test) - EFDPO](https://www.efdpo.eu/clouds-outside-of-the-scope-of-gdpr-identifiability-test/) - In 2016 Court of Justice of the European Union (“CJEU”) issued a landmark ruling in Breyer case (C-5...

9. [In Breyer decision today, Europe's highest court rules on definition of ...](https://iapp.org/news/a/in-breyer-decision-today-europes-highest-court-rules-on-definition-of-personal-data) - The CJEU was not considering pseudonymization directly, but rather the definition of personal data a...

10. [[PDF] ARTICLE 29 DATA PROTECTION WORKING PARTY](https://ec.europa.eu/justice/article-29/documentation/opinion-recommendation/files/2014/wp216_en.pdf) - In this Opinion, the WP analyses the effectiveness and limits of existing anonymisation techniques a...

11. [The EU Article 29 Working Party's Opinion on Privacy and Anonymity](https://news.bloomberglaw.com/privacy-and-data-security/the-eu-article-29-working-partys-opinion-on-privacy-and-anonymity-its-harder-than-you-think) - On April 10, 2014, the European Union's Article 29 Data Protection Working Party adopted “Opinion 05...

12. [[PDF] GDPR: Anonymisation and pseudonymisation - PwC Luxembourg](https://www.pwc.lu/en/general-data-protection/docs/pwc-anonymisation-and-pseudonymisation.pdf) - The Article 29. Working Party notes, in Opinion 05/2014, that resulting data from an attempt at anon...

13. [Data Pseudonymisation vs Anonymisation: Key Differences](https://gdprlocal.com/data-pseudonymisation-vs-anonymisation/) - Anonymisation under GDPR is the process of permanently and irreversibly modifying personal data so t...

14. [EDPB adopts pseudonymisation guidelines and paves the way to ...](https://www.edpb.europa.eu/news/news/2025/edpb-adopts-pseudonymisation-guidelines-and-paves-way-improve-cooperation_en) - In its guidelines, the EDPB clarifies the definition and applicability of pseudonymisation and pseud...

15. [The EDPB 01/2025 Guidelines on Pseudonymisation](https://www.europeanlawblog.eu/pub/tfef074h) - The EDPB Guidelines include in its annex several detailed examples of the application of pseudonymis...

16. [[PDF] Guidelines 01/2025 on Pseudonymisation](https://www.edpb.europa.eu/system/files/2025-01/edpb_guidelines_202501_pseudonymisation_en.pdf) - Thus, pseudonymisation is a safeguard that can be applied by controllers to meet the requirements of...

17. [INTRODUCTION TO THE HASH FUNCTION AS A ...](https://collab.dpa.gr/wp-content/uploads/2023/07/aepd-edps_INTRODUCTION-TO-THE-HASH-FUNCTION-AS-A-PERSONAL-DATA-PSEUDONYMISATION-TECHNIQUE_en.pdf)

18. [Introduction to the hash function as a personal data ...](https://www.edps.europa.eu/data-protection/our-work/publications/papers/introduction-hash-function-personal-data_en) - Joint paper of the Spanish data protection authority, Agencia española de protección de datos (AEPD)...

19. [Spanish Supervisory Authority and EDPS Release Guidance on ...](https://www.insideprivacy.com/data-privacy/spanish-supervisory-authority-and-edps-release-guidance-on-hashing-for-data-pseudonymization-and-anonymization-purposes/) - On November 4, 2019, the Spanish Supervisory Authority (“AEPD”), in collaboration with the European ...

20. [Information Commissioner's Office adopted guidance on ...](https://digitalpolicyalert.org/event/28718-information-commissioners-office-adopted-guidance-on-anonymisation-techniques-for-data-protection) - The guidance addresses the use of anonymisation and pseudonymisation by organisations processing per...

21. [ICO Anonymisation Guidance: UK Data Protection Compliance ...](https://www.studocu.com/en-gb/document/queens-university-belfast/law-of-evidence/ico-anonymisation-guidance-uk-data-protection-compliance-insights/127913330) - Below, we explore some of the central concepts in the ICO's new guidance and the tests the authority...

22. [Anonymisation and Pseudonymisation of Personal Data](https://www.ucl.ac.uk/data-protection/guidance-staff-students-and-researchers/practical-data-protection-guidance-notices/anonymisation-and) - This guidance outlines the key differences between anonymisation and pseudonymisation, how they appl...

23. [Pseudonymisation | ICO](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-sharing/anonymisation/pseudonymisation/?search=mapping+the+concept+of+the+spectrum) - At a glance. Pseudonymisation refers to techniques that replace, remove or transform information tha...

24. [CNIL Publishes Initial Assessment on Blockchain and GDPR](https://www.hunton.com/privacy-and-information-security-law/cnil-publishes-initial-assessment-blockchain-gdpr)

25. [Solutions for a responsible use of the blockchain ...](https://www.cnil.fr/sites/default/files/atoms/files/blockchain_en.pdf)

26. [Blockchain and the GDPR: Solutions for a responsible use of ... - CNIL](https://www.cnil.fr/en/blockchain-and-gdpr-solutions-responsible-use-blockchain-context-personal-data) - What is a blockchain? A blockchain is a database in which data is stored and distributed to a large ...

27. [Često postavljana pitanja - Agencija za zaštitu osobnih podataka](https://azop.hr/najcesce-postavljena-pitanja/) - Na koje načine moji osobni podaci mogu biti zloupotrebljeni? Zlouporabu tuđih osobnih podataka netko...

28. [Opća uredba o zaštiti podataka (General Data Protection Regulation ...](https://azop.hr/opca-uredba-gdpr/) - Osobni podaci su primjerice: ime i prezime, adresa fizičke osobe, e-mail adresa, podaci o zdravlju, ...

29. [Dostava podatka o bruto plaći zaposlenika ministarstva ... - AZOP](https://azop.hr/dostava-podatka-o-bruto-placi-zaposlenika-ministarstva-sukladno-pravu-na-pristup-informacijama/) - Objavljeno: 12.1.2022. Agencija za zaštitu osobnih podataka zaprimila je Vaš upit u kojem navodite d...

30. [European Data Protection Board Issues Blockchain GDPR Guidelines](https://fullycrypto.com/european-data-protection-board-issues-blockchain-gdpr-guidelines) - The European Data Protection Board has released new guidelines clarifying how blockchain technology ...

31. [The Right to be Forgotten and the Immutability of Blockchain ...](https://goodlawsoftware.co.uk/law/the-right-to-be-forgotten-and-the-immutability-of-blockchain-technology/) - The Right to be Forgotten and the Immutability of Blockchain Technology To understand why it is diff...

32. [When Blockchain Meets the Right to be Forgotten - Secure Privacy](https://secureprivacy.ai/blog/blockchain-immutability-vs-gdpr-article-17-right-to-be-forgotten) - Organizations worldwide are grappling with an unprecedented regulatory puzzle: how to harness blockc...

33. [[PDF] Reconciling the conflict between the 'immutability' of public and ...](https://www.utupub.fi/bitstream/handle/10024/146293/Jussila_Jani-Pekka_opinnayte.pdf?sequence=1&isAllowed=y)

34. [Reconciling blockchain technology and data protection laws](https://academic.oup.com/cybersecurity/article/11/1/tyaf002/8024082) - Abstract. This paper thoroughly explores the complex interplay between blockchain technology and the...

35. [How GDPR Affects Blockchain Technology](https://gdprlocal.com/gdpr-blockchain/) - Blockchain addresses, public keys, and transaction data often qualify as personal data under GDPR, e...

36. [EDPB Issues Guidelines for Blockchain Compliance with GDPR](https://nquiringminds.com/ai-legal-news/edpb-issues-guidelines-for-blockchain-compliance-with-gdpr/) - These guidelines are designed to ensure that personal data processing through blockchain technologie...

37. [Guidelines of the European Data Protection Board on processing of ...](https://fintech.gov.pl/component/content/article/guidelines-of-the-european-data-protection-board-on-processing-of-personal-data-through-blockchain-technologies?catid=19&Itemid=491) - On 8 April 2025, the European Data Protection Board adopted Guidelines 02/2025 on processing of pers...

38. [EDPB adopts guidelines on processing personal data through ...](https://www.edpb.europa.eu/news/news/2025/edpb-adopts-guidelines-processing-personal-data-through-blockchains-and-ready_en) - In its guidelines, the EDPB explains how blockchains work, assessing the different possible architec...

39. [Pseudomization and Re-identification - MyEDPO](https://www.myedpo.com/post/pseudomization-and-re-identification) - The applicant SRB accepted the case law that there are two conditions to be assessed concerning the ...

40. [When does an organization need to conduct DPIA in GDPR?](https://vistainfosec.com/blog/when-does-an-organization-need-to-conduct-dpia-in-gdpr/) - Unsure when to conduct a DPIA under GDPR? Learn the key triggers and protect your organization. Read...

41. [When do we need to do a DPIA? | ICO](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/accountability-and-governance/data-protection-impact-assessments-dpias/when-do-we-need-to-do-a-dpia/)

42. [Data Protection Impact Assessment (DPIA) - GDPR.eu](https://gdpr.eu/data-protection-impact-assessment-template/) - How to conduct a Data Protection Impact Assessment (template included) A Data Protection Impact Asse...

43. [GDPR DPIA Guide (Article 35) - Glocert International](https://www.glocertinternational.com/resources/guides/gdpr-dpia-guide-article-35/) - When DPIA is required and how to document it for GDPR compliance.

44. [Your EU GDPR Article 35: Data Protection Impact Assessment (DPIA ...](https://trustarc.com/resource/data-protection-impact-assessment-article35/) - The GDPR compliance deadline has passed, and if you don't have a documented process for conducting D...

45. [What is considered personal data under the EU GDPR?](https://gdpr.eu/eu-gdpr-personal-data/) - The EU’s GDPR only applies to personal data, which is any piece of information that relates to an id...

46. [GDPR Article 6: What are the 7 Legal Bases for Data Processing?](https://www.exabeam.com/explainers/gdpr-compliance/gdpr-article-6-what-are-the-7-legal-bases-for-data-processing/) - The GDPR is the EU’s primary data protection framework. Article 6 details lawful bases for processin...

47. [Refresher: The GDPR's Six Legal Bases for Data Processing - IAPP](https://iapp.org/resources/article/refresher-the-gdprs-six-legal-bases-for-data-processing) - This resource provides a refresher on the six bases for lawful processing under Article 6 of the EU ...

48. [GDPR Legitimate Interest: Article 6(1)(f) Overview](https://gdprlocal.com/gdpr-legitimate-interest/) - Legitimate interest is a flexible lawful basis under GDPR that allows processing personal data witho...

49. [What is the 'legitimate interests' basis? | ICO](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/lawful-basis/legitimate-interests/what-is-the-legitimate-interests-basis/)

50. [Sigurnosni i pravni aspekti obrade osobnih podataka u testnim ...](https://azop.hr/sigurnosni-i-pravni-aspekti-obrade-osobnih-podataka-u-testnim-okruzenjima-ikt-sustava/) - Opće uredbe o zaštiti podataka definirana su načela obrade osobnih podataka, odnosno da osobni podac...

51. [Opća uredba o zaštiti osobnih podataka](https://www.pakostane.hr/hr/info/gdpr)

52. [Često postavljana pitanja: obrada osobnih podataka u stambenim ...](https://azop.hr/preporuka-cesto-postavljana-pitanja-obrada-osobnih-podataka-u-stambenim-zgradama/) - Je li dopušteno u stambenim zgradama uspostaviti sustav videonadzora? U stambenim zgradama je dopušt...

53. [PRIVOLA ZA OBRADU OSOBNIH PODATAKA](https://azop.hr/wp-content/uploads/2021/01/Obrazac-privole_web_azop.pdf)

54. [Privola ispitanika | Zaštita osobnih podataka](https://gdpr-2018.hr/33/privola-ispitanika-uniqueidmRRWSbk196E4DjKFq6pChCIlnaxBaAPQGQq0vvv-Zl3xT00Xhq8dwg/?forcedesktop=off&uniqueid=mRRWSbk196E4DjKFq6pChCIlnaxBaAPQGQq0vvv-Zl3xT00Xhq8dwg&coolurl=1&query=Ugovor&version_year=2025&section=33) - 3.3 Privola ispitanika Kristian Plazoni�, dipl.iur., Igor Radeli�, dipl. iur. Zahtijevanje privole u...

55. [Mišljenje AZOP-a odnosno na privole ispitanika (GDPR)](https://gdpr-2018.hr/33/misljenje-azop-a-odnosno-na-privole-ispitanika-gdpr-uniqueidRCViWTptZHJrD63HGNzcj499sA-zs-VpjAfKtmfKBkk/) - Od Agencije za za�titu osobnih podataka zatra�eno je mi�ljenje treba li voditelj obrade prikupljati ...

56. [How to Demonstrate Compliance With GDPR Article 6](https://www.isms.online/general-data-protection-regulation-gdpr/gdpr-article-6-compliance/) - GDPR Article 6 outlines the basic lawful principles that prohibits all processing of personal data u...

57. [EDPB: Preliminary version of Guidelines 3/2019 on video surveillance](https://www.delucapartners.it/en/insights/edpb-preliminary-version-of-guidelines-3-2019-on-video-surveillance/) - Another important clarification concerns filming based on legitimate interest. Data processing is co...

58. [[PDF] Video devices & data protection: When to act and what to do](https://www.edpb.europa.eu/system/files/2026-04/summary_edpb_guidelines_201903_video_devices_en.pdf) - The EDPB Guidelines on processing of personal data through · video devices provide guidance covering...

59. [Blockchain i GDPR - Parser compliance](https://parser.hr/blockchain-gdpr/) - ... Agencija za zaštitu osobnih podataka). Tehnička i integrirana ... blockchain-a te konačno rješen...

60. [FAQ | Semaphore](https://docs.semaphore.pse.dev/V3/faq) - What is Semaphore?

61. [Introduction | Semaphore Documentation](https://akinovak.github.io/semaphore-spec/) - Overview

62. [What Is Semaphore?](https://docs.semaphore.pse.dev/V1) - Overview

63. [What you need to know before anonymizing personal data](https://erislaw.se/artiklar/what-you-need-to-know-before-anonymizing-personal-data/) - To anonymize personal data, you must conduct an impact assessment to identify, prevent, and protect ...

64. [German Regulator Orders Worldcoin to Delete Biometric Data ...](https://idtechwire.com/german-regulator-orders-worldcoin-to-delete-biometric-data-over-gdpr-violations/) - [Editor’s note 12/23/24: This article has been updated to include a response from Chief Legal and Pr...

65. [Worldcoin collecting biometric data - European Parliament](https://www.europarl.europa.eu/RegData/questions/reponses_qe/2024/000961/P9_RE(2024)000961_EN.pdf)

66. [Privacy Commissioner's Office Finds that the Operation of ... - PCPD](https://www.pcpd.org.hk/english/news_events/media_statements/press_20240522.html) - Office of the Privacy Commissioner for Personal Data, Date: 22 May 2024 Privacy Commissioner’s Offic...

67. [Zakon o provedbi Opće uredbe o zaštiti podataka](https://www.zakon.hr/z/1023/zakon-o-provedbi-opce-uredbe-o-zastiti-podataka) - (1) Ovim Zakonom osigurava se provedba Uredbe (EU) 2016/679 Europskog parlamenta i Vijeća od 27. tra...

68. [Zakon o provedbi Opće uredbe o zaštiti podataka - RRiF](https://www.rrif.hr/zakon_o_provedbi_opce_uredbe_o_zastiti_podataka-4230-propis/) - Zakonom se osigurava provedba Uredbe (EU) 2016/679. Europskog parlamenta i Vijeća od 27. travnja 201...

69. [Zakon o provedbi Opće uredbe o zaštiti podataka - AZOP](https://azop.hr/zakon-o-provedbi-opce-uredbe-o-zastiti-podataka/) - Poštovani/e,. obavještavamo Vas da je Zakon o provedbi Opće uredbe o zaštiti podataka koji je Hrvats...

70. [[PDF] Godišnje izvješće o radu 2022. - AZOP](https://azop.hr/wp-content/uploads/2023/12/AZOP_Izvjesce-2022.pdf) - Smjernica o Blockchain-u započeta u 2021. i dovršena u 2022. godine, čija svrha je opisati različite...

71. [[PDF] Godišnje izvješće o radu za 2024. godinu - AZOP](https://azop.hr/wp-content/uploads/2025/09/Godisnje-izvjesce-2024.pdf) - Tijekom godine dovršene su Blockchain smjernice i jednoglasno prihvaćene. Podskupina je kroz nekolik...

72. [Zaštita osobnih podataka u svijetu digitalnih financija - HNB](https://www.hnb.hr/-/zastita-osobnih-podataka-u-svijetu-digitalnih-financija) - U panel-raspravi sudjelovali su i Vlaho Hrdalo (odvjetnik i predsjednik Udruge za blockchain i kript...

73. [[PDF] RAZVOJ POSLOVNOG BANKARSTVA U DIGITALNOM OKRUŽENJU](https://repozitorij.efzg.unizg.hr/theses/efzg:15942/show-file/0) - Blockchain je digitalna, decentralizirana, neizmjenjiva i distribuirana knjiga koja bilježi transakc...

74. [Spremaju se tužbe nakon rješenja AZOP-a koje je išlo u prilog ...](https://www.index.hr/vijesti/clanak/spremaju-se-tuzbe-nakon-rjesenja-azopa-koje-je-islo-u-prilog-bankama/2553461.aspx) - No, postavljalo se pitanje odakle utjerivačima dugova osobni podaci koji nisu nužno vezani uz naplat...

75. [B2 Kapital se oglasio oko curenja osobnih podataka - Jutarnji list](https://www.jutarnji.hr/vijesti/hrvatska/b2-kapital-se-oglasio-oko-curenja-osobnih-podataka-ne-mozemo-iskljuciti-da-se-radi-o-hakerima-15286821) - Državna Agencija za zaštitu osobnih podataka (AZOP) potvrdila je da provodi istragu o mogućem curenj...

