# Pravna analiza

**Status:** Nacrt — zahtijeva pregled pravnog stručnjaka
**Prioritet:** Visok — regulatorni rizici mogu blokirati projekt

**NAPOMENA:** Ovaj dokument nije pravni savjet. Služi kao polazna točka za razgovor s pravnim stručnjacima specijaliziranim za EU digitalnu regulativu.

---

## 1. eIDAS regulativa i korištenje Certilije

### Pitanje

Imamo li pravo koristiti JWT koji Certilia izdaje izvan njene predviđene namjene? Certilia je dizajnirana za autentikaciju na e-usluge (e-Građani), ne za kovanje tokena na blockchainu.

### Analiza

| Aspekt | Situacija | Rizik |
|--------|-----------|-------|
| Certilia uvjeti korištenja | Nepoznato — trebamo detaljno pročitati uvjete AKD-a | Visok ako uvjeti eksplicitno zabranjuju korištenje JWT-a izvan predviđenih e-usluga |
| eIDAS regulativa (910/2014) | eIDAS definira razine osiguranja identiteta (niska, značajna, visoka). Certilia je na razini "značajna". | Naš sustav ne smije tvrditi višu razinu osiguranja od one koju Certilia pruža |
| Prosljeđivanje JWT-a klijentu | Standardna OIDC praksa — klijent dobiva JWT. Ali naša namjena (slanje na blockchain) je nestandardna. | Srednji — AKD bi mogao prigovoriti nestandardnoj upotrebi |

### Otvorena pitanja

| # | Pitanje | Koga pitati |
|---|---------|-------------|
| L1 | Zabranjuju li Certilia uvjeti korištenja prosljeđivanje JWT-a trećim stranama (uključujući pametne ugovore)? | AKD pravni odjel |
| L2 | Trebamo li posebno odobrenje AKD-a za korištenje Certilije u Web3 kontekstu? | AKD |
| L3 | Možemo li se registrirati kao "pouzdajuća strana" (relying party) kod AKD-a? | AKD |
| L4 | Mijenja li eIDAS 2.0 (EUDI novčanik) pravila igre za nas? | EU regulatorni stručnjak |

---

## 2. GDPR i osobni podaci na lancu (AŽURIRANO v0.5 — istraživanje 06)

### Pravni zaključak (potvrđen dvama neovisnim izvorima, 93/100 podudarnost)

**`keccak256(OIB)` pohranjen na javnom blockchainu JEST osobni podatak prema GDPR-u.** Ovo je nedvosmisleno.

Temeljni izvori:
- **EDPB Smjernice 02/2025 o blockchainu** (8.4.2025.): "hash će se također smatrati osobnim podatkom"
- **EDPB Smjernice 01/2025 o pseudonimizaciji** (16.1.2025.): pseudonimizirani podaci = osobni podaci
- **CJEU Breyer (C-582/14)** (19.10.2016.): relativni test identifikabilnosti
- **WP29 WP216** (10.4.2014.): hashing nije anonimizacija
- **CNIL blockchain smjernice** (24.9.2018.): keyed hash s uništavanjem ključa kao "bliže brisanju"
- **ICO smjernice** (28.3.2025.): "motivated intruder" test

OIB ima prostor od ~10^11 vrijednosti — brute-force izračun svih hasheva traje **minutu do sati** na modernom hardveru.

### Implikacije za naš sustav

| Podatak | Na lancu? | Osobni podatak? | GDPR rizik |
|---------|-----------|-----------------|------------|
| `identityClaimed[hash(sub)]` | Da (pohrana ugovora) | Da — pseudonimni identifikator | Visok |
| JWT calldata (ako šaljemo cijeli JWT) | Da (calldata) | Da — ime, e-pošta, identifikator | Kritičan |
| Adresa novčanika | Da | Moguće — ako se poveže s identitetom | Srednji |
| SBT (samo prisutnost, bez metapodataka) | Da | Ne sam po sebi | Nizak |

### Ključni GDPR zahtjevi

| Zahtjev | Naš sustav | Problem |
|---------|------------|---------|
| **Pravo na brisanje** (Članak 17) | Podaci na blockchainu su nepromjenjivi | **Direktan konflikt.** Ne možemo obrisati `identityClaimed[hash(sub)]` iz ugovora. |
| **Minimizacija podataka** (Članak 5(1)(c)) | Trebamo hash(sub) za Sybil zaštitu | Moramo dokazati da je hash minimalan neophodan podatak. |
| **Pravna osnova obrade** (Članak 6) | Privola (korisnik svjesno kuje SBT) | Prihvatljivo ako je privola informirana i slobodna. |
| **Prijenos podataka izvan EU** (Članak 44-49) | Gnosis lanac ima validatore globalno | Upitno — jesu li validatori "primatelji" podataka? |

