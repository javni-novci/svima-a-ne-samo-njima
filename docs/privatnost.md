# ADR-004: Privatnost lančanih podataka

**Status:** Otvoreno — zahtijeva odluku prije implementacije
**Prioritet:** Kritičan — blokira viziju nacionalnog glasanja

---

## Kontekst

Tvrdimo da "nema osobnih podataka na lancu", ali trenutna arhitektura ima dva ozbiljna propusta privatnosti koja moramo riješiti prije nego napišemo ijednu liniju kôda.

### Problem 1: Calldata curenje

Kada korisnik pozove `claimCertiliaSBT(bytes jwt)`, **cijeli JWT završava u calldata transakcije** — trajno vidljiv na javnom lancu. Čak i ako pametni ugovor ne sprema JWT u svoju pohranu, calldata je dostupan zauvijek putem bilo kojeg istraživača blokova.

Tipični Certilia JWT može sadržavati:

```
{
  "sub": "HR-123456789",        // jedinstveni identifikator (moguće OIB)
  "name": "Ivan Horvat",        // ime i prezime
  "email": "ivan@example.com",  // elektronička pošta
  "nonce": "0xABC...",          // adresa novčanika
  "iss": "https://certilia.hr", // izdavač
  "exp": 1720000000             // istek
}
```

**Posljedica:** Bilo tko može pročitati transakciju i doznati ime, prezime i identifikator osobe koja stoji iza određene Ethereum adrese. Ovo je **potpuna deanonimizacija** i kršenje GDPR-a.

### Problem 2: Pseudonimni identifikator u pohrani

Čak i ako riješimo calldata problem, `identityClaimed[hash(jwt.sub)]` je pseudonimni identifikator trajno zapisan u pohrani ugovora. Ako itko ikada poveže taj hash s OIB-om (npr. curenje baze podataka, korelacijska analiza), svi SBT-ovi postaju deanonimizirani.

Za nacionalno glasanje ovo znači: **glasovi postaju javni.**

---

## Razmatrane opcije

### Opcija A: Selektivno prosljeđivanje (minimalni JWT)

Umjesto slanja cijelog JWT-a, klijent šalje samo neophodne dijelove: potpis, sažetak sadržaja, nonce i `sub` tvrdnju.

```
Na lanac ide:
  - RSA potpis (256 bajtova)
  - SHA-256 sažetak JWT sadržaja (32 bajta)
  - nonce / adresa novčanika (20 bajtova)
  - hash(sub) za Sybil zaštitu (32 bajta)

Na lanac NE ide:
  - ime, prezime, e-pošta, ostale tvrdnje
```

**Preduvjet:** Pametni ugovor mora moći verificirati RSA potpis nad sažetkom, a ne nad cijelim JWT-om. Ovo zahtijeva da Certilia koristi standardni JWT format gdje je potpis nad base64(header).base64(payload).

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 60 | Rješava calldata, ali hash(sub) ostaje pseudonimni identifikator. Korelacijski napad i dalje moguć. |
| Korisničko iskustvo (20%) | 85 | Transparentno za korisnika — klijent automatski filtrira. |
| Gorivo i mrežna učinkovitost (20%) | 80 | Manje podataka na lancu = manje goriva. |
| Skalabilnost i nadogradivost (20%) | 50 | Ne rješava temeljni problem za glasanje. Potrebna migracija na ZK za fazu 3. |

**Ponderirani prosjek: 66**

### Opcija B: Nullifier pristup (à la Semaphore / World ID)

Korisnik off-chain generira kriptografski "nullifier" — jedinstven, nepovratan identifikator koji dokazuje jedinstvenost bez otkrivanja identiteta.

```
Tok:
1. Korisnik dobije JWT od Certilije
2. Klijent lokalno računa:
   - identityCommitment = hash(privatniKljuc, hash(jwt.sub))
   - nullifier = hash(identityCommitment, kontekst)
3. Klijent generira ZK dokaz koji dokazuje:
   "Imam validan JWT potpisan od AKD-a
    čiji sub hashira u moj identityCommitment,
    a moj nullifier je jedinstven za ovaj kontekst"
4. Na lanac ide SAMO:
   - ZK dokaz (konstantne veličine)
   - nullifier (32 bajta)
   - adresa novčanika (20 bajtova)
```

