# Putokaz skaliranja

**Status:** Nacrt — zahtijeva validaciju
**Svrha:** Definirati tehničke zahtjeve za svaku fazu rasta sustava

---

## Pregled faza

```
Faza 1                    Faza 2                    Faza 3
DOKAZ KONCEPTA            LOKALNE ZAJEDNICE          NACIONALNA SKALA
Podcast tržište           Općinski/DAO proračuni     Glasanje i državni proračun
6-12 mjeseci              12-24 mjeseca              24-48 mjeseci
~1.000 korisnika          ~10.000 korisnika          ~100.000+ korisnika
```

---

## Faza 1: Podcast tržište (dokaz koncepta)

### Cilj

Dokazati da **Certilia identitet + lančano upravljanje sredstvima** funkcionira u stvarnom svijetu s pravim korisnicima i pravim eurima.

### Opseg

| Komponenta | Opis | Status |
|------------|------|--------|
| CertiliaSBT ugovor | Lančana JWT verifikacija, kovanje SBT-a | Dizajn |
| Razdjelnik plaćanja | 90/10 automatska raspodjela EURe | Dizajn |
| Node.js poslužitelj (poštar) | OIDC stražnji kanal s Certilijom | Dizajn |
| Frontend (web) | Novčanik, Certilia prijava, kupnja sadržaja | Dizajn |
| Platitelj goriva | ERC-4337 sponzorirano kovanje | Dizajn |
| Platformski Safe | 2/3 višepotpisni za operativne troškove | Dizajn |

### Tehniški zahtjevi

- Lančana RSA ili P-256 verifikacija na Gnosis lancu.
- Selektivno prosljeđivanje JWT-a (Opcija A iz [privatnost.md](privatnost.md)) — minimalan skup podataka u calldata.
- Kovanje SBT-a bez goriva za nove korisnike (ERC-4337 platitelj — vidi Q8).
- Osnovna web aplikacija s integriranim novčanikom.
- CertiliaSBT ugovor s ERC-5192 sučeljem i `IIdentityVerifier` apstrakcijskim slojem (vidi [architecture.md](architecture.md) odjeljak 7.1 i Dodatak).
- `resetIdentityClaim()` za oporavak novčanika (vidi [architecture.md](architecture.md) odjeljak 7.1).
- Višestruki ključevi u ugovoru za podršku rotacije Certilia ključa.

### Odluke koje moraju biti donesene PRIJE početka faze 1

Sve iz centralnog registra otvorenih pitanja u [architecture.md](architecture.md) odjeljak 8, kategorija "Kritična".

### Metrike uspjeha

- 100+ verificiranih korisnika s CERTILIA SBT-om.
- 10+ aktivnih kreatora koji primaju EURe.
- 0 sigurnosnih incidenata.
- Pozitivan odgovor zajednice na GitHub pitanjima.

### Rizici specifični za fazu

| Rizik | Vjerojatnost | Utjecaj | Ublažavanje |
|-------|-------------|---------|-------------|
| Certilia ne podržava prilagođeni nonce | Srednja | Kritičan | Alternativni mehanizam vezanja (vidi Q3 u architecture.md) |
| RSA verifikacija preskupa čak i na Gnosisu | Niska | Visok | ZK off-chain verifikacija kao zamjena |
| Mali broj korisnika (nema interesa) | Srednja | Srednji | Fokus na specifičnu nišu (HR podcast zajednica) |

---

## Faza 2: Lokalne zajednice i DAO proračuni

### Cilj

Proširiti sustav izvan podcast tržišta na **upravljanje zajedničkim proračunima** — npr. lokalne udruge, studentski zborovi, mali općinski projekti.

### Novi zahtjevi (iznad faze 1)

| Komponenta | Opis | Razlog |
|------------|------|--------|
| Glasački modul | Lančano glasanje s jednakom težinom (1 SBT = 1 glas) | Zajednice trebaju donositi zajedničke odluke |
| Predlaganje troškova | Korisnici predlažu isplate iz zajedničkog trezora | Transparentno upravljanje zajedničkim sredstvima |
| Višestruki trezori | Svaka zajednica ima vlastiti Safe | Izolacija sredstava između zajednica |
| Ulazna rampa za eure | Integracija s bankovnim prijenosima (SEPA → EURe) | Korisnici koji nisu kripto izvorni trebaju jednostavan način uplate |
| Poboljšana privatnost | Nullifier pristup (Opcija B iz privatnost.md) | Glasanje mora biti tajno |

### Tehniški zahtjevi

- ZK dokazi za privatno glasanje (Semaphore ili sličan protokol).
- Tvornica ugovora (factory) za lako stvaranje novih zajedničkih trezora.
- Pametni ugovori za predlaganje i odobravanje troškova (governance modul).
- SEPA → EURe ulazna rampa (integracija s Monerium API-jem ili sličnim).

### Metrike uspjeha

- 5+ aktivnih zajednica s vlastitim trezorima.
- 1.000+ glasova provedenih na lancu.
- Pozitivna medijska pokrivenost.
- Interes lokalnih samouprava za pilot projekte.