### Moguća rješenja

**Nullifier pristup (Opcija B iz privatnost.md) rješava većinu GDPR problema:**
- Na lancu nema pseudonimnog identifikatora — nullifier nije izveden iz OIB-a na obrnuti način.
- ZK dokaz ne sadrži osobne podatke.
- "Pravo na brisanje" postaje manje relevantno jer nema osobnih podataka za brisati.

**Do tada (faza 1), minimalne mjere:**
- Koristiti soljeni hash: `hash(sub + sol)` gdje je sol poznata samo korisniku i čuva se u njegovom novčaniku.
- Dokumentirati DPIA (Data Protection Impact Assessment) za lančanu obradu.
- Pribaviti informiranu privolu korisnika prije kovanja SBT-a.

---

## 3. MiCA regulativa i EURe

### Kontekst

MiCA (Markets in Crypto-Assets Regulation, 2023/1114) uređuje kriptovalute u EU. EURe (Monerium) je licencirani e-money token pod MiCA-om.

### Implikacije

| Aspekt | Situacija | Rizik |
|--------|-----------|-------|
| **Zamrzavanje sredstava** | MiCA daje izdavaču (Monerium) pravo zamrznuti tokene na zahtjev regulatora ili u slučaju sumnje na pranje novca | Visok — Monerium može zamrznuti sredstva u platformskom Safe-u ili razdjelnicima bez našeg pristanka |
| **KYC zahtjevi** | Monerium zahtijeva KYC za direktnu kupnju EURe | Srednji — korisnici moraju proći KYC kod Moneriuma neovisno o našoj Certilia verifikaciji |
| **Putni nalozi (travel rule)** | MiCA zahtijeva identifikaciju pošiljatelja i primatelja za transfere iznad 1.000 EUR | Srednji — naš razdjelnik plaćanja automatski prenosi sredstva bez eksplicitnog putnog naloga |
| **Licenciranje platforme** | Ako djelujemo kao posrednik u kripto transakcijama, možda trebamo CASP licencu (Crypto-Asset Service Provider) | Visok — zahtijeva pravnu procjenu |

### Otvorena pitanja

| # | Pitanje | Koga pitati |
|---|---------|-------------|
| L5 | Trebamo li CASP licencu za posredovanje u EURe transakcijama? | HANFA (hrvatska regulatorna agencija) |
| L6 | Podliježe li naš razdjelnik plaćanja putnom nalogu za transfere >1.000 EUR? | HANFA / pravni savjetnik za MiCA |
| L7 | Što se događa s sredstvima u razdjelnicima ako Monerium zamrzne EURe na tim adresama? | Monerium pravni tim |
| L8 | Može li se koristiti alternativni stabilni token (npr. DAI) kao rezervna opcija? | Tehniški tim + pravni savjetnik |

---

## 4. Elektroničko glasanje — pravni okvir

### Kontekst

Naša vizija faze 3 uključuje lančano glasanje. U Hrvatskoj i EU, elektroničko glasanje ima specifičan pravni kontekst.

### Trenutno stanje

| Jurisdikcija | Status elektroničkog glasanja | Relevantno |
|-------------|-------------------------------|------------|
| Hrvatska | Nema zakonskog okvira za obvezujuće elektroničko glasanje | Kritično — naše glasanje bi bilo savjetodavno, ne obvezujuće |
| Estonija | Jedina EU država s obvezujućim internetskim glasanjem | Referentna točka |
| Švicarska | Pilot projekti u pojedinim kantonima | Referentna točka |
| EU (općenito) | Nema jedinstvenog okvira, ali eIDAS 2.0 otvara vrata | Dugoročno pozitivno |

### Implikacije za naš projekt

1. **Faza 1-2:** Glasanje unutar privatnih zajednica (udruge, DAO-ovi) nije pravno regulirano — možemo slobodno implementirati.
2. **Faza 3:** Za obvezujuće javno glasanje potrebna je promjena zakona. Naš projekt može poslužiti kao **tehnički dokaz koncepta** koji informira zakonodavca.
3. **Ključno:** Moramo jasno komunicirati da naš sustav **ne zamjenjuje** državni izborni sustav, već ga **nadopunjuje** ili **demonstrira alternativu**.

