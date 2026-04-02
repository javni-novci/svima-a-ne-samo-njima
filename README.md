# javni-novci / svima-a-ne-samo-njima

**Povezivanje hrvatskog e-Građani identiteta (Certilia) s Web3 infrastrukturom.**
*Gradimo otvoreni temelj za potpuno transparentno i sigurno upravljanje zajedničkim sredstvima.*

## Vizija

Počinjemo s jednostavnim problemom: Kako napraviti potpuno transparentan trezor za digitalne kreatore u kojem su sredstva sigurna, a isplate matematički pravedne?

No, naš **ultimativni cilj** je puno veći. Želimo stvoriti otvorenu infrastrukturu koja dokazuje da je **elektroničko lančano glasanje** i **potpuno transparentno upravljanje državnim proračunom** tehnički moguće već danas.

Ako možemo sigurno upravljati novcem podcast kreatora pomoću Certilije, sutra istom tehnologijom možemo upravljati novcem svih građana. **Svima, a ne samo njima.**

## Problem: Web3 anonimnost nasuprot pravnom identitetu

Blockchain rješava problem transparentnosti, ali pati od *Sybil* napada (jedna osoba može napraviti 100 novčanika). S druge strane, državni sustavi (poput AKD Certilia Mobile.id) imaju savršen pravni identitet, ali su zatvoreni (silosi) i oslanjaju se na slijepo povjerenje u centralni poslužitelj.

**Naš izazov:** Kako blockchain pametnom ugovoru (Safe trezor) dokazati da iza određene Ethereum adrese stoji stvarna, verificirana osoba iz Hrvatske (eIDAS usklađenost), **bez da ikada zapišemo njene osobne podatke na blockchain** i **bez da naš poslužitelj ima pristup njihovom novčaniku**?

## Arhitektura (dokaz koncepta)

Odbacujemo klasične Web2 arhitekture gdje poslužitelj drži sve ključeve. Gradimo sustav bez povjerenja na **Gnosis lancu**, koristeći eure (MiCA usklađeni **EURe**) i **neprenosive tokene identiteta (SBT)**.

### Tok korak po korak

1. **OIDC verifikacija kroz stražnji kanal:** Korisnik se prijavljuje preko klijenta putem `Certilia Mobile.id`. OIDC zahtjev kao `nonce` parametar šalje korisnikovu Web3 adresu.
2. **Poslužitelj kao poštar:** Naš Node.js poslužitelj sigurno komunicira s Certilijom, zaprima kriptografski potpisan **JWT** od AKD-a i samo ga prosljeđuje nazad klijentu. Poslužitelj *nema* Web3 privatne ključeve.
3. **Samokovanje SBT-a:** Klijent u svom pregledniku (novčanik pod vlastitom skrbi) uzima taj JWT i poziva funkciju na našem pametnom ugovoru na Gnosis mreži.
4. **Lančana JWT verifikacija:** Pametni ugovor na Gnosisu (zbog jeftinog goriva) samostalno dekodira JWT, provjerava javni ključ AKD-a i verificira da je `nonce` jednak adresi pošiljatelja.
5. **Izdavanje:** Ako je sve ispravno, pametni ugovor kuje **CERTILIA neprenosivi token identiteta (SBT)** na korisnikov novčanik.
6. **Safe integracija:** Safe trezor (kroz Zodiac module ili stražara) prepoznaje SBT. Samo novčanici s važećim CERTILIA SBT-om mogu inicirati isplate ili sudjelovati u glasanju (DAO).

### Dijagram toka

