# Certilia Mobile.id — Perplexity izvještaj

**Prompt:** [01-certilia-mobileid.md](../prompts/01-certilia-mobileid.md)
**Izvor:** Perplexity Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Perplexity deep research ovdje -->

# Certilia Mobile.ID i OIDC – tehničke činjenice i nepoznanice

## 1. Potvrđene činjenice (službeni izvori)

### 1.1. Opći opis i identitet pružatelja usluge

- Certilia je mobilna i cloud platforma za elektroničku identifikaciju i kvalificirano elektroničko potpisivanje dokumenata, namijenjena hrvatskim građanima i poslovnim subjektima.[^1][^2]
- Mobilna aplikacija Certilia omogućuje prijavu na e-Građane i druge online servise s najvišom razinom sigurnosti, kao i kvalificirano elektroničko potpisivanje dokumenata sukladno Uredbi eIDAS (EU 910/2014).[^2][^3][^1]
- Pravna osoba iza Certilie je **Agencija za komercijalnu djelatnost d.o.o. (AKD)**, kvalificirani pružatelj usluga povjerenja (QTPS) sa sjedištem u Zagrebu.[^4][^5][^6]

### 1.2. Identity Provider / OIDC usluga

- Certilia nudi **Identity Provider (IDP)** / **Identity API** uslugu koja omogućuje prijavu korisnika u web i mobilne aplikacije koristeći OAuth2 / OpenID Connect standard.[^7][^8][^9]
- Službeni opis Identity API-ja navodi da tehnički proces funkcionira kao „prijava s Google ili Facebook računom“, ali uz razliku da Certilia IDP vraća **službene i provjerene podatke osobe** koja se prijavljuje (ime, prezime, OIB i ostale podatke).[^8]
- Vlasnik aplikacije može birati koje podatke traži; korisnik pri prvoj prijavi vidi koje se informacije traže i daje privolu za njihovo slanje, pri čemu Certilia ističe usklađenost s GDPR-om.[^9][^7][^8]

### 1.3. eIDAS / razina osiguranja i NIAS

- Na službenom popisu prihvaćenih vjerodajnica za NIAS (e-Građani) navodi se više vjerodajnica koje izdaje AKD, uključujući **mobile.ID osobnu vjerodajnicu** i **mobile.ID poslovnu vjerodajnicu**, klasificirane pod **visokom razinom sigurnosti**.[^10]
- Dokumenti Porezne uprave te korisničke upute za mPoreznu izrijekom navode da mPorezna podržava vjerodajnice visoke razine sigurnosti, pri čemu su kao primjeri navedeni **Mobile ID osobne iskaznice i Certilia osobni ili poslovni mobile ID**.[^11][^12]
- Službeni opis Identity provider usluge naglašava da prijava s Certilia mobileID vjerodajnicom predstavlja prijavu **visoke razine po eIDAS-u**, prikladnu i za najosjetljivije slučajeve identifikacije.[^8]
- Certilia je usklađena s eIDAS uredbom te omogućuje kvalificirano elektroničko potpisivanje s pravnim učinkom jednakim vlastoručnom potpisu na području cijele EU.[^1][^2]

### 1.4. Registracija klijenta (relying party) i trošak

- Službene stranice Certilie navode da **„svatko može besplatno registrirati svoju aplikaciju u Certilia IDP-u“** i da je usluga Identity Provider (IDP) **besplatna** za paket „Start“.[^7][^8]
- Cjenik Identity providera prikazuje:
  - Paket **Identity provider Start** – do **100.000 prijava mjesečno** u aplikaciju, po cijeni **0 EUR**.[^7][^8]
  - Paket **Identity provider Business** – **neograničen broj prijava** mjesečno, po cijeni **66,36 EUR mjesečno**.[^8][^7]
- Aktivacija/registracija aplikacije provodi se putem portala **Certilia Developer**: službeni materijali navode da se klikom na „Aktiviraj“ korisnik prijavljuje u Certilia Developer i generira ključeve za svoju aplikaciju „u samo nekoliko trenutaka“.[^13][^7][^8]
- Za ostale Certilia usluge (npr. vremenski žig, eSign API) također se koristi Certilia Developer za aktivaciju i upravljanje pristupnim podacima, što potvrđuje postojanje jedinstvenog developerskog portala.[^14][^13]