---

## 5. eIDAS 2.0 i EUDI novčanik

### Kontekst

Europska komisija uvodi EUDI novčanik (European Digital Identity Wallet) kroz eIDAS 2.0 regulativu. Svaka država članica mora ponuditi EUDI novčanik do 2026-2027.

### Utjecaj na naš projekt

| Aspekt | Utjecaj | Implikacija |
|--------|---------|-------------|
| **EUDI novčanik zamjenjuje Certiliju?** | Moguće — EUDI postaje primarni digitalni identitet | Moramo dizajnirati sustav koji je agnostičan prema izvoru identiteta (Certilia ILI EUDI) |
| **EUDI koristi W3C Verifiable Credentials** | Standardizirani format tvrdnji | Naš sustav bi trebao podržavati VC standard, ne samo JWT |
| **EUDI uključuje kriptografski novčanik** | Korisnik već ima kriptografske ključeve | Mogućnost izravnog vezanja EUDI ključa s lančanim identitetom — bez potrebe za zasebnim Web3 novčanikom? |
| **eIDAS 2.0 zahtijeva selektivno otkrivanje** | Korisnik mora moći otkriti samo potrebne atribute | Poklapa se s našim ZK pristupom (Opcija B iz privatnost.md) |

### Preporuka

eIDAS 2.0 ide **u korist** našeg projekta. Ali moramo od prvog dana projektirati **sučelje za identitet** (abstraction layer) koje može raditi s Certilijom danas i EUDI sutra.

---

## 6. Pravna osnova obrade — legitimni interes (NOVO v0.5)

Iz istraživanja 06 (93/100 podudarnost):

**Legitimni interes (čl. 6(1)(f)) je najperspektivnija pravna osnova.** Zahtijeva trodijelni test:

1. **Test svrhe:** Sprječavanje Sybil napada = zaštita integriteta platforme. Uvodna izjava 47 GDPR-a prepoznaje sprječavanje prijevare kao legitimni interes.
2. **Test nužnosti:** Moramo dokazati da ne postoji manje invazivna alternativa. ZK nullifier ovo zadovoljava — minimizira podatke na lancu.
3. **Test ravnoteže:** Nullifier pristup značajno smanjuje utjecaj na prava ispitanika u usporedbi s hash pristupom.

**Privola (čl. 6(1)(a)) je PROBLEMATIČNA** za neizbrisive blockchain zapise — čl. 7(3) zahtijeva da povlačenje privole bude jednako jednostavno kao davanje, što je nemoguće na nepromjenjivom lancu.

### Worldcoin kao presedan

Worldcoin (World) koristi legitimni interes za obradu iris kodova za Sybil otpornost. Regulatorna reakcija bila je masivna (BayLDA, AEPD, CNPD, Garante — kazne i privremene zabrane u 8+ jurisdikcija), ali prigovori su se primarno odnosili na **biometrijske podatke** (čl. 9), ne na sam koncept legitimnog interesa za Sybil otpornost. Naš projekt NE koristi biometrijske podatke — hash/nullifier OIB-a nije biometrija — što nas stavlja u povoljniji regulatorni položaj.

## 7. Prioritetni pravni koraci (AŽURIRANO v0.5)

| Prioritet | Korak | Trošak (procjena) | Status |
|-----------|-------|--------------------|--------|
| 1 | Pročitati Certilia uvjete korištenja (L1-L3) | Besplatno | Čeka registraciju na developer portal |
| 2 | **DPIA (procjena utjecaja na zaštitu podataka)** | 1.000-5.000 EUR | **OBAVEZNO** prema EDPB 02/2025 |
| 3 | **Prethodno savjetovanje s AZOP-om** | Besplatno (ali DPIA potreban kao ulaz) | Preporučeno iz istraživanja 06 |
| 4 | HANFA upit o CASP licenci (L5-L6) | 500-1.000 EUR | Čeka |
| 5 | Konzultacija s GDPR stručnjakom o nullifier pristupu | 500-2.000 EUR | Po potrebi — ako AZOP zatraži |
| 6 | Kontakt s AKD-om oko službene suradnje | Besplatno | Čeka |
| 7 | Praćenje eIDAS 2.0 implementacije u Hrvatskoj | Kontinuirano | EUDI rok kraj 2026. (istraživanje 05) |
