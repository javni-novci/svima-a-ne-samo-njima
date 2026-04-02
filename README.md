# javni-novci / svima-a-ne-samo-njima

**Povezivanje hrvatskog e-Gradani identiteta (Certilia) s Web3 infrastrukturom.**
*Gradimo open-source temelj za potpuno transparentno i sigurno upravljanje zajednickim sredstvima.*

## Vizija

Pocinjemo s jednostavnim problemom: Kako napraviti potpuno transparentan trezor (treasury) za digitalne kreatore u kojem su sredstva sigurna, a isplate matematicki pravedne?

No, nas **ultimativni cilj** je puno veci. Zelimo stvoriti open-source infrastrukturu koja dokazuje da je **elektronicko on-chain glasanje** i **potpuno transparentno upravljanje drzavnim proracunom** tehnicki moguce vec danas.

Ako mozemo sigurno upravljati novcem podcast kreatora pomocu Certilije, sutra istom tehnologijom mozemo upravljati novcem svih gradana. **Svima, a ne samo njima.**

## Problem: Web3 Anonimnost vs. Pravni Identitet

Blockchain rjesava problem transparentnosti, ali pati od *Sybil* napada (jedna osoba moze napraviti 100 novcanika). S druge strane, drzavni sustavi (poput AKD Certilia Mobile.id) imaju savrsen pravni identitet, ali su zatvoreni (silosi) i oslanjaju se na slijepo povjerenje u centralni server.

**Nas izazov:** Kako blockchain pametnom ugovoru (Safe Treasury) dokazati da iza odredene Ethereum adrese stoji stvarna, verificirana osoba iz Hrvatske (eIDAS uskadenost), **bez da ikada zapisemo njene osobne podatke na blockchain** i **bez da nas backend server ima pristup njihovom novcaniku**?

## Arhitektura (Proof of Concept)

Odbacujemo klasicne Web2 arhitekture gdje server drzi sve kljuceve. Gradimo "Trustless" sustav na **Gnosis Chainu**, koristeci eure (MiCA uskladeni **EURe**) i **Soulbound Tokene (SBT)**.

### Korak po korak tok (Flow)

1. **OIDC Back-channel Verifikacija:** Korisnik se prijavljuje preko klijenta putem `Certilia Mobile.id`. Vas OIDC zahtjev kao `nonce` parametar salje korisnikovu Web3 adresu.
2. **Server kao Postar:** Nas Node.js server sigurno komunicira s Certilijom, zaprima kriptografski potpisan **JWT** (JSON Web Token) od AKD-a i samo ga prosljeduje nazad klijentu. Server *nema* Web3 privatne kljuceve.
3. **Self-Minting SBT-a:** Klijent u svom pregledniku (Self-custody wallet) uzima taj JWT i poziva funkciju na nasem pametnom ugovoru na Gnosis mrezi.
4. **On-Chain JWT Verifikacija:** Pametni ugovor na Gnosisu (zbog jeftinog gasa) samostalno dekodira JWT, provjerava javni kljuc AKD-a i verificira da je `nonce` jednak adresi posiljatelja.
5. **Izdavanje:** Ako je sve ispravno, pametni ugovor minta **CERTILIA Soulbound Token (SBT)** na korisnikov novcanik.
6. **Safe integracija:** Safe Treasury (kroz Zodiac module ili Guard) prepoznaje SBT. Samo novcanici s vazecim CERTILIA SBT-om mogu inicirati isplate ili sudjelovati u glasanju (DAO).

### Dijagram toka

```
Korisnik (Preglednik + Self-custody Wallet)
    |
    |  1. Prijava putem Certilia Mobile.id
    |     (nonce = korisnikova Web3 adresa)
    v
+-------------------+
|   Node.js Server  |  <-- 2. OIDC Back-channel
|   (Postar)        |      Zaprima potpisani JWT od AKD-a
+-------------------+      Prosljeduje ga klijentu
    |                       Nema pristup privatnim kljucevima!
    |  3. JWT vracen klijentu
    v
Korisnik (Preglednik)
    |
    |  4. Poziva pametni ugovor s JWT-om
    v
+---------------------------+
|  Gnosis Chain             |
|  Pametni Ugovor           |
|  - Dekodira JWT           |
|  - Provjerava AKD potpis  |  <-- 5. On-chain verifikacija
|  - nonce == msg.sender?   |
|  - Minta CERTILIA SBT     |
+---------------------------+
    |
    |  6. SBT izdan
    v
+---------------------------+
|  Safe Treasury (DAO)      |
|  - Zodiac Role Module     |
|  - Samo SBT nositelji     |  <-- Glasanje, isplate
|    mogu sudjelovati       |
+---------------------------+
```

## Sigurnost i Privatnost (Privacy by Design)

* **Nema Single Point of Failure:** Ako netko hakira nas Node.js server, ne moze ukrasti sredstva iz trezora jer server nema pristup privatnim kljucevima niti moze lazirati Certilia (AKD) potpis.
* **Nema osobnih podataka on-chain:** SBT sluzi samo kao kriptografska "znacka" (Zero-Knowledge pristup). Ime, prezime i OIB ostaju privatni.
* **Opozivost (Revocability):** U slucaju gubitka mobitela ili isteka osobne iskaznice, DAO/Safe moze opozvati SBT on-chain.

## Od PoC-a do Drzavnog Proracuna

Ovaj repozitorij trenutno razvija infrastrukturu za marketplace. Ali zamislite buducnost:

* **Glasanje:** Svaki gradanin sa CERTILIA SBT-om ima pravo na tocno jedan on-chain glas. Nemoguce je lazirati izbore.
* **Proracun:** Gnosis Safe trezor postaje digitalna drzavna riznica. Svaki isplaceni EURe je javno vidljiv, provjerljiv i isplacen iskljucivo verificiranim izvodacima.

## Dokumentacija

- **[Architecture Decision Record & Threat Model](docs/architecture.md)** — detaljan tehniicki pregled odluka, odbacenih alternativa, analize prijetnji i specifikacija pametnih ugovora.

## Kako doprinijeti (Call for Brainstorming)

Ovo je radni dokument (Whitepaper v0.2). Prije nego napisemo ijednu liniju koda, zelimo razbiti ovu arhitekturu na komade i pronaci sve potencijalne mane.

**Otvorene teme za raspravu u GitHub Issues:**

1. Koji algoritam koristi Certilia za JWT potpis i podrzava li custom `nonce`?
2. Koji je najoptimalniji nacin za on-chain RSA/P-256 verifikaciju na Gnosis mrezi?
3. Kako najelegantnije rijesiti "Gasless" iskustvo preko ERC-4337 ili Gelato Relayera?
4. Kako strukturirati Zodiac Role Module za automatsku raspodjelu sredstava kreatorima?

Forkajte, otvarajte PR-ove, pisite Issues. Idemo napraviti nesto veliko.

## Licenca

[MIT](LICENSE)