### 1.5. Povezanost s mobile.ID vjerodajnicom

- Portal **gov.hr** opisuje integraciju **sms.ID** i **mobile.ID** vjerodajnica s NIAS-om, pri čemu se navodi da se korisnik mobile.ID osobne vjerodajnice autenticira putem korisničkog računa i **mobilne aplikacije (Certilia mobileID)** instalirane na pametnom telefonu.[^15]
- Tijekom registracije na Certilia portal svaki korisnik aktivira **mobile.ID** i **sms.ID osobnu vjerodajnicu**, što potvrđuje da je Certilia ključni pružatelj usluge za mobile.ID vjerodajnice u NIAS-u.[^15]

### 1.6. Mobilna aplikacija i korisnički portal

- Službeni korisnički portal Certilie dostupan je na adresi **portal.certilia.com**, gdje se korisnici prijavljuju Certilia računom, eOI ili drugim vjerodajnicama.[^16]
- Certilia mobilna aplikacija dostupna je u Google Play i Apple App Store trgovinama; opisi naglašavaju funkcionalnosti elektroničke identifikacije i potpisivanja, te prijavu u e-Građane i druge sustave s najvišom razinom sigurnosti.[^17][^3][^2]

### 1.7. Pravna dokumentacija i uvjeti korištenja

- Certilia ima objavljene **Uvjeti korištenja** i **pravnu dokumentaciju** na domeni certilia.com; u općim uvjetima navodi se da je svako korištenje web-mjesta i usluga podložno tim uvjetima.[^18][^4]
- AKD i Certilia su kao kvalificirani pružatelj usluga povjerenja i izdavatelj certifikata uvršteni u nacionalne i EU „Trusted List“ popise kvalificiranih pružatelja usluga povjerenja, što je preduvjet za pravno valjan kvalificirani potpis u EU.[^5][^19][^20]


## 2. Djelomično potvrđeno (neslužbeni, ali pouzdani izvori / generički standardi)

Ovaj odjeljak obuhvaća informacije koje proizlaze iz:
- neslužbenih, ali relativno pouzdanih izvora (npr. LinkedIn objave stručnjaka), ili
- primjene generičkih OIDC standarda na Certiliu, **bez izričite potvrde u njihovoj javnoj dokumentaciji**.

### 2.1. Postojanje javnog OIDC sustava i self‑service registracije

