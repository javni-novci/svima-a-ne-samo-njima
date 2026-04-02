# eIDAS 2.0 / EUDI novčanik — Perplexity izvještaj

**Prompt:** [05-eidas2-eudi.md](../prompts/05-eidas2-eudi.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

# eIDAS 2.0 i EUDI novčanik: vremenski okvir, tehničke specifikacije i implikacije za hrvatske sustave

## 1. Sažetak

Regulativa eIDAS 2.0 (Uredba (EU) 2024/1183) objavljena je u SL EU 30. travnja 2024. i stupila je na snagu 20. svibnja 2024., čime je pravno uspostavljen okvir za Europski digitalni identitetski novčanik (EUDI Wallet). Države članice u roku od otprilike 24 mjeseca od donošenja provedbenih akata moraju osigurati najmanje jedan certificirani EUDI novčanik, što se u praksi prevodi u rok krajem 2026., dok je prihvaćanje novčanika od strane javnog i većine privatnih pružatelja usluga obvezno do kraja 2027.[^1][^2][^3][^4][^5]

EUDI novčanik se tehnički temelji na kombinaciji ISO/IEC 18013‑5 (mdoc) za bliske (proximity) scenarije i SD‑JWT‑VC (Selective Disclosure JWT Verifiable Credentials) za udaljene web scenarije, preko OpenID4VCI (izdavanje) i OpenID4VP/SIOPv2 (prezentacija), uz JOSE/COSE kriptografiju usklađenu sa SOG‑IS smjernicama. ARF i PID Rulebook namjerno izbjegavaju globalni jedinstveni identifikator osobe; umjesto toga koriste skup PID atributa i kriptografsko vezivanje (device/key binding), što je važno za vaše pitanje o Sybil zaštiti.[^6][^7][^8][^9][^10][^11][^12][^13][^14]

Hrvatska koristi NIAS kao nacionalnu eID shemu te Certilia Mobile.ID kao mobilni credential visoke razine pouzdanosti; ti sustavi ostaju relevantni i pod eIDAS 2.0 te će vjerojatno služiti kao podloga za izdavanje PID‑a u EUDI novčanik, a ne kao nešto što je automatski zamijenjeno. EUDI okvir nigdje ne propisuje blockchain/DLT niti standardni „nullifier“, iako pojedini piloti (posebno DC4EU kroz EBSI) eksperimentiraju s blockchainom za registraciju/validaciju vjerodajnica; stoga se blockchain sada može promatrati samo kao opcionalni sloj iznad standardnog PKI‑temeljenog ekosustava.[^15][^16][^17][^18][^19][^6]


## 2. Vremenski okvir i obveze

### 2.1 Ključni datumi i rokovi

| Događaj | Opis | Pravna/osnovna referenca | Datum / rok |
|--------|------|--------------------------|-------------|
| Objavljivanje eIDAS 2.0 | Uredba (EU) 2024/1183 objavljena u Službenom listu EU | Službeni sažeci eIDAS 2.0[^1][^20] | 30.4.2024. |
| Stupanje na snagu | Uredba stupa na snagu 20. dan nakon objave | Isti izvori[^1][^20][^2] | 20.5.2024. |
| Donošenje provedbenih akata | Skup 5 provedbenih uredbi Komisije (funkcionalnosti, PID, protokoli, obavješćivanje, certificiranje) usvojen 28.11.2024. | Pregled EUDI Wallet na Wikipediji[^2] | 28.11.2024. |
| Rok za dostupnost barem jednog EUDI novčanika | Države članice moraju osigurati najmanje jedan certificirani EUDI novčanik u roku 24 mjeseca od provedbenih akata; pravni i stručni komentari to prevode na „kraj 2026.“ (npr. 6.12.2026.) | Pravna analiza i komentari odvjetničkih kuća i IAM dobavljača[^3][^21][^22][^4][^5] | Praktično: kraj 2026. |
| Obvezno prihvaćanje u javnom i polu‑javnom sektoru | Tijela javnog sektora i regulirani sektori moraju prihvaćati EUDI novčanik za autentikaciju i identifikaciju najkasnije 36 mjeseci nakon provedbenih akata | Sažeci rokova eIDAS 2.0[^23][^4][^5] | Kraj 2027. |
| Obvezno prihvaćanje za većinu privatnog sektora | Velike online platforme i većina privatnih subjekata koji moraju jednoznačno identificirati korisnike moraju prihvaćati EUDI novčanik u istom 36‑mjesečnom horizontu | Isti izvori[^23][^4][^5] | Tijekom/ do kraja 2027. |
| Cilj Digitalnog desetljeća | Politički cilj EU: ~80% građana da koristi europski digitalni identitet (uključivo EUDI novčanik) do 2030. | Digital Decade i analize stanja[^24][^22] | 2030. |

### 2.2 Status Hrvatske

Hrvatska nacionalna eID shema NIAS je notifikacijom priopćena pod eIDAS‑om razine pouzdanosti „high“, pri čemu je nadležni organ tada bio Središnji državni ured za razvoj digitalnog društva. Postojeći mobilni identitet Certilia Mobile.ID temelji se na eOI i NIAS‑u te je već danas sredstvo visoke razine pouzdanosti za pristup javnim e‑uslugama i udaljeno potpisivanje.[^16][^17][^18][^19]

Javni izvještaj s Trust Services and eID Foruma 2025. navodi da Hrvatska integrira EUDI okvir u nacionalnu Digitalnu informacijsku infrastrukturu (DII), usklađuje se s ARF‑om te planira izdavanje certificiranog EUDI novčanika u Q4 2026., uz razvoj novčanika, uspostavu certifikacijske sheme i pripremu atributnih izvora. Neovisna analitika iz 2025. ističe da Hrvatska, zajedno s Bugarskom, Maltom i Rumunjskom, kasni s formalnim nacionalnim EUDI inicijativama, ali koristi iskustva ranijih država kroz sudjelovanje u konzorcijima i radnim skupinama.[^25][^26]

### 2.3 Tko je u Hrvatskoj „nosač“ EUDI novčanika?

NIAS shemom upravlja država (trenutno Ministarstvo nadležno za digitalnu transformaciju, kao nasljednik ranijeg državnog ureda), uz AKD/MUP kao ključnog izdavatelja eOI i pripadajućih vjerodajnica. FINA je kvalificirani pružatelj usluga povjerenja (QTSP), no javni dokumenti dosad je eksplicitno ne navode kao budućeg „EUDI Wallet Providera“. Prema izlaganju predstavnika Ministarstva digitalne transformacije, formirana je nacionalna radna skupina koja priprema provedbeni akt, no nije javno objavljeno koja će konkretna organizacija biti certificirani pružatelj EUDI novčanika; realno je očekivati da će kombinacija Ministarstva/NIAS‑a, AKD‑a i QTSP‑ova (npr. FINA) biti uključena, ali formalna odluka još nije javno objavljena.[^18][^25][^16]


## 3. Tehničke specifikacije EUDI novčanika

### 3.1 Pregled standarda i formata

EUDI Architecture and Reference Framework (ARF) definira da je EUDI novčanik referentno rješenje koje koristi skup standarda: ISO/IEC 18013‑5 (mdoc) za mobilne identifikacijske dokumente, IETF SD‑JWT‑VC za selektivno otkrivanje u web scenarijima, W3C Verifiable Credentials (posebno profil VC‑JOSE‑COSE), te OpenID4VCI i OpenID4VP/SIOPv2 kao transportni sloj. PID Rulebook dodatno specificira PID kao vjerodajnicu koja se može kodirati ili kao ISO mdoc ili kao SD‑JWT‑VC, uz obveznu podršku za selektivno otkrivanje.[^7][^8][^27][^9][^28][^10][^12][^13][^14][^6]

#### Tablica: Tehničke specifikacije — formati i protokoli

| Element | Specifikacija / standard | Napomena |
|--------|--------------------------|----------|
| Osnovni tipovi vjerodajnica | PID (Person Identification Data), (Qualified) Electronic Attestations of Attributes — (Q)EAA | Definicije u ARF‑u i izmijenjenom eIDAS‑u[^6][^2] |
| Formati vjerodajnica (općenito) | „Attestation“ je potpisani skup atributa u mdoc formatu (ISO/IEC 18013‑5) ili SD‑JWT formatu (SD‑JWT‑VC) | ARF definira attestation upravo tako[^6][^12][^13] |
| PID format | Dvije kodiranja: ISO/IEC 18013‑5‑kompatibilni mdoc (CBOR, mobilni dokument) i SD‑JWT‑VC (JSON/JWT), oba s istom logičkom PID shemom | PID Rulebook i ARF[^6][^12][^13][^11] |
| (Q)EAA format | Preporučeno CBOR i/ili JSON, mogu koristiti iste mehanizme kao PID (mdoc ili SD‑JWT‑VC) te opcionalno W3C VC JSON‑LD za specifične sektore | ARF i analize formata[^6][^27][^11] |
| W3C VC profil | ARF i radna skupina standarda navode W3C VC‑JOSE‑COSE kao ključnu specifikaciju za potpisivanje/verifikaciju VC‑ova; DC4EU je demonstrirao W3C VC JSON‑LD profil u EUDI referentnoj implementaciji | W3C VC‑JOSE‑COSE bilješka i izvješća DC4EU[^29][^14] |
| Transport — izdavanje | OpenID for Verifiable Credential Issuance (OpenID4VCI) | ARF i OpenID dokumentacija; ovo je osnova za interoperabilno izdavanje PID‑a i (Q)EAA‑a[^6][^30][^10] |
| Transport — prezentacija | OpenID for Verifiable Presentations (OpenID4VP) + Self‑Issued OpenID Provider v2 (SIOPv2) za udaljene scenarije; ISO/IEC 18013‑5 za bliske/proximity scenarije, ISO/IEC 18013‑7 za online mDL | ARF, ISO i vodiči za EUDI/OpenID4VC[^6][^8][^27][^30][^10] |
| Selektivno otkrivanje | SD‑JWT‑VC za web (remote) slučajeve, ISO/IEC 18013‑5 mdoc (Mobile Security Object + device binding) za proximity; selektivno otkrivanje eksplicitno definirano kao sposobnost EUDI novčanika | ARF definira „Selective Disclosure“, PID Rulebook opisuje SD‑JWT‑VC kodiranje[^6][^7][^31][^32][^12][^13] |
| Kriptografija — okvir | ARF upućuje na JOSE/COSE RFC‑ove i SOG‑IS „Agreed Cryptographic Mechanisms“ v1.2; konkretne liste algoritama definiraju se kroz te profile i nacionalne sheme | ARF verzija 1.2.0 i novije online verzije[^6][^8][^33][^14] |
| Uobičajeni algoritmi u implementacijama | ECDSA na P‑256 (ES256) i P‑384, RSA‑PSS/RSASSA‑PKCS1‑v1_5, EdDSA/Ed25519, u JOSE/COSE kontekstu | Dokumentacija EUDI i ekosustav alata (SD‑JWT biblioteka, potpisni moduli, komercijalne implementacije)[^34][^35][^36] |
| Sigurni elementi / WSCD | ARF uvodi Wallet Secure Cryptographic Device (WSCD) i Wallet Secure Cryptographic Application (WSCA); eIDAS 2.0 i stručni tekstovi naglašavaju korištenje sigurnih elemenata/TEE‑ova/QSCD‑ova za LoA High | ARF, sigurnosne analize i GlobalPlatform blog[^6][^37][^38][^39] |
| Potpisi i QES | EUDI novčanik mora omogućiti stvaranje kvalificiranih elektroničkih potpisa/pečata putem lokalnog ili udaljenog QSCD‑a; interakcije će biti dodatno specificirane u budućim verzijama ARF‑a | ARF, eIDAS 2.0 tekst i stručne analize QSCD‑ova[^6][^2][^37] |

### 3.2 Selektivno otkrivanje i ZK dokazi

ARF definira selektivno otkrivanje kao temeljnu sposobnost EUDI novčanika, tj. mogućnost prezentiranja podskupa PID/(Q)EAA atributa. Za udaljene (web) scenarije standardizirani mehanizam je SD‑JWT‑VC, koji omogućuje da se od vjerodajnice otkrije samo minimalan skup atributa, dok ostatak ostaje skriven, uz kriptografski dokaz integriteta. Za proximity slučajeve koristi se ISO/IEC 18013‑5 mdoc format i MSO (Mobile Security Object) koji također podržava selektivno otkrivanje atributa putem čitanja samo traženih podskupova podataka.[^27][^9][^32][^11][^12][^6]

Sam eIDAS 2.0 (npr. Recital 29 i čl. 5a) eksplicitno ističe selektivno otkrivanje i načelo minimizacije podataka kao cilj, ali ne propisuje konkretan ZK sustav; analize ARF‑a primjećuju da trenutačne konfiguracije (SD‑JWT, mdoc) nude selektivno otkrivanje, ali ne zadovoljavaju u potpunosti zahtjeve za nelinkabilnost bez naprednijih ZK dokaza. U stručnim materijalima o EUDI Walletu ZK dokazi se spominju kao poželjna nadogradnja (npr. range proofs, nejednakosti), ali zasad nisu normativno specificirani u ARF‑u ni u provedbenim aktima.[^40][^31]

### 3.3 Kriptografski algoritmi (realističan pogled)

Normativno, ARF navodi da se algoritmi moraju birati iz skupa odobrenog u SOG‑IS Agreed Cryptographic Mechanisms (v1.2) te iz odgovarajućih JOSE/COSE profila, što u praksi znači asimetrične algoritme poput ECDSA na krivuljama P‑256/P‑384, RSA‑PSS/RSASSA‑PKCS1‑v1_5 te, sve više, EdDSA/Ed25519. EUDI open‑source biblioteke i reference implementacije (npr. jvm SD‑JWT biblioteka) demonstriraju korištenje ECDSA s P‑521 (ES512) za potpisivanje SD‑JWT‑ova, ali to je primjer implementacije, ne jedini dopušteni izbor.[^34][^33][^14][^36][^6]

Za vas je ključno da su svi ovi algoritmi klasični (ne post‑kvantni) i većinom podržani u modernim kriptografskim knjižnicama, no neke kombinacije (npr. ECDSA P‑256 u EVM pametnim ugovorima) mogu biti skupe ili potrebuju specijalizirane prekompajlirane funkcije, dok EdDSA/Ed25519 često ima bolju podršku u nekim blockchainovima.[^41][^36]


## 4. ARF i PID (Architecture and Reference Framework)

### 4.1 Najnovija verzija ARF‑a

Javna tablica na eudi.dev navodi da je ARF verzija 1.0.0 objavljena 26.1.2023., verzija 1.1.0 20.4.2023., a verzija 1.2.0 u studenom 2023., pri čemu 1.2.0 uvodi novi poglavlje o trust modelu te anekse s prvim iteracijama PID i mDL Rulebooka. Online instanca ARF‑a na eudi.dev se povremeno ažurira (npr. 1.3.x, 1.4.x) kao radna verzija, ali 1.2.0 je i dalje eksplicitno navedena kao ključna referentna točka Toolboxa prije usvajanja provedbenih akata.[^8][^6]

ARF jasno ističe da je dokument radni i nenormativni, te da će se uskladiti s konačnim zakonodavnim tekstom i provedbenim aktima; obvezujuće su samo uredba eIDAS 2.0 i pripadajuće provedbene/delegirane uredbe.[^2][^6]

### 4.2 PID: format i jedinstveni identifikator

PID Rulebook definira PID kao skup podataka koji omogućuje utvrđivanje identiteta fizičke (ili pravne) osobe, s obveznim atributima za fizičke osobe poput family_name, given_name i birth_date, te nizom opcionalnih atributa (mjesto rođenja, adresa, državljanstvo, indikatori dobi, itd.). Dokument propisuje dvije ekvivalentne kodne reprezentacije:[^12][^13]

- ISO/IEC 18013‑5‑kompatibilni mdoc (CBOR) PID, i 
- SD‑JWT‑VC kodiranje PID‑a, koje koristi JSON/JWT format i SD‑JWT selektivno otkrivanje.[^13][^12]

ARF i PID Rulebook **namjerno ne uvode globalni jedinstveni identifikator osobe** (poput EU‑wide „subject ID“); umjesto toga, oslanjaju se na kombinaciju PID atributa i lokalnih/nacionalnih identifikatora, uz pravila da dvije osobe ne smiju imati isti skup obveznih PID atributa. Uz to, svaki EUDI novčanik ima vlastiti Wallet Instance attestation i interni identifikator koji služi za upravljanje instancom i revokaciju, ali ne predstavlja stabilni, javno vidljivi globalni ID za korisnika.[^6][^12][^13]

Za vaš slučaj to znači da u EUDI svijetu nema standardiziranog „nullifiera“ niti jedinstvenog pseudonima koji biste mogli koristiti kao Sybil zaštitu preko različitih Relying Partyja bez narušavanja privatnosti; umjesto toga, predviđen je model s različitim, kontekstnim pseudonimima po RP‑u/usluzi.[^31][^6]

### 4.3 Potpisivanje transakcija EUDI novčanikom

ARF navodi da korisnici putem EUDI novčanika mogu stvarati kvalificirane elektroničke potpise i pečate, bilo da je sam novčanik certificiran kao QSCD ili da služi kao sigurni front‑end prema udaljenom QSCD‑u kojim upravlja QTSP. Time EUDI novčanik de facto postaje generički potpisni klijent: isti par ključeva/certifikata može se koristiti za potpis dokumenata, ugovora ili transakcija (uključujući potpisivanje zahtjeva prema pametnim ugovorima, ako vaš backend to modelira kao „dokument za potpis“).[^37][^38][^6]

To je važno jer implicira da vaš postojeći tok za „remote signing“ preko Certilie i kvalificiranog certifikata konceptualno može biti preslikan na EUDI novčanik + QTSP, uz promjenu protokola, ali bez nužne promjene pravne semantike potpisa (QES ostaje QES).[^2][^37]

### 4.4 Blockchain / DLT u ARF‑u

ARF i PID Rulebook ne spominju blockchain ni DLT; povjerenje se temelji na PKI‑ju, listama povjerenja (trusted lists), nacionalnim akreditacijskim tijelima i certificiranim pružateljima usluga povjerenja, a ne na distribuiranim knjigama. Jedini formalno spomenuti „ledger“ u izmijenjenom eIDAS‑u jest „electronic ledger“ kao moguća usluga povjerenja (npr. za evidenciju podataka), ali to je generički koncept, ne specifična tehnologija poput EBSI‑ja ili javnih blockchainova.[^13][^6][^2]

S druge strane, DC4EU pilot i druge inicijative eksplicitno eksperimentiraju s EBSI‑jem (European Blockchain Services Infrastructure) kao ledgerom za određene javne vjerodajnice i trust frameworke; to je, međutim, komplementarni sloj iznad standardnog EUDI ekosustava i zasad nije normativno dio Toolboxa.[^15]


## 5. Pilot projekti i sudjelovanje Hrvatske

### 5.1 Četiri Large Scale Pilots (LSP)

EU financira četiri glavna LSP projekta za EUDI novčanik:[^42][^43]

- **EWC (EUDI Wallet Consortium)** – fokus na putovanja, plaćanja i organizacijsku digitalnu identitet (ODI), s preko 60 sudionika iz javnog i privatnog sektora; konzorcij uključuje predstavnike **svih 27 država članica i EFTA**.[^44][^45][^46]
- **POTENTIAL** – konzorcij 19 država članica i Ukrajine, fokusiran na niz slučajeva (bankarstvo, eGovernment itd.); popis članica uključuje Austriju, Belgiju, Cipar, Češku, Estoniju, Finsku, Francusku, Njemačku, Grčku, Mađarsku, Italiju, Litvu, Luksemburg, Nizozemsku, Poljsku, Portugal, Slovačku, Sloveniju, Španjolsku i Ukrajinu (Hrvatska nije navedena).[^47]
- **NOBID** – nordijsko‑baltički projekt primarno fokusiran na plaćanja, s partnerskim zemljama Norveškom, Danskom, Islandom, Latvijom, Italijom i Njemačkom; ostatak Nordijâ/Baltika sudjeluje kroz širi NOBID okvir.[^48][^49][^50]
- **DC4EU (Digital Credentials for Europe)** – fokus na obrazovne i socijalne vjerodajnice, uključujući W3C VC profile i integraciju s EUDI referentnom implementacijom; finalno izvješće opisuje pluralistički trust model i korištenje EBSI‑ja.[^29][^15]

### 5.2 Uključenost Hrvatske

Unutar EWC konzorcija registriran je „Identity Consortium: Croatia“ kao pridruženi partner pod državom HRV, a hrvatska tvrtka Identyum javno komunicira da je jedini hrvatski ID Wallet provider uključen u ovu inicijativu. To znači da Hrvatska kroz EWC sudjeluje u testiranju EUDI novčanika, barem u kontekstu putovanja, plaćanja i B2B/ODI scenarija, iako formalni nacionalni EUDI novčanik još nije objavljen.[^51][^52][^45][^42]

Za POTENTIAL i NOBID nema indikacija da Hrvatska sudjeluje kao država članica konzorcija, dok se DC4EU fokusira primarno na druge države, iako se rezultati projekta (kod, specifikacije) objavljuju kao javno dobro primjenjivo i u Hrvatskoj.[^43][^47][^15]

### 5.3 Blockchain / Web3 u pilotima

DC4EU finalno izvješće opisuje **pluralistički trust model** u kojem se kombiniraju: 

- EBSI blockchain za nove javne vjerodajnice i trajne, verifikabilne zapise,
- klasični PKI i eIDAS usluge povjerenja za postojeće vjerodajnice, i
- OpenID‑bazirani protokoli za akademske i druge federirane okoline.[^15]

To pokazuje da je **blockchain u EUDI kontekstu promatran kao dodatni sloj za verifikaciju i audit**, ne kao primarni trust anchor; ne postoji formalizirana specifikacija „on‑chain verifikacije EUDI vjerodajnica“ u Toolboxu ili ARF‑u. Web3 integracije u većini slučajeva ostaju eksperimentalne i projektno‑specifične.[^6][^15]

### 5.4 Javna open‑source implementacija i SDK‑ovi

Europska komisija vodi GitHub organizaciju **eu-digital-identity-wallet** koja sadrži **referentnu implementaciju EUDI novčanika** i prateće biblioteke:[^53][^54]

- eudi‑app‑ios‑wallet‑ui i eudi‑app‑android‑wallet‑ui – prototip mobilnih novčanika (iOS/Kotlin).
- eudi‑lib‑ios‑wallet‑kit i eudi‑lib‑android‑verifier‑core – knjižnice za izradu vlastitog novčanika/verifikatora, uključivo ISO 18013‑5 podršku.
- eudi‑lib‑jvm‑sdjwt‑kt – JVM/Kotlin biblioteka za SD‑JWT‑VC, s DSL‑om za definiranje selektivno otkrivih tvrdnji.[^34]

Ovo predstavlja službeni ulaz za integraciju; dodatno, treće strane (igrant.io, Procivis One, i dr.) nude vlastite SDK‑ove usklađene s OpenID4VC i EUDI profilima.[^30][^55][^27]


## 6. Učinak na Certiliju i postojeći OIDC tok

### 6.1 Hoće li Certilia biti zamijenjena EUDI novčanikom?

eIDAS 2.0 **ne ukida** postojeće notifikovane eID sheme (poput NIAS‑a) nego ih nadograđuje EUDI novčanikom kao novom komponentom; postojeći eID i usluge povjerenja ostaju na snazi, a EUDI novčanik se na njih naslanja kao izdavatelj i nositelj PID‑a i (Q)EAA‑a. U Hrvatskoj NIAS i eOI (s mobilnim credentialom Certilia Mobile.ID) ostaju temeljni identitetski sloj, a EUDI novčanik će s velikom vjerojatnošću koristiti iste izvore (NIAS, eOI) za inicijalnu identifikaciju i izdavanje PID‑a.[^17][^56][^19][^16][^18][^2]

Certilia se danas koristi kao mobilno sredstvo za autentikaciju i udaljeni potpis putem OIDC‑a i kvalificiranih certifikata; EUDI novčanik uveden prema eIDAS 2.0 vjerojatno će koegzistirati, pri čemu će se dio slučajeva (posebno prekogranična autentikacija i dijeljenje atributa) postupno preusmjeravati s „klasičnog“ OIDC IdP‑a (Certilia/NIAS) na OpenID4VP prezentacije iz EUDI novčanika.[^25][^2]

### 6.2 OIDC vs OpenID4VP u praksi

Današnji tok s Certilijom obično izgleda kao standardni OIDC Authorization Code Flow: 

1. Aplikacija preusmjerava korisnika na Certilia/NIAS OIDC endpoint.
2. Nakon autentikacije korisnika, RP prima ID Token i (po potrebi) Access Token s tvrdnjama o identitetu.

U EUDI kontekstu, tipičan tok s OpenID4VP/SIOPv2 izgleda ovako:[^28][^10][^30]

1. RP (verifier) objavljuje svoj `openid-configuration` i `openid4vp` metadata (podržane credential typeove, formate, zahtijevane atribute).
2. RP pokreće „presentation request“ (QR kod / deeplink) koji klijent (EUDI novčanik) preuzima.
3. Novčanik od korisnika traži suglasnost, selektivno bira atribute iz odgovarajućih vjerodajnica (PID, (Q)EAA) i formira Verifiable Presentation (VP Token) potpisan korisnikovim i/ili izdavateljevom ključem.
4. RP validira VP Token (JOSE/COSE potpisi, revokacija, chain of trust) i iz njega izvlači atribute; može, ali ne mora, dobiti OIDC ID Token u klasičnom smislu.

Za vas to znači da **i dalje postoji preusmjeravanje i tok „auth‑code‑like“**, ali se semantika mijenja: umjesto da vjerujete tvrdnjama koje izdaje IdP (Certilia), vjerujete VC‑ovima koji su već potpisani od strane PID/(Q)EAA izdavatelja i prezentirani kroz novčanik. 

### 6.3 Koliko se tok mora promijeniti?

Za postojeću aplikaciju koja koristi Certiliu kao OIDC IdP, razina promjene ovisit će o tome kako tretirate identitet:

- Ako aplikacija već sada radi s generičkim OIDC claimovima (`sub`, `given_name`, `family_name`, itd.), osnovna promjena je u sloju integracije: umjesto ID Tokena, potrebno je parsirati VP Token i mapirati atribute iz PID/(Q)EAA shema na vaše interné modele korisnika.[^10][^30]
- Ako aplikacija snažno ovisi o specifičnostima NIAS/Certilia (npr. određen format OIB‑a, interne tehničke identifikatore), trebat će dodatni sloj normalizacije i mapiranja, jer EUDI PID shema nije 1:1 preslikavanje postojećih lokalnih identifikatora.[^12][^13]

S arhitektonske strane, isplativ je pristup u kojem se uvodi **poseban „identity/credential verification service“** koji apstrahira detalje protokola (klasični OIDC IdP vs OpenID4VP) i aplikaciji isporučuje već normalizirane atribute i interne ID‑ove. Time se promjena protokola u budućnosti (npr. prelazak sa SD‑JWT‑VC na VC‑JOSE‑COSE profil) ne razlijeva po cijeloj kodnoj bazi. 


## 7. Kompatibilnost s blockchainom i pametnim ugovorima

### 7.1 Može li se EUDI VC verificirati na blockchainu?

Standardni EUDI VC (PID ili (Q)EAA) potpisan je JOSE/COSE potpisom koristeći klasične algoritme (ECDSA P‑256/P‑384, RSA‑PSS, eventualno EdDSA/Ed25519), uz lance X.509 certifikata izdavatelja temeljene na nacionalnim korijenskim CA‑ovima. Teoretski ništa ne sprječava da se takav potpis verificira unutar pametnog ugovora, pod uvjetom da blockchain:[^14][^12][^13][^6]

- podržava odgovarajući algoritam i
- može na neki način pristupiti javnim ključevima/certifikatima izdavatelja (npr. preko orakla ili ugrađene liste).[^41][^15]

U praksi, provjera ECDSA P‑256 potpisa unutar EVM‑kompatibilnih lanaca je skupa i kompleksna (jer nativna prekompajlirana funkcija obično podržava secp256k1, ne P‑256), dok EdDSA i druge kombinacije zahtijevaju dodatne prekompajlirane funkcije ili posebne VM‑ove. Nadalje, provjera statusa revokacije (CRL/OCSP, status QEAA/PID‑a) i povjerenja u izdavatelja (`Trusted List`) teško je modelirati potpuno on‑chain; zato većina ozbiljnih arhitektura ostavlja verifikaciju vjerodajnica **off‑chain**, a pametnim ugovorima se šalju samo minimalni dokazi ili hash‑commitmenti.[^36][^41]

### 7.2 SD‑JWT i „minimalne tvrdnje“ za pametni ugovor

SD‑JWT‑VC omogućuje da korisnik prema RP‑u otkrije samo podskup claimova; taj RP može, nakon off‑chain verifikacije potpisa i selektivno otkrivenih atributa, u pametni ugovor poslati npr.:

- boolean (npr. `is_over_18 = true`),
- hash kombinacije atributa,
- ili neki interno definirani „proof token“.[^9][^32][^11]

Pametni ugovor tada vjeruje RP‑u kao oraklu, a ne izravno EUDI ekosustavu; ovo je sada praktičniji obrazac od punog on‑chain verifikacijskog stoga. Ako želite „jače“ jamstvo, možete kombinirati više RP‑ova/orakla ili koristiti specijalizirane ZK‑schemove nad VC‑ovima (izvan EUDI standarda, npr. custom circuite koji dokazivo implementiraju provjeru SD‑JWT‑VC‑a), ali to zasad ostaje istraživačko područje, ne dio službenog Toolboxa.[^31][^15]

### 7.3 „Nullifier“ / jedinstveni identifikator za Sybil zaštitu

Ni eIDAS 2.0 ni ARF/PID Rulebook **ne definiraju standardizirani, globalni „nullifier“** (tj. stabilni, ponovljivo izračunljiv pseudonim) za EUDI korisnika. Naprotiv, naglasak je na sprječavanju neželjenog povezivanja različitih transakcija i RP‑ova; Recitali i ARF upozoravaju da dodatni identifikatori koji omogućuju linkabilnost mogu biti problematični s gledišta GDPR‑a i čl. 5a(16) eIDAS‑a.[^31][^2][^12][^13][^6]

Za Sybil zaštitu u Web3 scenarijima to znači:

- Ne možete se osloniti na „EU‑wide PID hash“ kao standardizirani, javno vidljiv identifikator bez kršenja principa minimizacije i potencijalno regulatornih smjernica.
- Morate dizajnirati vlastiti sloj pseudonima (npr. `hash(PID, domain_salt)` ili per‑RP/per‑dApp identificatore) i paziti da se ova konstrukcija događa off‑chain, pod kontrolom korisnika ili pouzdanog orakla, a ne na javnom lancu.[^31][^15]


## 8. Preporuke za dizajn vašeg sustava

### 8.1 Arhitektura usklađena s budućim EUDI okvirom

1. **Apstrahirati sloj identiteta i vjerodajnica**  
   Uvesti interni „Identity & Credentials“ servis koji danas priča „klasični“ OIDC s Certiliom/NIAS‑om, a sutra OpenID4VP/VCI s EUDI novčanikom; aplikacije iznad njega trebaju vidjeti samo normalizirane atribute i interne korisničke ID‑ove. 

2. **Planirati podršku za dva scenarija paralelno (prijelazno razdoblje)**  
   Do kraja 2026./2027. očekivano je da će dio korisnika i dalje koristiti Certiliu/NIAS, a dio EUDI novčanik; dizajnirati tokove i korisničko sučelje tako da RP može ponuditi oba načina prijave/verifikacije, uz zajedničku tablicu korisničkih profila koja može povezati više izvora identiteta. 

3. **Konzumirati službene open‑source SDK‑ove**  
   Za integraciju s EUDI ekosustavom koristiti GitHub repozitorije `eu-digital-identity-wallet` (verifier SDK‑ovi, SD‑JWT biblioteke, referentne aplikacije) umjesto vlastite implementacije protokola, posebno za OpenID4VP/VCI i SD‑JWT‑VC.[^54][^53][^34]

4. **Modelirati PID/(Q)EAA mape eksplicitno**  
   Napraviti jasan mapping sloj između PID sheme (family_name, given_name, birth_date, nacionalni identifikatori…) i vašeg internog korisničkog modela; izbjegavati da se poslovna logika direktno naslanja na „sirove“ claimove iz ID/VP tokena. 

5. **Minimalizirati pohranu osobnih podataka**  
   Po mogućnosti u bazi čuvati samo ono što je stvarno nužno (npr. interni korisnički ID, hash OIB‑a ili pseudonim, poslovne podatke), a ostale atribute dohvaćati ad‑hoc kroz EUDI prezentacije, u skladu s načelima minimizacije i selektivnog otkrivanja.[^2][^31]

### 8.2 Prilagodba postojeće Certilia integracije

1. **Zadržati OIDC danas, dodati OpenID4VP sutra**  
   Umjesto radikalne zamjene, pametnije je zadržati postojeći OIDC tok kao „legacy“/fallback i dodati OpenID4VP endpoint(e) u istu aplikaciju, tako da RP može birati između AuthN putem klasičnog IdP‑a i VC‑prezentacije preko EUDI novčanika. 

2. **Odvojiti autentikaciju od autorizacije**  
   U postojećem kodu jasno razdvojiti „tko je korisnik“ (identitet iz NIAS/Certilia/EUDI) od „što smije raditi“ (RBAC/ABAC pravila u vašem sustavu). To će olakšati prelazak na VC‑temeljene tokove bez promjene poslovne logike autorizacije. 

3. **Uvesti internu identitetnu šemu otpornu na promjene**  
   Nemojte nikada koristiti raw PID atribute (npr. OIB) kao primarni ključ u tablicama; radije generirati vlastiti stabilni `user_id` i držati reference na različite izvore (Certilia subject, EUDI PID pseudonim, eventualno sektorski ID‑ovi). Time ćete izbjeći probleme kod future‑proof migracija. 

### 8.3 Interakcija s blockchainom / Web3

1. **Držati kriptografsku verifikaciju off‑chain**  
   Verifikaciju SD‑JWT‑VC‑a, mdoc‑a i X.509 lanaca obaviti u backend servisu; prema pametnom ugovoru slati samo minimalne, ne‑identificirajuće informacije (booleani, hash‑commitmenti, agregirani skorovi). 

2. **Pažljivo dizajnirati pseudonime**  
   Ako trebate Sybil zaštitu, definirati shemu pseudonima koja uključuje domenski salt (npr. `hash(PID, dapp_salt)`) i paziti da se ova funkcija izvršava off‑chain, a rezultat ne omogućuje jednostavnu rekonstrukciju PID‑a ni međusobno povezivanje korisnikovih aktivnosti na različitim dApp‑ovima.[^31]

3. **Odabrati algoritme koje blockchain dobro podržava**  
   Ako planirate bilo kakvu on‑chain verifikaciju, birati algoritme koje ciljano okruženje dobro podržava (npr. Ed25519 ili secp256k1 na specifičnim lancima) i provjeriti kako ih uskladiti s eIDAS/EUDI politikama; za standardni P‑256 potpis očekujte veću kompleksnost i trošak. 

4. **Pratiti rad DC4EU i EBSI‑ja**  
   DC4EU i EBSI objavljuju rezultate pilot‑projekata i kod koje možete koristiti kao referencu za dizajn „VC + ledger“ arhitektura; iako nisu normativni, daju dobar uvid u obrasce koje regulator vjerojatno neće smatrati neprihvatljivima.[^15]

### 8.4 Operativni koraci 2026.–2027.

- 2026.:
  - Pratiti nacionalni plan implementacije EUDI novčanika (Ministarstvo digitalne transformacije / NIAS) i rano se uključiti u testne okoline.
  - Eksperimentirati s EUDI referentnim verifier SDK‑om i internim PoC‑ovima za OpenID4VP/VCI.
- 2027.:
  - Uvesti EUDI‑temeljeni tok u produkciju barem za dio korisnika (npr. nove korisnike ili određene usluge koje zahtijevaju višu LoA), uz zadržavanje Certilie kao fallbacka.
  - Usuglasiti se s QTSP‑ovima o korištenju EUDI novčanika kao klijenta za kvalificirani potpis gdje je to poslovno potrebno.


## 9. Ključni službeni izvori i dokumenti

- Uredba (EU) 2024/1183 (eIDAS 2.0) – službeni tekst i sažeci: publikacija 30.4.2024., stupanje na snagu 20.5.2024.[^20][^56][^1][^2]
- Skup pet provedbenih uredbi o EUDI novčaniku (funkcionalnosti, PID, protokoli, notifikacije, certificiranje) usvojenih 28.11.2024.[^2]
- Architecture and Reference Framework (ARF), verzija 1.2.0, studeni 2023., s aneksima (PID Rulebook, mDL Rulebook).[^8][^6]
- PID Rulebook (Annex 06 ARF‑a), v1.0.0, lipanj 2023., koji opisuje PID shemu i dvostruko kodiranje (ISO 18013‑5 i SD‑JWT‑VC).[^12][^13]
- W3C VC‑JOSE‑COSE Recommendation i pripadni test suite, koji ARF registrira kao ključni standard za osiguranje VC‑ova JOSE/COSE mehanizmima.[^57][^14]
- Službeni EU i informativni portali o EUDI regulativi i rokovima (Digital Strategy, Digital Decade, stručne publikacije).[^24][^56][^23][^22][^4][^5]
- Dokumenti i web stranice LSP‑ova: EWC, POTENTIAL, NOBID, DC4EU.[^49][^45][^50][^48][^44][^51][^42][^47][^43]
- GitHub organizacija `eu-digital-identity-wallet` s referentnom implementacijom i SDK‑ovima.[^53][^54][^34]
- Hrvatski izvori: notifikacijska dokumentacija NIAS‑a, MUP/AKD informacije o eOI i Certilii, te članci o nacionalnoj implementaciji EUDI okvira.[^19][^26][^16][^17][^18][^25]

---

## References

1. [eIDAS 2.0 | Updates, Compliance, Training](https://www.european-digital-identity-regulation.com) - On 30 April 2024, Regulation (EU) 2024/1183 was published in the Official Journal of the European Un...

2. [EU Digital Identity Wallet - Wikipedia](https://en.wikipedia.org/wiki/EU_Digital_Identity_Wallet)

3. [European Union: EUDI Wallet Harmonizes Identification and Age ...](https://www.bakermckenzie.com/en/insight/publications/2026/03/european-union-eudi-wallet-harmonizes-identification-and-age-gating) - Each Member State must provide at least one EUDI Wallet until 21 November 2026. ... However, the reg...

4. [21-05-2025](https://sphereon.com/news-and-insights/the-new-eu-eidas2-regulation-timeline/) - Read more about the new EU eIDAS2.0 regulation that entered into force on May 20th, 2024,

5. [eIDAS 2.0 & the EUDI Wallet: A Practical Guide for Enterprise IAM ...](https://www.wwpass.com/blog/eidas-2-0-the-eudi-wallet-a-practical-guide-for-enterprise-iam-2025-2027/) - Learn how eIDAS 2.0 and the EUDI Wallet transform enterprise IAM with phishing-resistant authenticat...

6. [Regulation (EU) 2024/1183 – The New Framework for a European ...](https://www.ey.com/en_gr/technical/tax/tax-alerts/ey-regulation-eu-2024-1183-the-new-framework-for-a-european-digital-identity) - 5. Timeline. The Regulation enters into force on the 20th day following its publication in the Offic...

7. [ANNEX 2 - High-Level Requirements - European Digital Identity Wallet](https://eudi.dev/1.9.0/annexes/annex-2/annex-2-high-level-requirements/) - HLRs for SD-JWT VC-compliant PIDs. The requirements in the table below are valid for PIDs in the EUD...

8. [European Digital Identity...](https://eudi.dev/1.1.0/arf/)

9. [IETF SD-JWT-based Verifiable Digital Credentials (SD-JWT VC ...](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/9) - Art 5a(23) PID/EAAPID & EAA issued to EUDI WalletsPID & EAA ... EUDI WalletThe ticket is about a pro...

10. [EUDI Wallet Guide | TrustID Solutions](https://trustid-solutions.eu/en/articles/eudi-wallet-guide) - The backbone of the entire EUDI wallet ecosystem. Three main components: OID4VCI - Credential Issuan...

11. [EUDI Credential Formats Crash Course: X.509, mDL, SD-JWT VC ...](https://shanedeconinck.be/posts/eudi-credential-formats-crash-course/) - Where mdoc's strength is proximity, SD-JWT VC's strength is the web. The split between mdoc and SD-J...

12. [PID Rule Book - EUDI Wallet](https://eudi.dev/1.3.0/annexes/annex-06-pid-rulebook/) - Requirement 6 in section 5.1.2 of the ARF specifies that PID attestation must be issued in accordanc...

13. [[PDF] PID Rule Book - European Digital Identity Wallet](https://eudi.dev/1.1.0/annexes/annex-06-pid-rulebook.pdf) - This document is the Person Identification Data (PID) Rule Book. It contains requirements specific t...

14. [W3C Securing Verifiable Credentials using JOSE and COSE, Jones ...](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/389) - ... EUDI Wallet domain and the ... The Technical Specification is listed in chapter 10 of the main d...

15. [DC4EU final report proposes pluralistic trust model to realise EUDI ...](https://connect.geant.org/2026/02/03/dc4eu-final-report-proposes-pluralistic-trust-model-to-realise-eudi-wallet-vision) - DC4EU (Digital Credentials for Europe), the project testing the real-world feasibility of the Europe...

16. [HR eID scheme: Pre-notification to Cooperation Network](https://ec.europa.eu/digital-building-blocks/sites/download/attachments/62885743/21112023_Notification%20update%20for%20HR%20eID%20scheme%20NIAS.pdf?version=1&modificationDate=1709628276200&api=v2)

17. [Opinion - View Source](https://ec.europa.eu/digital-building-blocks/sites/plugins/viewsource/viewpagesrc.action?pageId=65972757)

18. [Ministry of the Interior and Agency for Commercial Activities ...](https://gov.hr/en/ministry-of-the-interior-and-agency-for-commercial-activities-invite-citizens-to-activate-their-identity-card-and-download-mobile-application-certilia/2433) - Introduction of a digital identity card reduces administrative burden, simplifies numerous procedure...

19. [Mobile identity - E-osobna - eOI](https://www.eid.hr/en/mobile-identitity) - The Mobile.ID credential allows the user to be authenticated via his/her user account and the mobile...

20. [eIDAS 2.0: The Future of EU Digital Identity - Agrello](https://www.agrello.io/en/blog/eidas-2-0-the-future-of-eu-digital-identity-and-authentication-by-2030/) - Publication and Entry into Force. Regulation (EU) 2024/1183 published and entered into force. April ...

21. [EIDAS 2.0: Compliance Deadlines and Steps for Companies](https://www.signaturit.com/blog/eidas-2-0-compliance-deadlines-and-steps-for-companies/) - Learn about eIDAS 2.0 compliance deadlines, key requirements and the steps companies must take to pr...

22. [Is Europe Ready for the EUDI Wallet? A 2025 Status Overview](https://www.identt.pl/en/blog/is-europe-ready-for-the-eudi-wallet-a-2025-status-overview/) - Explore Europe’s readiness for the EUDI Wallet, regional adoption differences, pilot projects, and t...

23. [eIDAS 2.0 — Executive Overview](https://sphereon.com/content/uploads/PDF/eIDAS-2-Executive-Overview.pdf)

24. [eIDAS 2.0 Digital Identity Wallet: Compliance 2026 - Yousign](https://yousign.com/blog/eidas-2-0-digital-identity-wallet-compliance-requirements) - Essential eIDAS 2.0 compliance guide for e-signature providers. Navigate EU Digital Identity Wallet ...

25. [Croatian digitalization head shares concerns about EUDI Wallet ...](https://www.biometricupdate.com/202510/croatian-digitalization-head-shares-concerns-about-eudi-wallet-certification) - The country's eIDAS node was established in 2017, and by 2018, Croatia's eID and NIAS were notified ...

26. [European countries prepare for EU Digital ID Wallet implementation](https://doconchain.com/blog/european-countries-prepare-for-eu-digital-id-wallet-implementation) - While significant progress is evident, four EU states—Bulgaria, Croatia, Malta, and Romania—are yet ...

27. [EUDI Wallet Credential Format Guide - iGrant.io](https://igrant.io/articles/verifiable-credential-formats-in-the-eudi-wallet-w3c-vc-dm-and-iso-18013-5-mdlmdoc.html) - ISO/IEC TS 18013-7:2024 is a technical specification that extends the capabilities of mobile driving...

28. [OpenID4VC High Assurance Interoperability Profile with SD-JWT VC](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0-00.html) - This document defines a profile of OpenID for Verifiable Credentials in combination with the credent...

29. [W3C Verifiable Credentials in Europe: From Theory to Large-Scale ...](https://www.linkedin.com/pulse/w3c-verifiable-credentials-europe-from-theory-what-has-ari%C3%B1o-martin-4xhee) - Following recent publications on DC4EU's results, we have received considerable interest and questio...

30. [EUDI Wallets with OpenID for Verifiable Credentials (OpenID4VC)](https://docs.igrant.io/concepts/openID4vc/) - Learn how OpenID4VC standards enable secure issuance and presentation of verifiable credentials in d...

31. [The impact of zero-knowledge proofs on data minimisation ...](https://policyreview.info/articles/analysis/impact-zero-knowledge-proofs) - Zero-knowledge proofs allow the implementation of the data minimisation principle imposed by the GDP...

32. [SD-JWT-Based Verifiable Credentials: An Introduction - Idura](https://idura.eu/blog/sd-jwt-based-verifiable-credentials) - SD-JWT-Based Verifiable Credentials let Holders protect their privacy by disclosing only the necessa...

33. [Architecture and reference framework - EUDI Wallet](https://eudi.dev/1.4.0/arf/) - The EUDI Wallet Instance authenticates the EUDI Wallet Provider, meaning that the EUDI Wallet ... [S...

34. [GitHub - eu-digital-identity-wallet/eudi-lib-jvm-sdjwt-kt](https://github.com/eu-digital-identity-wallet/eudi-lib-jvm-sdjwt-kt) - This is a library offering a DSL (domain-specific language) for defining how a set of claims should ...

35. [Create a Signature (*AdES) | EUDI Wallet](https://didroom.com/guides/Signature/signature.html) - EUDI Wallet. Search K. Main NavigationHome. Guide. Wallet App · Web ... EdDSA on Ed25519. RSA. RSASS...

36. [Building a Production-Ready Cryptographic Key Manager for EUDI ...](https://www.linkedin.com/pulse/building-production-ready-cryptographic-key-manager-eudi-hugo-perez-yjchf) - EdDSA (Ed25519): Elegant and fast, but uneven adoption in gov ecosystems and policy profiles. ES256 ...

37. [Is WSCD and WSCA described in European Digital Identity Wallet ...](https://www.methics.fi/is-wscd-and-wsca-equivalent-to-sscd/) - SOG-IS has been instrumental in defining the security requirements ... EUDI Wallet app on Smartphone...

38. [GlobalPlatform Part Two: EU Digital Identity Wallets – Security ...](https://globalplatform.org/part-two-eu-digital-identity-wallets-security-reach-convenience-with-secure-elements/) - This blog has been written in collaboration with the GlobalPlatform eID Wallet Task Force to share i...

39. [[PDF] EUDI Wallet - Anja Lehmann](https://hpi.de/oldsite/fileadmin/user_upload/fachgebiete/lehmann/Publications/PQC_EUDI_25.pdf) - ... SOGIS approval and device binding. □ Ongoing standardisation activities ... EUDI Wallet requires...

40. [eIDAS2 Toolbox: Selective Disclosure and ZKP in the Identity Wallet](https://www.kuppingercole.com/watch/eidas-toolbox-eic24) - In this session, Sebastian Elfors gives an introduction into what selective disclosure really means ...

41. [Cryptography - German National EUDI Wallet](https://bmi.usercontent.opencode.de/eudi-wallet/wallet-development-documentation-public/architecture-concept/05-cryptography/)

42. [SHAPING EU's DIGITAL FUTURE THROUGH EUDI WALLETS](https://identyum.com/shaping-eus-digital-future-through-eudi-wallets-summary/) - As the only ID Wallet provider from Croatia involved in this groundbreaking initiative, Identyum is ...

43. [EU Digital Identity Wallet Pilot implementation](https://digital-strategy.ec.europa.eu/en/policies/eudi-wallet-implementation) - Large scale pilot projects are testing the technical specifications for the Common Toolbox that will...

44. [Signicat part of two EU Digital Identity Wallet Large Scale Pilots](https://www.signicat.com/press-releases/signicat-part-of-two-eu-digital-identity-wallet-large-scale-pilots) - The two large scale pilots, EWC (EUDI Wallet Consortium) and NOBID (Nordic-Baltic eID Project), will...

45. [EWC has been selected by the European Commission to participate in EU Digital Identity Wallet Large Scale Pilots - IDnext](https://idnext.eu/ewc-has-been-selected-by-the-european-commission-to-participate-in-eu-digital-identity-wallet-large-scale-pilots) - The EUDI Wallet Consortium (EWC) is proud to announce that we have been selected by the European Com...

46. [EUDI Wallet Consortium: Home](https://eudiwalletconsortium.org)

47. [19 European Member States and Ukraine - Potential](https://www.digital-identity-wallet.eu/about-us/19-european-member-states-and-ukraine/) - Austria; Belgium; Cyprus; Czechia; Estonia; Finland; France; Germany; Greece; Hungary; Italy; Lithua...

48. [NOBID wraps up pilot under the EU Digital Identity Wallet Programme](https://www.nobidconsortium.com/nobid-wraps-up-successful-pilot-under-the-european-digital-identity-wallet-programme/) - After two years of intense collaboration, the NOBID project is concluding its role as one of the Lar...

49. [About - NOBID Consortium](https://www.nobidconsortium.com/about/) - The Nordic-Baltic eID Project (NOBID) has joined other countries to create a consortium for the EU D...

50. [The NOBID Consortium Chosen to Launch Pan-European Payments ...](https://www.intesigroup.com/en/news/nobid-consortium/) - NOBID works to harmonise the eID solutions in eight Nordic and Baltic countries to realize cross-bor...

51. [EUDI Wallet Consortium Archives - Identyum](https://identyum.com/tag/eudi-wallet-consortium/) - Identyum is a member of EUDI Wallet Consortium (EWC), which has been ... KYC IN CROATIA – LEGAL FRAM...

52. [Members - EUDI Wallet Consortium](https://eudiwalletconsortium.org/about-us/members/)

53. [eu-digital-identity-wallet repositories - GitHub](https://github.com/orgs/eu-digital-identity-wallet/repositories) - Official GitHub Organization of the European Digital Identity project. - European Digital Identity

54. [Roadmap of the EUDI Wallet ...](https://github.com/eu-digital-identity-wallet/eudi-wallet-reference-implementation-roadmap) - Roadmap of the EUDIW reference implementation. . Contribute to eu-digital-identity-wallet/eudi-walle...

55. [Supported standards and technologies | Procivis One Documentation](https://docs.procivis.ch/standards) - JOSE / EdDSA Ed25519; JOSE / ML-DSA-65 (FIPS 204)*. Backwards compatibility ... EUDI wallet · Swiss ...

56. [European Digital Identity (EUDI) Regulation](https://digital-strategy.ec.europa.eu/en/policies/eudi-regulation) - The European Digital Identity (EUDI) Regulation will revolutionise digital identity in the EU by ena...

57. [VC JOSE COSE Test Suite: Securing Verifiable Credentials using ...](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/473) - Formal name VC JOSE COSE Test Suite: Securing Verifiable Credentials using JOSE and COSE ... eu-digi...