```
Korisnik (Preglednik + Novčanik pod vlastitom skrbi)
    |
    |  1. Prijava putem Certilia Mobile.id
    |     (nonce = korisnikova Web3 adresa)
    v
+---------------------+
|   Node.js Poslužitelj|  <-- 2. OIDC stražnji kanal
|   (Poštar)           |      Zaprima potpisani JWT od AKD-a
+---------------------+      Prosljeđuje ga klijentu
    |                         Nema pristup privatnim ključevima!
    |  3. JWT vraćen klijentu
    v
Korisnik (Preglednik)
    |
    |  4. Poziva pametni ugovor s JWT-om
    v
+---------------------------+
|  Gnosis lanac             |
|  Pametni ugovor           |
|  - Dekodira JWT           |
|  - Provjerava AKD potpis  |  <-- 5. Lančana verifikacija
|  - nonce == msg.sender?   |
|  - Kuje CERTILIA SBT      |
+---------------------------+
    |
    |  6. SBT izdan
    v
+---------------------------+
|  Safe trezor (DAO)        |
|  - Zodiac modul uloga     |
|  - Samo SBT nositelji     |  <-- Glasanje, isplate
|    mogu sudjelovati       |
+---------------------------+
```

## Sigurnost i privatnost (privatnost od temelja)

* **Nema jedne točke kvara:** Ako netko hakira naš Node.js poslužitelj, ne može ukrasti sredstva iz trezora jer poslužitelj nema pristup privatnim ključevima niti može lažirati Certilia (AKD) potpis.
* **Minimalni podaci na lancu:** Na lanac idu samo kriptografski potpis i sažetak — osobni podaci (ime, OIB) nikada ne napuštaju JWT koji se ne sprema. Dugoročno prelazimo na ZK nullifier pristup za potpunu privatnost (vidi [privatnost.md](docs/privatnost.md)).
* **Opozivost i oporavak:** U slučaju gubitka mobitela ili ključeva, Safe može opozvati SBT i omogućiti ponovno kovanje na novoj adresi.

## Od dokaza koncepta do državnog proračuna

Ovaj repozitorij trenutno razvija infrastrukturu za tržište. Ali zamislite budućnost:

* **Glasanje:** Svaki građanin sa CERTILIA SBT-om ima pravo na točno jedan lančani glas. Nemoguće je lažirati izbore.
* **Proračun:** Gnosis Safe trezor postaje digitalna državna riznica. Svaki isplaćeni EURe je javno vidljiv, provjerljiv i isplaćen isključivo verificiranim izvođačima.

## Dokumentacija

- **[Arhitektonske odluke (ADR)](docs/architecture.md)** — trezor, identitet, izbor mreže, specifikacije pametnih ugovora
- **[Privatnost lančanih podataka](docs/privatnost.md)** — GDPR zahtijeva nullifier od faze 1, ZK opcije, EDPB 02/2025
- **[Model prijetnji](docs/model-prijetnji.md)** — 18 napada (T1-T18), stabla napada, matrica rizika
- **[Ekonomski model](docs/ekonomski-model.md)** — tok sredstava, održivost platitelja, poslovni model
- **[Analiza postojećih rješenja](docs/analiza-postojecih.md)** — World ID, Polygon ID, Gitcoin Passport, Estonija
- **[Putokaz skaliranja](docs/putokaz.md)** — faza 0 (preduvjeti) + 1a/1b/2/3 s GDPR usklađenošću
- **[Dubinska istraživanja](research/README.md)** — 7 tema, 14 izvještaja, 7 križnih provjera (prosječna podudarnost 85/100)
- **[Pravna analiza](docs/pravna-analiza.md)** — eIDAS, GDPR, MiCA, elektroničko glasanje

## Kako doprinijeti (poziv na raspravu)

Ovo je radni dokument (v0.5 — ažurirano na temelju 7 dubinskih istraživanja s križnom provjerom). Prije nego napišemo ijednu liniju koda, želimo razbiti ovu arhitekturu na komade i pronaći sve potencijalne mane.

**Otvorene teme za raspravu u GitHub pitanjima:**

1. Koji algoritam koristi Certilia za JWT potpis i podržava li prilagođeni `nonce`?
2. Koji je najoptimalniji način za lančanu RSA/P-256 verifikaciju na Gnosis mreži?
3. Kako najelegantnije riješiti iskustvo bez goriva preko ERC-4337 ili Gelato posrednika?
4. Kako strukturirati Zodiac modul uloga za automatsku raspodjelu sredstava kreatorima?

Forkajte, otvarajte zahtjeve za izmjenama, pišite pitanja. Idemo napraviti nešto veliko.

## Licenca

[MIT](LICENSE)