- LinkedIn objava (Felix Magedanz) navodi da je Certilia „nacionalno rješenje mobilnog identiteta“ (EU LoA High) s **javno dostupnim OIDC sustavom**, te da **„svatko može registrirati OIDC klijenta besplatno na Certilia Developer (https://developer.certilia.com) i omogućiti Login with Certilia“**.[^21]
- Ova tvrdnja je u skladu s marketinškim materijalima Certilie o Identity provider usluzi i cjeniku (besplatan Start paket, jednostavna integracija), iako sam developerski portal i tehnička dokumentacija nisu javno dostupni bez prijave.[^9][^7][^8]

### 2.2. OIDC Discovery URL i JWKS URL – generički obrasci

- OpenID Connect Discovery specifikacija definira standardnu putanju `/.well-known/openid-configuration` koju svaki OIDC pružatelj može (i tipično treba) objaviti na svom **issuer** URL-u.[^22][^23]
- Standardni obrazac je da se Discovery dokument nalazi na `https://<issuer>/.well-known/openid-configuration`, iz kojeg se čitaju `authorization_endpoint`, `token_endpoint`, `jwks_uri`, podržani algoritmi (`id_token_signing_alg_values_supported`) itd.[^24][^22]
- Iako je iz OIDC standarda gotovo sigurno da Certilia ima takav Discovery dokument, **iz dostupnih javnih izvora se ne može potvrditi koja je točno vrijednost `issuer` URL-a, niti konkretan Discovery endpoint**, jer zahtijevaju pristup developerskom portalu ili mrežnom prometu tijekom prijave.

### 2.3. Podržani parametri `state` i `nonce`

- OpenID Connect Core 1.0 propisuje parametre `state` (za CSRF zaštitu i očuvanje aplikacijskog stanja) i `nonce` (za zaštitu od replay napada, posebno za implicitni/hibridni tok; obični authorization code flow ga također često koristi).[^25][^26]
- Svi produkcijski OIDC provideri za ozbiljne eIDAS LoA high scenarije u praksi podržavaju oba parametra i vraćaju vrijednost `nonce` unutar ID tokena, a `state` natrag u redirect-u.[^26][^22][^25]
- Iz činjenice da Certilia naglašava korištenje standarda OAuth2/OIDC i uspoređuje se s „Login with Google/Facebook“, može se **razumno očekivati**, ali ne i službeno potvrditi, da podržavaju proizvoljnu vrijednost `nonce` i standardni `state` parametar u authorization zahtjevu.[^9][^8]

### 2.4. JWT / ID token – tipični sadržaj kod OIDC identitetskih providera

Za Certiliu ne postoji javni primjer JWT-a ili službeni popis claimova, ali iz kombinacije službenih i neslužbenih izvora te generičkih OIDC obrazaca proizlazi sljedeće:

- Identity API eksplicitno navodi da IDP „vraća službene i provjerene podatke osobe koja se prijavljuje (ime, prezime, OIB i ostale podatke)“.[^8]
- Standardni OIDC claims za identitet uključuju `sub`, `name`, `given_name`, `family_name`, `email`, `birthdate` i dr., pri čemu je `sub` stabilan pseudonim korisnika a ostalo ovisi o scope-ovima i privoli.[^27][^25]
- Stoga je razumno pretpostaviti da Certilia, ovisno o konfiguraciji klijenta i privoli korisnika, može vraćati barem **ime, prezime i OIB** – no **nije jasno** jesu li ti podaci u ID tokenu ili (djelomično) preko `userinfo` endpointa.
- Nema javnog podatka o tome:
  - je li `sub` deterministički jedinstven per korisnik (i u kojem je formatu),
  - koliko je širok set osobnih podataka koji mogu biti uključeni u token,
  - koliki je tipični `exp` (trajanje) ID tokena,
  - koji je točan format `aud` claim-a.

### 2.5. Kriptografski algoritmi i JWKS – generička očekivanja

- OIDC providerima za LoA high/eIDAS scenarije uobičajeno je korištenje **asimetričnih algoritama** (npr. RS256 ili ES256) i objava javnih ključeva putem **JWKS endpointa** navedenog u Discovery dokumentu (`jwks_uri`).[^28][^29][^30]
- Budući da je AKD kvalificirani pružatelj usluga povjerenja s vlastitom PKI infrastrukturom, vrlo je vjerojatno da se koriste RSA ili ECDSA ključevi minimalno 2048-bitne sigurnosti, no **nema javne tehničke potvrde** za konkretan OIDC/JWT profil Certilie.[^6][^5]


## 3. Nepoznato / zahtijeva direktan kontakt s AKD‑om ili pristup Certilia Developer portalu

U nastavku su pitanja iz korisničkog upita za koja **nije pronađen nijedan vjerodostojan javni izvor** (službena dokumentacija, kod primjera, sample JWT) koji bi dao nedvosmislen odgovor. Za sva ta pitanja nužno je:
- dobiti pristup tehničkoj dokumentaciji na **Certilia Developer** portalu,
- ili direktno kontaktirati AKD/Certilia tehničku podršku (helpdesk@certilia.com),
- ili izvesti vlastiti testni OIDC flow i dekodirati ID tokene.

### 3.1. OIDC konfiguracija – nepoznati detalji

Slijede stavke koje **nisu mogle biti potvrđene** iz javnih izvora:

- Točan **OIDC Discovery URL** (npr. `https://<neki-host>/.well-known/openid-configuration`).
- Točna vrijednost **`issuer`** claim-a u Discovery dokumentu i JWT-ovima.
- Točan popis podržanih **`response_types`** i **`grant_types`** (osim generičke činjenice da koriste OAuth2/OIDC).
- Detaljan popis podržanih **`scopes`** (osim općenite napomene o OpenID Connect standardu i GDPR-privolama).
- Službena potvrda o tome **podržava li Certilia proizvoljnu vrijednost `nonce`** (npr. Ethereum adresu ili njezin hash) u authorization zahtjevu, te vraća li je neizmijenjenu u ID tokenu.
- Službena potvrda o podršci `state` parametra – iako je u standardu i gotovo sigurno prisutan, javna dokumentacija ne navodi ništa eksplicitno.
- Točan format i ograničenja za **`redirect_uri`** (npr. zahtjev za HTTPS, ograničenja na domene, podrška za mobilne URI scheme-ove itd.).

### 3.2. JWT potpis i ključevi

Slijede pitanja na koja javni izvori ne daju odgovor:

- Točan algoritam za potpisivanje ID tokena / JWT-ova: **RS256, PS256, ES256 ili drugi**.
- Veličina ključa (npr. RSA‑2048, RSA‑4096, P‑256 krivulja itd.) korištena za OIDC/JWT potpis.
- Točna JWKS endpoint adresa (`jwks_uri`) i struktura objavljenih JWK-ova.
- Broj ključeva tipično prisutnih u JWKS-u (jedan ili više; obrazac rotacije).
- Učestalost rotacije ključeva (npr. mjesečno, kvartalno) i pravila za istodobno držanje više aktivnih ključeva.
- Format `kid` (Key ID) u JWT headeru (npr. hash ključa, random GUID, verzijski string itd.).

### 3.3. JWT sadržaj (claims)

Nijedan javni izvor ne daje primjer stvarnog Certilia ID tokena, pa ostaju neodgovorena sljedeća pitanja:

- Točan sadržaj claimova u ID tokenu (osim marketinške tvrdnje da se vraćaju provjereni osobni podaci, uključujući ime, prezime i OIB).
- Je li **`sub`**:
  - hash OIB-a,
  - interni identifikator,
  - pseudonim generiran po nekoj politici,
  - i je li deterministički jedinstven po osobi i stabilan kroz vrijeme.
- Jesu li u ID tokenu prisutni standardni claims: `name`, `given_name`, `family_name`, `email`, `birthdate` i eventualno drugi osobni atributi.
- Je li **OIB** eksplicitno prisutan kao poseban claim, i ako jest, pod kojim nazivom (npr. `oib`, `personal_number`, slično) i u kojem kontekstu (ID token, userinfo endpoint ili oba).
- Koliki je **tipični `exp` (time‑to‑live) ID tokena** – npr. nekoliko minuta, 30 min, 1 sat itd.
- Koji je točan format **`aud`** claim-a (tekstualni client_id, URL, lista vrijednosti i sl.).

### 3.4. Registracija klijenta – detalji procesa i ograničenja namjene

Izvan općih marketinških tvrdnji (da je registracija jednostavna i brza) nedostaju sljedeće informacije:

- Detaljan tehnički proces registracije klijenta (relying party) u Certilia Developer portalu:
  - ručni unos podataka,
  - postoji li **dinamička registracija** (Dynamic Client Registration),
  - kako se definiraju redirect URI‑jevi, scope‑ovi i traženi atributi.
- Postoji li **testno/sandbox okruženje** odvojeno od produkcije – ne nalazi se javna referenca na „sandbox“, „test“ ili sličan endpoint za Identity API.
- Specifični **uvjeti korištenja za Identity API/OIDC** (npr. zabrana korištenja ID tokena izvan svrhe autentikacije, ograničenja pohrane osobnih podataka, zabrana on‑chain objave OIB‑a itd.) – opći uvjeti korištenja postoje, ali nema izdvojene developerske licence ugovore dostupne javno.
- Potencijalna ograničenja vezana uz **on‑chain upotrebu JWT‑ova** (poput onoga što planira korisnik – predaja cjelovitog JWT‑a smart ugovoru) – o tome nema nikakve javne naznake; odgovor je moguće dobiti samo direktno od AKD‑a.

### 3.5. eIDAS prekogranična uporaba identiteta (ne samo potpisa)

- Potpisi i pečati koje izdaje Certilia preko kvalificiranih certifikata imaju **pravnu snagu kvalificiranog elektroničkog potpisa u cijeloj EU**, što je jasno navedeno u više službenih izvora.[^31][^2][^1]
- Međutim, pitanje **je li hrvatski nacionalni eID sustav (eOI + mobile.ID / Certilia)** formalno **notificiran kao eIDAS eID shema za prekograničnu autentikaciju** na druge državne e-usluge, zahtijeva detaljno čitanje eIDAS onboarding dokumentacije i europskih popisa notificiranih shema; dostupni javni sažeci ne daju jednostavan, jednoznačan odgovor za konkretno stanje u 2026.[^32][^33][^34]
- Za precizan status (trenutno notificirana/pre‑notificirana/ne‑notificirana shema za Hrvatsku) potrebno je konzultirati ažurirane materijale Europske komisije ili nacionalnog eIDAS tijela.

### 3.6. Tehnička infrastruktura / softverska platforma

- Nema javno dostupne informacije koji se **konkretni OIDC softver** koristi ispod haube (npr. Keycloak, custom rješenje, komercijalni IdP).
- Nema objavljenih **SDK‑ova, klijentskih biblioteka** niti javne „developer docs“ stranice koja bi opisivala endpoint-e, primjere konfiguracije ili primjerice integraciju s tipičnim frameworkovima – sve to je, čini se, iza Certilia Developer portala.


## 4. Relevantni linkovi (službene stranice i ključna dokumentacija)

### 4.1. Certilia – Identity / OIDC i integracija

- Certilia Identity API / Identity provider (IDP) – opis usluge, podaci koje vraća (ime, prezime, OIB), cjenik:
  - https://www.certilia.com/identity-api/[^8]
- Identity provider (IDP) – poslovni korisnici (vrlo sličan sadržaj uz naglasak na NIAS/eIDAS LoA high):
  - https://www.certilia.com/poslovni-korisnici/identity-provider[^7]
- Opći opis integracije Certilia proizvoda i usluga, spominjanje OAuth 2.0 i OpenID Connect standarda:
  - https://www.certilia.com/integriraj-certilia-proizvode-i-usluge-u-vlastiti-sustav/[^9]
- Certilia Developer (spominje se u više službenih stranica – aktivacija usluga):
  - linkovi s certilia.com koji vode na developerski portal (prijava/aktivacija), i neslužbeno: https://developer.certilia.com[^13][^21][^8]

### 4.2. Certilia – pravna dokumentacija i uvjeti

- Pravna dokumentacija (CP, CPS, uvjeti za certifikate i usluge povjerenja):
  - https://www.certilia.com/dokumenti/[^4]
- Opći uvjeti korištenja web-mjesta i usluga:
  - https://www.certilia.com/uvjeti-koristenja[^18]

### 4.3. Mobilni identitet, NIAS i razina osiguranja

- Popis prihvaćenih vjerodajnica za NIAS (e-Građani) – uključuje mobile.ID osobnu i poslovnu vjerodajnicu AKD-a, razina sigurnosti „visoka“:
  - https://gov.hr/hr/lista-prihvacenih-vjerodajnica/1792[^10]
- Integrirane sms.ID i mobile.ID vjerodajnice (opis kako mobile.ID koristi Certilia mobileID aplikaciju):
  - https://gov.hr/hr/integrirane-nove-sms-id-i-mobile-id-osobne-i-poslovne-vjerodajnice/2286[^15]
- Portal eID.hr – mobilni identitet i opis certilijskog mobilnog identiteta:
  - https://www.eid.hr/hr/mobilni-identitet[^35]
- e‑Porezna / mPorezna – dokumenti koji navode Certilia mobile ID kao vjerodajnicu visoke razine sigurnosti:
  - Primjer uputa: https://e-porezna.porezna-uprava.hr/Upute/G2B/Korisnicke_upute_mPorezna.pdf[^12]

### 4.4. AKD i eIDAS / usluge povjerenja

- AKD – kvalificirani pružatelj usluga povjerenja, pregled rješenja (uključujući ePotpis):
  - https://www.akd.hr/hr/proizvodi-i-rjesenja/rjesenja/epotpis[^31]
- AKD kao kvalificirani pružatelj usluga povjerenja (povezano s osobnim certifikatima i eOI):
  - npr. HRIDCA Certification Practice Statement: https://www.eid.hr/datastore/file/65/EN-HRIDCA-Certification-Practice-Statement-2.5.pdf[^36]
- EU Trusted Lists – popis kvalificiranih pružatelja usluga povjerenja u EU (AKD i srodne usluge):
  - https://digital-strategy.ec.europa.eu/en/policies/eu-trusted-lists[^20]

### 4.5. Marketinški i informativni materijali o Certiliji

- Glavna stranica Certilie (opis proizvoda, naglasak na visoku sigurnost, eIDAS, ISO 27001, Common Criteria):
  - https://www.certilia.com/certilia/[^1]
- Članak Bug.hr o Certilia aplikaciji i prijavi na e-Građane:
  - https://www.bug.hr/appdana/certilia--e-potpisivanje-dokumenata-i-pristup-e-gradjanima-22632[^2]
- Članak Story.hr o Certiliji (opći opis funkcionalnosti, eIDAS i kvalificirani potpis):
  - https://story.hr/Smartlife/a335279/SVI-BISMO-JU-TREBALI-IMATI-Znate-li-sto-je-Certilia.html[^37][^3]

### 4.6. Generički OIDC i JWKS resursi (za razumijevanje standarda)

Iako nisu specifični za Certiliu, ovi linkovi su korisni za razumijevanje OIDC Discovery, JWKS-a i validacije JWT-ova:

- OpenID Connect Discovery 1.0 specifikacija:
  - https://openid.net/specs/openid-connect-discovery-1_0.html[^23]
- Objašnjenje `/.well-known/openid-configuration` i kako ga konzumirati:
  - https://www.authgear.com/post/well-known-openid-configuration[^22]
- Uvod u JWKS i JWKS URI:
  - https://www.authgear.com/post/what-is-jwks[^29]
- Upute za korištenje JWKS endpointa kod raznih IdP-ova (primjeri):
  - npr. https://auth0.com/docs/secure/tokens/json-web-tokens/locate-json-web-key-sets[^30]

---

## References

1. [Certilia ENG](https://www.certilia.com/certilia/) - Independent professional Independent professional U samo dva koraka ostvarite visoku razinu identifi...

2. [Certilia – e-potpisivanje dokumenata i pristup e-Građanima](https://www.bug.hr/appdana/certilia--e-potpisivanje-dokumenata-i-pristup-e-gradjanima-22632) - Nova domaća besplatna aplikacija u kombinaciji s e-osobnom iskaznicom prve ili druge generacije omog...

3. [SVI BISMO JU TREBALI IMATI: Znate li što je Certilia? | Story](https://vjencanja.story.hr/Smartlife/a335279/SVI-BISMO-JU-TREBALI-IMATI-Znate-li-sto-je-Certilia.html) - Uz ovu aplikaciju moći ćete potpisivati dokumente online gdje god da se nalazite

4. [Pravna dokumentacija - Certilia](https://www.certilia.com/dokumenti/) - AKD d.o.o je pravna osoba koja izdaje certifikate za elektronički potpis i elektronički pečat, koja ...

5. [Zatražite AKD-ovu vjerodajnicu za e-Komunikaciju sa sudovima](https://www.akd.hr/hr/o-nama/novosti/zatrazite-akd-ovu-vjerodajnicu-za-e-komunikaciju-sa-sudovima) - Spomenute vjerodajnice razine 4 tvrtkama izdaje AKD u obliku poslovne kartice s čipom na kojem su po...

6. [[PDF] AKD PKI CERTIFICATE POLICY - European Commission](https://ec.europa.eu/digital-building-blocks/sites/download/attachments/62885743/T5.v1.0_AKD%20PKI%20Certificate%20Policy.pdf?version=1&modificationDate=1531761005303&api=v2) - Agencija za komercijalnu djelatnost d.o.o (hereinafter: AKD) is a legal person, acting as a trust se...

7. [Identity provider (IDP) - Certilia](https://www.certilia.com/poslovni-korisnici/identity-provider) - Usluga koristi OAuth2 / OpenID Connect standard, te je iznimno jednostavna za integraciju i prikladn...

8. [Identity API](https://www.certilia.com/identity-api/) - Identity provider (IDP) Certilia Identity Provider je besplatna usluga koja omogućava prijavu u Vaše...

9. [Integriraj Certilia proizvode i usluge u vlastiti sustav](https://www.certilia.com/integriraj-certilia-proizvode-i-usluge-u-vlastiti-sustav/) - Korištenjem modernih tehničkih standarda poput OAuth 2.0 i OpenID Connect, koje omogućavaju jednosta...

10. [Lista prihvaćenih vjerodajnica - gov.hr - e-Građani](https://gov.hr/hr/lista-prihvacenih-vjerodajnica/1792)

11. [Porezna uprava](https://e-porezna.porezna-uprava.hr/Upute/G2B/Korisnicke_upute_Bez_racuna_se_ne_racuna.pdf)

12. [[PDF] Korisnicke_upute_mPorezna.pdf - ePorezna - Porezna uprava](https://e-porezna.porezna-uprava.hr/Upute/G2B/Korisnicke_upute_mPorezna.pdf)

13. [Kvalificirani vremenski žig - Certilia](https://www.certilia.com/poslovni-korisnici/vremenski-zig) - Aplikacije šalju pozive standardnim RFC 3161 protokolom za vremensku ovjeru. Prijavite se u Certilia...

14. [eSign API - Certilia](https://www.certilia.com/poslovni-korisnici/esign-api) - Dokumenti koje potpišu vaši korisnici u cijeloj EU imaju istu pravnu snagu kao vlastoručno potpisani...

15. [Integrirane nove sms.ID i mobile.ID osobne i poslovne vjerodajnice](https://gov.hr/hr/integrirane-nove-sms-id-i-mobile-id-osobne-i-poslovne-vjerodajnice/2286) - sms.ID/ mobile.ID osobna vjerodajnica omogućuje korisnicima prijavu na portal e-Građani putem NIAS-a...

16. [Certilia](https://portal.certilia.com)

17. [Certilia na usluzi App Store](https://apps.apple.com/hr/app/certilia/id1577545855?l=hr) - Certilia mobile vam omogućava elektroničku identifikaciju i potpisivanje dokumenata sa pravnom snago...

18. [Uvjeti korištenja - Certilia](https://www.certilia.com/uvjeti-koristenja) - Često postavljana pitanjaPreuzimanjaCertilia Developer. Kontakt. E-mail: helpdesk@certilia.com. Tele...

19. [Popis kvalificiranih pružatelja usluga povjerenja u EU-u](https://digital-strategy.ec.europa.eu/hr/policies/eu-trusted-lists) - Uredbe eIDAS države članice EU-a obvezuju se na uspostavu, održavanje i objavu pouzdanih popisa. Ti ...

20. [List of qualified trust service providers in the EU](https://digital-strategy.ec.europa.eu/en/policies/eu-trusted-lists) - EU countries have the obligation to establish, maintain and publish trusted lists of qualified trust...

21. [Felix Magedanz's Post - LinkedIn](https://www.linkedin.com/posts/felixmagedanz_it-doesnt-always-have-to-be-sign-in-with-activity-7440294761975967744-NBHk) - Anyone can register an OIDC client for free at Certilia Developer (https://developer.certilia.com) a...

22. [What Is .well-known/openid-configuration? A Developer's Guide](https://www.authgear.com/post/well-known-openid-configuration) - Understand .well-known/openid-configuration: what it is, what every field means, and how to fetch an...

23. [OpenID Connect Discovery 1.0 - draft 12](https://openid.net/specs/openid-connect-discovery-1_0-12.html) - OpenID Connect Discovery 1.0 - draft 12

24. [Get OpenID Connect Well-Known Configuration | Akana OAuth API](https://help.akana.com/content/current/cm/api_oauth/oauth_discovery/m_oauth_getOpenIdConnectWellknownConfiguration.htm)

25. [How OpenID Connect Works - OpenID Foundation](https://openid.net/developers/how-connect-works/) - What is OpenID Connect OpenID Connect is an interoperable authentication protocol based on the OAuth...

26. [OpenID Connect (OIDC) | developer.overheid.nl](https://developer.overheid.nl/kennisbank/security/standaarden/oidc) - OpenID Connect (OIDC) is een authenticatielaag bovenop OAuth 2.0. Waar OAuth toegang regelt tot API’...

27. [OpenID Connect 1.0 Claims | Integration - Authelia](https://www.authelia.com/integration/openid-connect/openid-connect-1.0-claims/) - An introduction into utilizing the Authelia OpenID Connect 1.0 Claims functionality

28. [Authelia | NetFoundry Documentation](https://netfoundry.io/docs/openziti/guides/external-auth/identity-providers/authelia/) - Configure Authelia for with OpenZiti

29. [What Is JWKS? JSON Web Key Set and JWKS URI Explained](https://www.authgear.com/post/what-is-jwks) - Learn what JWKS is, how JWKS URI works, JWK format examples, and practical tips to generate and mana...

30. [Locate JSON Web Key Sets - Auth0 Docs](https://auth0.com/docs/secure/tokens/json-web-tokens/locate-json-web-key-sets) - Describes how to use the JSON Web Keys (JWKs) discovered using the JSON Web Key Set (JWKS) endpoint ...

31. [ePotpis – akd.hr](https://www.akd.hr/hr/proizvodi-i-rjesenja/rjesenja/epotpis) - Kvalificirani elektronički potpis, sukladno eIDAS uredbi Europskog parlamenta i Vijeća (EU 910/2014)...

32. [Overview of pre-notified and notified eID schemes under eIDAS](https://ec.europa.eu/digital-building-blocks/sites/display/EIDCOMMUNITY/Overview+of+pre-notified+and+notified+eID+schemes+under+eIDAS) - Croatia · Cyprus - Cyprus National eID · Czech Republic · Czech Republic ... 2022). Estonian eID sch...

33. [Information about login via eIDAS Node - Slovensko.sk](https://www.slovensko.sk/en/eidas/information-about-login-via-ei) - Estonia - eID, residence card, mobile ID, diplomatic card, digital card (level of assurance "high"),...

34. [Overview of pre-notified and notified eID schemes under eIDAS ...](https://ec.europa.eu/digital-building-blocks/sites/pages/viewpage.action?pageId=65972749&navigatingVersions=true) - Overview of pre-notified and notified eID schemes ... 2022). Estonian eID scheme: Mobile-ID (since 0...

35. [Mobilni identitet - E-osobna - eOI](https://www.eid.hr/hr/mobilni-identitet) - Putem linka Mobile.ID korisnik može pristupiti sučelju za registraciju u sustav Certilia koji mu aut...

36. [[PDF] HRIDCA Certification Practice Statement - eOI](https://www.eid.hr/datastore/file/65/EN-HRIDCA-Certification-Practice-Statement-2.5.pdf) - Both certificates are issued by the Agencija za komercijalnu djelatnost d.o.o. (hereinafter: AKD), w...

37. [SVI BISMO JU TREBALI IMATI: Znate li što je Certilia? - Story](https://story.hr/Smartlife/a335279/SVI-BISMO-JU-TREBALI-IMATI-Znate-li-sto-je-Certilia.html) - Uz ovu aplikaciju moći ćete potpisivati dokumente online gdje god da se nalazite