### Rizici specifični za fazu

| Rizik | Vjerojatnost | Utjecaj | Ublažavanje |
|-------|-------------|---------|-------------|
| ZK složenost usporava razvoj | Visoka | Srednji | Koristiti postojeće ZK biblioteke (Semaphore, circom) |
| Regulatorni otpor (tko smo mi da upravljamo novcem?) | Srednja | Visok | Partnerstvo s pravnim stručnjacima. Pozicioniranje kao alat, ne institucija. |
| Korisnici ne razumiju novčanik pod vlastitom skrbi | Visoka | Srednji | Apstrakcija računa (ERC-4337) — korisnik ne vidi kripto složenost |

---

## Faza 3: Nacionalna skala — glasanje i državni proračun

### Cilj

Demonstrirati da je **potpuno transparentno upravljanje državnim proračunom** i **sigurno elektroničko glasanje** tehnički izvedivo koristeći infrastrukturu razvijenu u fazama 1 i 2.

### Novi zahtjevi (iznad faze 2)

| Komponenta | Opis | Razlog |
|------------|------|--------|
| Potpuna ZK privatnost glasanja | Glasač dokazuje da IMA pravo glasa bez otkrivanja TKO je | Tajnost glasanja je ustavno pravo |
| eIDAS 2.0 / EUDI integracija | Podrška za europski digitalni novčanik | Certilia može biti zamijenjena EUDI standardom |
| Višedržavna podrška | eIDAS interoperabilnost s drugim EU državama | Sustav ne smije biti ograničen na Hrvatsku |
| Skalabilnost lanca | L2 ili specifični lanac za glasanje | Gnosis lanac možda nema kapacitet za milijune glasova |
| Formalna verifikacija ugovora | Matematički dokaz ispravnosti | Nacionalno glasanje ne trpi greške |
| Otvoreni klijenti | Više neovisnih implementacija frontenda | Eliminira ovisnost o jednom timu |

### Tehniški zahtjevi

- ZK-SNARK ili ZK-STARK za dokaz prava glasa bez identifikacije.
- Kompatibilnost s EUDI novčanikom (W3C Verifiable Credentials).
- L2 rješenje ili dedicirani lanac za visoku propusnost glasanja.
- Formalno verificirani pametni ugovori (Certora, Halmos, ili slično).
- Višestruki neovisni frontend klijenti.
- Transparentni lančani proračun — svaka isplata vidljiva javnosti.

### Metrike uspjeha

- Pilot glasanje na razini lokalne samouprave.
- Formalna verifikacija kritičnih ugovora.
- Prepoznavanje od strane državnih institucija.
- Barem 2 neovisne implementacije klijenta.

### Rizici specifični za fazu

| Rizik | Vjerojatnost | Utjecaj | Ublažavanje |
|-------|-------------|---------|-------------|
| Politički otpor | Visoka | Kritičan | Graditi odozdo — zajednice, udruge, lokalne samouprave |
| Regulatorna zabrana lančanog glasanja | Srednja | Kritičan | Raditi s regulatorima, ne protiv njih. eIDAS 2.0 ide u našu korist. |
| Tehnička nezrelost ZK sustava | Srednja | Visok | Koristiti dokazane protokole (Semaphore). Ne izumljivati vlastitu kriptografiju. |
| Gnosis lanac nedovoljan za nacionalnu skalu | Srednja | Srednji | Planirati migraciju na dedicirani L2 ili validium. |

---

## Ovisnosti između faza

```
Faza 1                          Faza 2                          Faza 3
+----------------------------+  +----------------------------+  +----------------------------+
| CertiliaSBT                |  | ZK privatnost              |  | Formalna verifikacija      |
| Razdjelnik plaćanja        |->| Glasački modul             |->| EUDI integracija           |
| OIDC tok                   |  | Tvornica trezora           |  | L2 skalabilnost            |
| Platitelj goriva           |  | SEPA rampa                 |  | Višestruki klijenti        |
+----------------------------+  +----------------------------+  +----------------------------+
       |                               |                               |
       v                               v                               v
  Dokazujemo da radi            Dokazujemo da skalira          Dokazujemo da je spremno
  za male grupe                 za zajednice                   za državu
```

### Kritičan put

1. **Q1-Q4 i L1 iz [architecture.md](architecture.md)** moraju biti riješeni prije početka faze 1.
2. **Calldata privatnost** ([privatnost.md](privatnost.md)) mora biti riješena prije faze 1 (minimalno Opcija A).
3. **GDPR procjena** ([pravna-analiza.md](pravna-analiza.md) L1-L3, P4) — hash(sub) u pohrani zahtijeva DPIA.
4. **ZK privatnost** mora biti riješena prije faze 2 (Opcija B iz [privatnost.md](privatnost.md)).
5. **eIDAS 2.0 praćenje** mora početi odmah — EUDI novčanik može promijeniti cijelu arhitekturu. Zato architecture.md definira `IIdentityVerifier` apstrakcijski sloj od prvog dana.