**Na lancu nema nikakvih osobnih podataka, čak ni pseudonimnih.**

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 95 | Potpuna privatnost. Nullifier sprečava Sybil bez otkrivanja identiteta. Jedini rizik je greška u ZK krugu. |
| Korisničko iskustvo (20%) | 40 | ZK generiranje dokaza traje 10-30s na mobitelu. Korisnik mora čekati. |
| Gorivo i mrežna učinkovitost (20%) | 70 | ZK verifikacija troši ~300K goriva (Groth16). Jeftinije od RSA, ali zahtijeva postavljeni ceremonijalac. |
| Skalabilnost i nadogradivost (20%) | 95 | Isti pristup funkcionira za glasanje, proračun i sve buduće primjene. Nullifier kontekst omogućuje različite primjene istog identiteta. |

**Ponderirani prosjek: 79**

### Opcija C: Hibridni pristup (A za dokaz koncepta, B za produkciju)

Faza 1 (dokaz koncepta) koristi selektivno prosljeđivanje (Opcija A) jer je jednostavnije i brže za implementaciju. Faza 2+ migrira na nullifier pristup (Opcija B) za glasanje i osjetljive primjene.

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 70 | Prihvatljivo za tržište (nema glasanja), neprihvatljivo za nacionalnu skalu — ali migracija je planirana. |
| Korisničko iskustvo (20%) | 85 | Faza 1 je jednostavna. ZK trenje dolazi tek kad je ekosustav zreliji. |
| Gorivo i mrežna učinkovitost (20%) | 80 | Optimalno za svaku fazu. |
| Skalabilnost i nadogradivost (20%) | 75 | Zahtijeva migraciju ugovora. Stari SBT-ovi moraju biti zamijenjeni nullifier pristupom. |

**Ponderirani prosjek: 76**

### Opcija D: Potpuno izvanlanačna verifikacija s lančanom atestacijom (ODBAČENO)

Verifikacija se obavlja off-chain (npr. na TEE-u ili putem oracle-a), a na lanac ide samo potpisan rezultat.

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 25 | Uvodi povjerenje u off-chain verifikatora. Centralna točka kvara. |
| Korisničko iskustvo (20%) | 90 | Najjednostavnije za korisnika. |
| Gorivo i mrežna učinkovitost (20%) | 90 | Minimalan trošak goriva. |
| Skalabilnost i nadogradivost (20%) | 40 | Off-chain komponenta je usko grlo i SPOF. |

**Ponderirani prosjek: 51**

---

## Preporuka

**Opcija C (hibridni pristup)** — pragmatičan put koji ne žrtvuje dugoročnu viziju.

Ali ovo zahtijeva da od prvog dana dizajniramo ugovor s **migrabilnošću na umu** — SBT ugovor mora imati verzioniranje i mogućnost zamjene verifikacijske logike.

---

## Neriješena pitanja specifična za privatnost

| # | Pitanje | Kontekst |
|---|---------|----------|
| P1 | Koji ZK sustav koristiti? Groth16, PLONK, Halo2? | Groth16 zahtijeva pouzdanu ceremoniju postavljanja. PLONK ne zahtijeva ali je sporiji. Halo2 nema ceremoniju ali je najnoviji. |
| P2 | Može li se RSA verifikacija učinkovito implementirati unutar ZK kruga? | RSA je skup za ZK. Ako Certilia koristi P-256, circom-ecdsa biblioteke već postoje. |
| P3 | Kako riješiti opozivost u nullifier modelu? | Ako korisnik izgubi mobitel, kako opozvati nullifier bez otkrivanja identiteta? |
| P4 | Je li hash(OIB) "osobni podatak" po GDPR-u? | Sudska praksa EU sugerira da DA — pseudonimizacija nije anonimizacija. Vidi pravna-analiza.md. |
