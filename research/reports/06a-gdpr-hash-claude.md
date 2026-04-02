# GDPR i hashirani identifikatori — Claude Opus 4.6 izvještaj

**Prompt:** [06-gdpr-hash-identifikatori.md](../prompts/06-gdpr-hash-identifikatori.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# GDPR analiza hashiranih osobnih identifikatora na javnom blockchainu

**Keccak256 hash OIB-a pohranjen na javnom blockchainu (Gnosis Chain) predstavlja osobni podatak u smislu GDPR-a.** To je nedvosmislen zaključak koji proizlazi iz kumulativnog tumačenja članka 4. stavka 1. GDPR-a, presuda CJEU-a u predmetima Breyer (C-582/14) i SRB (C-413/23 P), Smjernica EDPB-a 01/2025 o pseudonimizaciji i — najvažnije — Smjernica EDPB-a 02/2025 o obradi osobnih podataka putem blockchain tehnologija iz travnja 2025., koje izričito navode da **hash pohranjen na blockchainu predstavlja osobni podatak**. Projektni tim mora ozbiljno razmotriti alternativne pristupe, posebice ZK nullifier mehanizam, kako bi se izbjegao fundamentalni sukob između nepromjenjivosti blockchaina i prava na brisanje iz članka 17. GDPR-a.

---

## 1. Pravni zaključak — hash OIB-a jest osobni podatak

### Definicija osobnog podatka i test identifikacije

Članak 4. stavak 1. GDPR-a definira osobni podatak kao „svaku informaciju koja se odnosi na identificiranu ili identificirabilnu fizičku osobu". Uvodna izjava (recital) 26 pojašnjava da se pri ocjeni identifikabilnosti trebaju uzeti u obzir **sva sredstva koja je razumno vjerojatno da će biti korištena** — od strane voditelja obrade ili **bilo koje druge osobe** — za izravno ili neizravno identificiranje pojedinca. Procjena razumnosti temelji se na objektivnim čimbenicima: troškovima, vremenu, dostupnoj tehnologiji i tehnološkom razvoju.

OIB je 11-znamenkasti broj s prostorom od otprilike **10^11 mogućih vrijednosti** (u praksi manje, s obzirom na kontrolnu znamenku). Izračun keccak256 hasheva svih mogućih OIB-ova na modernom hardveru trajao bi **od nekoliko minuta do nekoliko sati** — daleko manje od napora koji je CJEU u predmetu Breyer smatrao dovoljnim za kvalificiranje podatka kao osobnog. Tzv. rainbow table napad je trivijalan.

### Stav EDPB-a 02/2025 — izričita potvrda

Smjernice EDPB-a 02/2025 o obradi osobnih podataka putem blockchain tehnologija (usvojene 8. travnja 2025., javna konzultacija do 9. lipnja 2025.) predstavljaju **najautoritativniji i najnoviji izvor** za ovu analizu. EDPB izričito navodi: „GDPR će se i dalje primjenjivati na tu aktivnost obrade te **će se hash također smatrati osobnim podatkom**, kao i svi drugi identifikatori koji mogu postojati." Nadalje, EDPB upozorava da **nesoljeni ili nekeyirani hashevi u pravilu ne bi trebali biti smatrani dostatnima** za osiguranje potrebne razine zaštite povjerljivosti pri pohranjivanju osobnih podataka na javnom blockchainu.

### WP216 — hashing nije anonimizacija

Mišljenje Radne skupine iz članka 29. (WP216, Opinion 05/2014 o tehnikama anonimizacije) eksplicitno navodi: **„Pseudonimizacija nije metoda anonimizacije."** Hashing je klasificiran kao tehnika pseudonimizacije koja „preslikava ulaz bilo koje veličine u izlaz fiksne veličine". WP29 izričito upozorava na ranjivost hash funkcija kad je prostor ulaznih vrijednosti poznat i ograničen — točno slučaj s OIB-om. Niti jedna tehnika pseudonimizacije (hashing, enkripcija, tokenizacija) ne zadovoljava sva tri kriterija rizika: **izdvajanje (singling out), povezivanje (linkability) i zaključivanje (inference)**.

### Zaključak o klasifikaciji

Keccak256(OIB) na javnom blockchainu ispunjava sve uvjete za kvalifikaciju osobnog podatka prema svakom od mjerodavnih izvora: GDPR tekstu, praksi CJEU-a, smjernicama EDPB-a, mišljenju WP29 i smjernicama ICO-a. **Ne postoji pravno utemeljeni argument da je takav hash anoniman podatak.**

---

## 2. Relevantne presude i smjernice

| Izvor | Datum | Ključni zaključak |
|-------|-------|-------------------|
| **GDPR čl. 4(1), Uvodna izjava 26** | 25.5.2016. | Osobni podatak je svaka informacija o identificiranoj ili identificirabilnoj osobi; pseudonimizirani podaci su i dalje osobni podaci |
| **CJEU Breyer (C-582/14)** | 19.10.2016. | Dinamička IP adresa je osobni podatak ako postoje pravna sredstva za identifikaciju; test „sredstava koja je razumno vjerojatno da će biti korištena" uključuje i treće osobe |
| **CJEU SRB (C-413/23 P)** | 4.9.2025. | Potvrđen „relativni pristup": pseudonimizirani podaci mogu biti osobni za jednu stranku, ali ne za drugu; za primatelja ovisi o mogućnostima re-identifikacije. Za voditelja obrade hash je uvijek osobni podatak (t. 76) |
| **WP29 WP216** | 10.4.2014. | Pseudonimizacija (uklj. hashing) nije anonimizacija; hash poznatog formata identifikatora (poput OIB-a) trivijalno se reverzira brute-force napadom |
| **EDPB Smjernice 01/2025 o pseudonimizaciji** | 16.1.2025. | Hashirani podaci su pseudonimizirani = osobni; preporuča se argon2 ili slični algoritmi otporni na brute-force |
| **EDPB Smjernice 02/2025 o blockchainu** | 8.4.2025. | Hash na blockchainu je osobni podatak; nesoljeni hashevi nedostatni za javni blockchain; tehniĉka nemogućnost ne opravdava neusklađenost |
| **CNIL smjernice o blockchainu** | 24.9.2018. | Hashevi na blockchainu su pseudonimizirani podaci; predlaže keyed hash s uništavanjem ključa kao pristup „bliži brisanju" |
| **EU Blockchain Observatory** | 16.10.2018. | „Ne postoji blockchain tehnologija usklađena s GDPR-om — postoje samo usklađeni slučajevi korištenja"; izbjegavati osobne podatke na lancu |
| **ICO smjernice (UK)** | 28.3.2025. | Hashing je pseudonimizacija; „motivated intruder" test — reverziranje hasha OIB-a zadovoljava ovaj test |
| **AEPD (Španjolska) — tehniĉka bilješka** | 11.2024. | Proof-of-concept za brisanje podataka s blockchaina putem hard forka na privatnom Ethereum lancu |

---

## 3. Blockchain i pravo na brisanje — članak 17 pred nepromjenjivim sustavima

### Fundamentalni sukob

EDPB u Smjernicama 02/2025 nedvosmisleno navodi: **„Tehnička nemogućnost ne može se pozivati kao opravdanje za neusklađenost sa zahtjevima GDPR-a."** Pravo na brisanje i pravo na prigovor **moraju biti osigurani by design** — već prilikom projektiranja sustava. Voditelj obrade dužan je osigurati da se svaki osobni podatak pohranjen na blockchainu može **učinkovito učiniti anonimnim** ako stigne zahtjev za brisanje.

### Što regulatori smatraju „funkcionalnim brisanjem"

CNIL je u smjernicama iz 2018. predložio četiri razine tehničkih rješenja, od najpoželjnijih do najnepoželjnijih:

- **Kriptografski commitment (perfectly hiding scheme)** — najsnažniji pristup. Nakon uništenja svjedoka (witness) i izvornih podataka, commitment na lancu postaje „beskoristan" i „nije više moguće niti oporaviti niti prepoznati izvorne osobne podatke" (EDPB 02/2025).
- **Keyed hash (hash s tajnim ključem/soli)** — uništavanje ključa čini hash nepovezivim s izvornim podatkom, „pod uvjetom da algoritam nije kompromitiran, ključevi nisu procurili i sol nije slaba" (EDPB 02/2025). CNIL navodi da ovo „približava učincima brisanja podataka", ali izričito ne potvrđuje da u potpunosti zadovoljava članak 17.
- **Enkripcija s uništavanjem ključa (crypto-shredding)** — podatak postaje „nerazumljiv", ali EDPB upozorava da će „čak i najsuvremenija enkripcija biti premašena vremenom ako se blockchain zadržava neograničeno".
- **Nesoljeni hash** — EDPB izričito navodi da **nije dostatan** za javni blockchain.

### Praksa industrije: crypto-shredding

Crypto-shredding — uništavanje enkripcijskih ključeva umjesto brisanja samih podataka — koriste Spotify (sustav „Padlock"), Google Cloud (putem Cloud KMS), i druge tvrtke. Međutim, **niti jedno nadzorno tijelo za zaštitu podataka nije izričito potvrdilo** da crypto-shredding u potpunosti zadovoljava članak 17. EDPB upozorava na **vremensko ograničenje** učinkovitosti: kriptografski algoritmi slabe s vremenom, a blockchain podatke zadržava zauvijek.

### Praktična implikacija za projekt

Pohrana nesoljenog keccak256(OIB) na Gnosis Chain **u potpunosti onemogućuje ostvarivanje prava na brisanje**. Čak i pristup hash(sol + OIB) ostaje u „sivoj zoni" — EDPB ga prihvaća kao mjeru koja može spriječiti buduću identifikaciju, ali ne potvrđuje bezuvjetnu usklađenost. **Jedini pristup koji EDPB smatra „beskorisnim" (tj. blizu pravoj anonimizaciji) jest perfectly hiding kriptografski commitment s uništenim svjedokom.** Projekt mora uzeti u obzir da, čak i s najboljom tehničkom mjerom, ostaje pravna nesigurnost — područje je neriješeno i predmet je daljnjeg razvoja na europskoj razini.

---

## 4. DPIA obveza — procjena učinka na zaštitu podataka je neizbježna

### Zašto je DPIA obvezan

Članak 35. stavak 1. GDPR-a zahtijeva DPIA kada je obrada „korištenjem novih tehnologija, uzimajući u obzir prirodu, opseg, kontekst i svrhe obrade, vjerojatno rezultira visokim rizikom za prava i slobode fizičkih osoba." EDPB u Smjernicama 02/2025 eksplicitno navodi da obrada osobnih podataka putem blockchaina „često uvodi specifične rizike" te da će **DPIA vjerojatno biti potreban** za blockchain obradu osobnih podataka.

Prema WP29 kriterijima (WP 248 rev.01), pohrana keccak256(OIB) na javnom blockchainu ispunjava najmanje **tri od devet kriterija** koji pokreću obvezu DPIA-a:

- **Kriterij 8**: Inovativna uporaba novih tehnoloških rješenja (blockchain)
- **Kriterij 9**: Obrada koja sprječava ispitanike u ostvarivanju prava (nemogućnost brisanja)
- **Kriterij 5**: Obrada velikih razmjera (javni blockchain distribuira podatke svim čvorovima globalno)

### AZOP-ov popis obveznih DPIA-a

AZOP je objavio Odluku o uspostavi popisa vrsta postupaka obrade koje podliježu zahtjevu za DPIA (revidiranu 21. prosinca 2018.). Popis sadrži **13 kategorija**, od kojih su za ovaj projekt najrelevantnije:

- **Kategorija 5**: Uporaba novih tehnoloških ili organizacijskih rješenja za obradu osobnih podataka
- **Kategorija 12**: Obrada upotrebom inovativnih tehnologija ili s mogućnošću obrade biometrijskih/genetskih podataka

Blockchain nije izričito naveden, ali AZOP napominje da je popis podložan razvoju i dopuni. Obrada OIB-a na javnom blockchainu nedvosmisleno potpada pod kategorije novih i inovativnih tehnologija. **DPIA je obvezan za ovaj projekt.**

### Sadržaj DPIA-a za ovaj specifični slučaj

Prema članku 35. stavku 7. GDPR-a i Smjernicama EDPB-a 02/2025, DPIA mora sadržavati:

**Sustavni opis obrade**: opis tehničke arhitekture — OIB se verificira putem Certilia Mobile.id (OIDC/JWT), derivira se keccak256 hash, hash se pohranjuje u smart contractu na Gnosis Chainu kao Soulbound Token za Sybil otpornost. Kategorija podataka: pseudonimizirani osobni podatak (hash nacionalnog identifikacijskog broja). Opseg: globalni (javni blockchain s čvorovima diljem svijeta).

**Ocjena nužnosti i proporcionalnosti**: zašto je blockchain nužan? Postoji li manje invazivna alternativa (centralizirana baza, ZK dokazi)? Je li trajna pohrana na javnom blockchainu proporcionalna cilju sprječavanja Sybil napada? EDPB izričito zahtijeva dokumentiranje razloga odabira blockchaina umjesto alternativa.

**Ocjena rizika za ispitanike**: nepromjenjivost (nemogućnost brisanja); rizik re-identifikacije (brute-force napadi na hash); međunarodni prijenosi podataka (čvorovi diljem svijeta); opasnost od povezivanja s drugim on-chain podacima; dugoročna degradacija kriptografskih funkcija.

**Mjere za smanjenje rizika**: uporaba soljenog/keyiranog hasha umjesto čistog keccak256; pohrana soli/ključa off-chain s mogućnošću brisanja; razmatranje kriptografskih commitmenta ili ZK nullifiera; governance okvir za međunarodne prijenose; mehanizmi za ostvarivanje prava ispitanika; dokumentirana procjena zašto javni blockchain umjesto privatnog/permissioniranog.

---

## 5. Pravna osnova obrade — legitimni interes kao najodržljivija opcija

### Tri opcije iz članka 6. i njihova procjena

**Privola (članak 6(1)(a)) — problematična i nepreporučena.** Privola za obradu čiji se učinci ne mogu poništiti (nepromjenjivi blockchain) suočava se s fundamentalnim problemom članka 7. stavka 3. GDPR-a koji zahtijeva da povlačenje privole bude **jednako jednostavno kao i njezino davanje**. EDPB u Smjernicama 02/2025 izričito navodi: „Kad se pohrana osobnih podataka temelji na privoli, osobni podaci moraju biti **izbrisani ili učinjeni anonimnima** ako se privola povuče." Na nepromjenjivom blockchainu to je tehnički nemoguće. Dodatno, ako je SBT minting uvjet za pristup platformi, privola nije „slobodno dana" prema članku 7. stavku 4. (uvjet pristupa usluzi). **Privola je najslabija pravna osnova za on-chain pohranu osobnih podataka.**

**Legitimni interes (članak 6(1)(f)) — najperspektivnija opcija.** Zahtijeva trodijelni test prema Smjernicama EDPB-a 1/2024:

- **Test svrhe**: Sprječavanje Sybil napada (zaštita integriteta platforme, sprječavanje prijevare) je prepoznatljiv legitimni interes. Uvodna izjava 47 identificira sprječavanje prijevare kao potencijalni legitimni interes, a uvodna izjava 49 mrežnu sigurnost.
- **Test nužnosti**: Treba dokazati da ne postoji manje invazivna alternativa koja postiže isti cilj. Ovo je **najkritičnija točka** — ako centralizirana baza ili ZK dokazi mogu postići istu Sybil otpornost, blockchain obrada može pasti na testu nužnosti.
- **Test ravnoteže**: Prava ispitanika naspram legitimnog interesa. Relevantni čimbenici: pseudonimizirani (ne cleartext) podatak, razumna očekivanja korisnika Web3 platformi, ali trajna pohrana bez mogućnosti brisanja predstavlja značajan utjecaj. Ključna prednost nad privolom: kod legitimnog interesa ne primjenjuje se pravo na povlačenje, već pravo na prigovor (članak 21.) — koje voditelj obrade može odbiti ako dokaže **uvjerljive legitimne osnove koje nadilaze** interese ispitanika.

Worldcoin/World Foundation koristio je legitimni interes (članak 6(1)(f)) za obradu iris kodova za „proof of personhood" (Sybil otpornost) — najsrodniji postojeći presedan. BayLDA je istraživao ovaj pristup, ali prigovori su se primarno odnosili na biometrijske podatke (članak 9.), a ne na sam koncept legitimnog interesa za Sybil otpornost.

**Izvršenje ugovora (članak 6(1)(b)) — uvjetno primjenjivo.** EDPB tumači „nužnost" usko — obrada mora biti objektivno nužna, ne samo korisna. Ako je platforma strukturirana tako da intrinzično zahtijeva jedinstvenost identiteta (npr. decentralizirani sustav glasovanja), hash pohrana bi mogla biti argumentirana kao nužna za izvršenje ugovora. Međutim, na javnom permissionless blockchainu „ne postoji stvarni ugovor i stvarna imenovana protustrana" (prema pravnoj analizi), što ovu osnovu čini teško primjenjivom.

### Preporuka

**Legitimni interes (članak 6(1)(f)) je najodržljivija pravna osnova**, uz dokumentiranu provedbu trodijelnog testa i implementaciju tehničkih mjera zaštite. Privolu treba izbjegavati kao primarnu osnovu.

---

## 6. ZK nullifier alternativa — značajno usklađeniji pristup

### Kako ZK nullifier funkcionira

U ZK nullifier pristupu (koji koriste protokoli poput Semaphore i World ID), korisnik generira **identitetski commitment** (hash tajnih vrijednosti) koji se pohranjuje u Merkle stablo na lancu. Pri dokazivanju jedinstvenosti, korisnik off-chain generira ZK dokaz (zk-SNARK) koji dokazuje: (1) da je član grupe (commitment postoji u Merkle stablu), i (2) da proizvodi deterministički **nullifier hash** = Poseidon(scope, tajni_skalar). Smart contract verificira dokaz i provjerava je li nullifier već korišten — ako jest, akcija se odbija.

**Ključno svojstvo**: nullifier ne otkriva ništa o identitetu korisnika, ali je deterministički za dani kontekst, čime omogućuje Sybil otpornost. Različiti konteksti (scope) proizvode različite nullifiere, pa su **akcije nepovezive između aplikacija**.

### Usporedba s hash pristupom

| Kriterij | Hash na lancu | ZK nullifier |
|----------|---------------|--------------|
| **Podatak na lancu** | Trajni identifikacijski hash | Samo nullifier hashevi (per-action) + Merkle root |
| **Povezivost** | Isti hash povezuje sve akcije korisnika | Različit nullifier po kontekstu — nepovezivo |
| **Rizik reverziranja** | Hash reverzibilan brute-force-om | Nullifier se ne može reverzirati u identitet |
| **Pravo na brisanje** | Nemoguće na nepromjenjivom blockchainu | Identitet se može ukloniti iz Merkle stabla |
| **Minimizacija podataka** | Pohranjuje trajni identifikator | Otkriva samo „da/ne" — je li osoba jedinstvena |
| **GDPR usklađenost** | Nizak stupanj usklađenosti | Značajno viši stupanj usklađenosti |

### Pravni stavovi o ZK dokazima i GDPR-u

**AEPD (Španjolska)** je objavio stručni članak koji izričito navodi da su ZK dokazi „skup tehnika koje omogućuju implementaciju dviju mjera utvrđenih u članku 25. GDPR-a: minimizaciju i ograničenje pristupačnosti podataka" te da se moraju primjenjivati „po defaultu i by design". AEPD ih kvalificira kao „moćne alate pseudonimizacije", uz napomenu da ne brišu osobne podatke.

**INATBA** (International Association for Trusted Blockchain Applications) u pozicijskom dokumentu iz 2025. zagovara ZK dokaze kao primarno rješenje za blockchain-GDPR kompatibilnost, ističući da osobni podaci mogu biti „učinkovito izbrisani opozivom kriptografskih ključeva".

Akademski rad Podda et al. (2025) u Internet Policy Review zaključuje da ZK dokazi omogućuju implementaciju načela minimizacije podataka iz GDPR-a u digitalnim novčanicima identiteta i preporučuje regulatornim tijelima da **nametnu uporabu ZK dokaza**.

### Lekcije iz Worldcoin istraga

Worldcoin (sada World) je najvažniji globalni presedan za blockchain sustave koji koriste biometrijske podatke za Sybil otpornost. **Regulatorna reakcija bila je masivna**: BayLDA (Njemačka) — korektivne mjere u prosincu 2024.; AEPD (Španjolska) — privremena zabrana u ožujku 2024.; CNPD (Portugal) — 90-dnevna zabrana u ožujku 2024.; Garante (Italija) — upozorenje u ožujku 2024.; Južna Koreja — kazna od **~860.000 USD** u rujnu 2024.; Hong Kong — naredba za prestanak rada u svibnju 2024.; Kenija — sudska naredba za trajno brisanje svih biometrijskih podataka u svibnju 2025.; Brazil — zabrana u siječnju 2025.

**Zajednički prigovori** regulatora uključuju: nedostatnu pravnu osnovu za obradu biometrijskih podataka, nemogućnost povlačenja privole, nemogućnost ostvarivanja prava na brisanje, nedostatnu transparentnost i neprovjeravanje dobi korisnika. World je u odgovoru implementirao: SMPC sustav (distribuirano šifrirano dijeljenje iris kodova), Personal Custody (podaci samo na korisnikovu uređaju), brisanje centralizirane baze iris kodova i ZK dokaze putem Semaphore protokola. Međutim, **BayLDA je utvrdio da su potrebne daljnje prilagodbe**, a ključno pravno pitanje — kvalificiraju li se SMPC-šifrirani fragmenti iris koda kao anonimni podaci — ostaje nerazriješeno. World je uložio žalbu upravo radi sudskog pojašnjenja.

### Je li nullifier osobni podatak?

Ovo je ključno nerazriješeno pravno pitanje. Argumenti da **nije**: nullifier hash je izveden iz tajni koje poznaje samo korisnik, kontekstno je specifičan i ne može se reverzirati; bez suradnje korisnika i verifikatora, re-identifikacija je računalno neizvediva. Argumenti da **jest**: pod GDPR-ovom definicijom, svaka informacija „koja se odnosi na identificiranu ili identificirabilnu osobu" može biti osobni podatak. Prema EDPB-ovu Mišljenju 28/2024 o anonimizaciji (prosinac 2024.), anonimizacija se mora procjenjivati od slučaja do slučaja, a vjerojatnost identifikacije putem „svih sredstava koja je razumno vjerojatno da će biti korištena" mora biti **zanemariva**. Dobro dizajniran nullifier sustav mogao bi potencijalno zadovoljiti ovaj kriterij — ali to dosad nije formalno potvrđeno od niti jednog regulatora.

---

## 7. Hrvatski kontekst — AZOP, zakonodavstvo i OIB

### AZOP nema smjernice o blockchainu

Na temelju opsežnog pretraživanja web stranica AZOP-a (azop.hr), **AZOP nije objavio nikakve specifične stavove, smjernice ili mišljenja o blockchainu i osobnim podacima, hashiranim identifikatorima ili pseudonimizaciji u kontekstu blockchain tehnologije**. AZOP-ov fokus u 2024.-2026. bio je na videonadzoru, bankama, telekomunikacijama i zaštiti podataka zaposlenika. Zanimljivo, direktor AZOP-a Zdravko Vukić obnaša dužnost **potpredsjednika EDPB-a**, što Hrvatsku stavlja bliže europskom mainstream pristupu.

### Zakon o provedbi GDPR-a (NN 42/18) — ključne odredbe

**Članak 20** — obrada nacionalnih identifikacijskih brojeva (uključujući OIB) mora se provoditi uz **odgovarajuće zaštitne mjere** za prava i slobode ispitanika u skladu s GDPR-om. Kršenje podliježe sankcijama prema članku 83. stavku 5. GDPR-a — do **20 milijuna EUR ili 4% ukupnog godišnjeg prometa**. Ovo je direktno primjenjivo na projekt koji obrađuje hasheve OIB-a.

**Članak 19** — dob privole za usluge informacijskog društva u Hrvatskoj je **16 godina** (GDPR dopušta raspon 13-16).

**Članak 22** — obrada biometrijskih podataka u privatnom sektoru dopuštena je samo ako je propisana zakonom ili nužna za zaštitu osoba, imovine ili sigurnu identifikaciju osobe, uz uvjet da interesi ispitanika ne prevaguju.

**Članak 47** — javna tijela u Hrvatskoj **ne mogu biti kažnjena** novčanim kaznama — ovo je hrvatska specifičnost koja nije relevantna za privatni projekt, ali je važna za kontekst.

**Ne postoje blockchain-specifične odredbe** u hrvatskom zakonu o provedbi GDPR-a.

### OIB — pravni status

OIB je **nedvosmisleno osobni podatak** prema hrvatskom pravu. Zakon o osobnom identifikacijskom broju (NN 60/08) definira OIB kao trajni, nekazujući, nepromjenjivi i neponovljivi identifikacijski broj od 11 znamenki. Iako OIB sam po sebi nije klasificiran kao tajna (pojavljuje se na osobnim iskaznicama i računima), **zaštita podataka primjenjuje se na obradu OIB-a**. Članak 20. hrvatskog zakona o provedbi GDPR-a nameće pojačane zahtjeve za obradu nacionalnih identifikacijskih brojeva.

### Certilia Mobile.id

Certilia **nije blockchain sustav** — to je tradicionalno PKI rješenje za mobilni digitalni identitet, razvijeno od AKD-a (Agencija za komercijalnu djelatnost). Certilia koristi kvalificirane elektroničke potpise (QES), integrirana je s NIAS-om i e-Građanima, te ima preko **350.000 korisnika**. Biometrija se koristi isključivo lokalno na uređaju. **Ne postoji javno dostupna dokumentacija o blockchain komponenti Certilije niti o specifičnom GDPR pristupu** za blockchain integraciju.

### Sudska praksa

**Ne postoje hrvatske sudske presude koje se specifično bave pseudonimizacijom, hashiranjem ili blockchainom** u kontekstu zaštite podataka. Odluke AZOP-a osporavaju se upravnim sporom pred upravnim sudovima, ali objavljeni slučajevi ne uključuju relevantnu problematiku.

---

## 8. Preporuke za projekt — konkretne mjere

### Preporuka 1: Ne pohranjivati nesoljeni keccak256(OIB) na javni blockchain

Ovo je **fundamentalna preporuka**. EDPB izričito navodi da nesoljeni hashevi nisu dostatni za javni blockchain. Keccak256(OIB) se trivijalno reverzira rainbow table napadom. Čak i s obzirom na pravnu nesigurnost, ovaj pristup nosi neprihvatljiv rizik regulatorne intervencije.

### Preporuka 2: Razmotriti ZK nullifier umjesto hasha

ZK nullifier pristup (npr. putem Semaphore protokola) značajno je usklađeniji s GDPR-om. Na lancu se ne pohranjuje pseudonimni identifikator, već samo kontekstno-specifični nullifier i Merkle root. Ispitanik se može ukloniti iz grupe, a različite akcije su nepovezive. Ovo je smjer u kojem se kreće industrija (World ID, Zupass) i koji regulatori sve više prepoznaju (AEPD).

### Preporuka 3: Ako se ipak koristi hash, implementirati keyed hash s mogućnošću uništenja ključa

Ako se projekt ipak odluči za hash pristup, minimalni zahtjev je hash(sol + OIB) gdje je sol jedinstvena po korisniku, dovoljno duga, pohranjena off-chain i pod isključivom kontrolom korisnika. Korisnik mora imati mogućnost uništenja soli, čime hash na lancu postaje nepoveziv s izvornim podatkom. Ovo nije zagarantirano usklađeno s člankom 17., ali je pristup koji CNIL i EDPB smatraju „bliže brisanju".

### Preporuka 4: Provesti DPIA prije pokretanja sustava

DPIA je obvezan. Mora sadržavati: sustavni opis obrade i tehničke arhitekture, ocjenu nužnosti blockchaina naspram alternativa (centralizirana baza, ZK dokazi), analizu rizika re-identifikacije (uključujući kvantitativnu procjenu otpornosti hasha na brute-force), mjere za smanjenje rizika, procjenu utjecaja na prava ispitanika (posebice pravo na brisanje i ispravak) te plan za međunarodne prijenose podataka (Gnosis Chain čvorovi globalno).

### Preporuka 5: Koristiti legitimni interes kao pravnu osnovu

Dokumentirati trodijelni test: legitimnost cilja (Sybil otpornost/sprječavanje prijevare), nužnost obrade (zašto ne postoji manje invazivna alternativa), i balansiranje interesa. **Ključno**: na testu nužnosti treba argumentirati zašto ZK nullifier ili centralizirana baza ne postižu isti cilj — ili, ako to nije moguće argumentirati, prihvatiti da test nužnosti nije zadovoljen i prijeći na ZK pristup.

### Preporuka 6: Dokumentirati razlog odabira javnog blockchaina

EDPB daje **jasnu prednost privatnim/permissioniranim blockchainima** i navodi da javni blockchaini „trebaju biti korišteni samo ako je javni pristup blockchainu nužan za barem jednu od svrha obrade". Projekt mora dokumentirati zašto je Gnosis Chain (javni blockchain) nužan umjesto privatnog lanca.

### Preporuka 7: Implementirati mehanizme za ostvarivanje prava ispitanika

Neovisno o tehničkom pristupu, sustav mora omogućiti: pravo na informiranje (transparentan opis obrade), pravo na pristup (ispitanik mora moći doznati je li njegov hash na lancu), pravo na prigovor (mehanizam za blokiranje budućeg korištenja hasha), i — u mjeri u kojoj je moguće — pravo na brisanje (uništavanje off-chain komponenti poput soli ili svjedoka).

### Preporuka 8: Pratiti razvoj regulatornog okvira

Smjernice EDPB-a 02/2025 su u javnoj konzultaciji do lipnja 2025. — konačna verzija može donijeti promjene. Presuda CJEU-a u predmetu SRB (C-413/23 P) iz rujna 2025. otvorila je mogućnost „relativnog pristupa" pseudonimizaciji, što bi u teoriji moglo ići u korist projekta — ali na javnom blockchainu, gdje svatko može pokušati re-identifikaciju, relativni pristup zapravo ne pomaže. World/Worldcoin žalba na odluku BayLDA mogla bi donijeti prvo sudsko tumačenje o tome kvalificiraju li se privacy-enhancing tehnologije kao anonimizacija.

---

## 9. Izvori

### Zakonodavstvo i uredbe
- Uredba (EU) 2016/679 (GDPR), osobito članci 4(1), 5, 6, 17, 25, 35 i Uvodne izjave 26, 47, 49
- Zakon o provedbi Opće uredbe o zaštiti podataka, NN 42/18, osobito članci 19, 20, 22, 47
- Zakon o osobnom identifikacijskom broju, NN 60/08

### Presude CJEU-a
- C-582/14, Breyer protiv Bundesrepublik Deutschland, 19.10.2016., ECLI:EU:C:2016:779
- C-413/23 P, EDPS protiv Single Resolution Board, 4.9.2025., ECLI:EU:C:2025:645

### Smjernice EDPB-a
- EDPB Smjernice 02/2025 o obradi osobnih podataka putem blockchain tehnologija, 8.4.2025.
- EDPB Smjernice 01/2025 o pseudonimizaciji, 16.1.2025.
- EDPB Smjernice 1/2024 o obradi osobnih podataka temeljem članka 6(1)(f), listopad 2024.
- EDPB Smjernice 05/2020 o privoli, verzija 1.1
- EDPB Mišljenje 28/2024 o anonimizaciji u kontekstu AI modela, 17.12.2024.

### Mišljenja Radne skupine iz članka 29
- WP216, Mišljenje 05/2014 o tehnikama anonimizacije, 10.4.2014.
- WP248 rev.01, Smjernice o procjeni učinka na zaštitu podataka (DPIA), 4.10.2017.

### Nacionalna nadzorna tijela
- CNIL, „Blockchain and the GDPR: Solutions for a responsible use", rujan 2018.
- BayLDA, Odluka o korektivnim mjerama protiv Worldcoin/World, 19.12.2024.
- AEPD, Privremena zabrana Worldcoinu, 6.3.2024. i naredba za brisanje, prosinac 2024.
- CNPD (Portugal), Privremena zabrana Worldcoinu, Deliberacija 2024/137, 25.3.2024.
- AEPD, Tehnička bilješka: Proof of Concept — blockchain i pravo na brisanje, studeni 2024.
- AEPD, Blog: „Encryption and Privacy IV: Zero-knowledge proofs"
- ICO, Anonymisation and Pseudonymisation Guidance, 28.3.2025.
- AZOP, Odluka o popisu vrsta postupaka obrade koje podliježu DPIA-u

### Ostali izvori
- EU Blockchain Observatory and Forum, „Blockchain and the GDPR", 16.10.2018.
- INATBA, „Leveraging Zero-Knowledge Proofs for GDPR Compliance in Blockchain Projects", 2025.
- Podda et al., „The impact of zero-knowledge proofs on data minimisation compliance of digital identity wallets", Internet Policy Review, Vol. 14, Issue 3, 2025.
- PIPC (Južna Koreja), Kazna za Worldcoin, 25.9.2024.
- Kenijski Visoki sud, Naredba za trajno brisanje biometrijskih podataka Worldcoina, 5.5.2025.
