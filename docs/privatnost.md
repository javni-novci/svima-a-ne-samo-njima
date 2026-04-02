# ADR-004: Privatnost lančanih podataka

**Status:** ODLUČENO — nullifier pristup (Opcija B) obvezan od faze 1
**Prioritet:** Kritičan — GDPR zahtijeva, blokira produkciju
**Ažurirano:** v0.5 — na temelju istraživanja 06 (GDPR, 93/100 podudarnost)

---

## Kontekst

Istraživanje 06 (GDPR i hashirani identifikatori) jednoglasno potvrđuje:

> **`keccak256(OIB)` pohranjen na javnom blockchainu JEST osobni podatak prema GDPR-u.**

Ovo nije mišljenje — ovo je pravni zaključak potkrijepljen:
- **EDPB Smjernice 02/2025** o blockchainu: "hash će se također smatrati osobnim podatkom"
- **CJEU Breyer (C-582/14):** relativni test identifikabilnosti
- **WP29 WP216:** hashing nije anonimizacija
- **EDPB Smjernice 01/2025:** pseudonimizirani podaci = osobni podaci
- **CNIL blockchain smjernice (2018):** nesoljeni hash nedostatan

OIB ima prostor od ~10^11 vrijednosti — brute-force izračun svih hasheva traje **minutu do sati** na modernom hardveru. Rainbow table napad je trivijalan.

### Problem 1: Calldata curenje

Certilia JWT (potvrđeno iz istraživanja 01) sadrži **OIB eksplicitno**. Ako cijeli JWT završi u calldata, to je potpuna deanonimizacija.

### Problem 2: Pseudonimni identifikator u pohrani

`identityClaimed[hash(sub)]` je pseudonimni identifikator trajno zapisan na javnom, nepromjenjivom lancu. EDPB 02/2025 eksplicitno navodi da ovo predstavlja osobni podatak. Članak 17. GDPR-a (pravo na brisanje) je u **direktnom sukobu** s nepromjenjivošću blockchaina.

**EDPB 02/2025:** "Tehnička nemogućnost ne može se pozivati kao opravdanje za neusklađenost sa zahtjevima GDPR-a."

---

## Razmatrane opcije

### Opcija A: Selektivno prosljeđivanje (minimalni JWT) — GDPR NEUSKLAĐENA

Na lanac ide samo potpis + sažetak + nonce + hash(sub). Osobni podaci ne idu u calldata.

```
Na lanac ide:
  - RSA potpis (256 bajtova)
  - SHA-256 sažetak JWT sadržaja (32 bajta)
  - nonce / adresa novčanika (20 bajtova)
  - hash(sub) za Sybil zaštitu (32 bajta)

Na lanac NE ide:
  - ime, prezime, OIB, e-pošta, ostale tvrdnje
```

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 60 | Rješava calldata, ali hash(sub) ostaje pseudonimni identifikator. Brute-force trivijalan. |
| Korisničko iskustvo (20%) | 85 | Transparentno za korisnika. |
| Gorivo i mrežna učinkovitost (20%) | 80 | Manje podataka na lancu = manje goriva. |
| Skalabilnost i nadogradivost (20%) | 30 | **GDPR neusklađena.** Hash(sub) je osobni podatak. Pravo na brisanje neostvarivo. |

**Ponderirani prosjek: 59** (sniženo zbog GDPR nalaza)

**GDPR status:** NEUSKLAĐENA za produkciju. EDPB 02/2025 eksplicitno: nesoljeni hash na javnom blockchainu nije dostatan.

### Opcija A2: Selektivno prosljeđivanje s user-held salt — MINIMALNO PRIHVATLJIVA

Varijanta Opcije A gdje korisnik drži tajnu sol. Na lancu: `hash(user_salt + sub)`. Sol poznaje SAMO korisnik.

```
Na lanac ide:
  - RSA potpis (256 bajtova)
  - SHA-256 sažetak JWT sadržaja (32 bajta)
  - nonce / adresa novčanika (20 bajtova)
  - hash(user_salt + sub) za Sybil zaštitu (32 bajta)

Korisnik čuva lokalno:
  - user_salt (32 bajta) — u novčaniku ili lokalnoj pohrani

Poslužitelj NIKADA ne vidi:
  - user_salt
```

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 70 | Brute-force otežan tajnom soli. Korisnik može "obrisati" identitet uništavanjem soli. Ali: ako korisnik izgubi sol, ne može dokazati identitet za novi SBT. |
| Korisničko iskustvo (20%) | 70 | Korisnik mora čuvati sol. Gubitak soli = gubitak identiteta. |
| Gorivo i mrežna učinkovitost (20%) | 80 | Isto kao Opcija A. |
| Skalabilnost i nadogradivost (20%) | 50 | CNIL smatra da keyed hash "približava učincima brisanja" ali ne potvrđuje potpunu usklađenost. Pravna siva zona. |

**Ponderirani prosjek: 67**

**GDPR status:** SIVA ZONA. CNIL prihvaća kao "bliže brisanju" ali nijedno tijelo nije formalno potvrdilo punu usklađenost. Zahtijeva DPIA i preporučeno prethodno savjetovanje s AZOP-om.

### Opcija B: Nullifier pristup (à la Semaphore / World ID) — PREPORUČENA

Korisnik off-chain generira kriptografski nullifier. Na lancu nema nikakvih osobnih podataka.

```
Tok:
1. Korisnik dobije JWT od Certilije (off-chain, kroz OIDC tok)
2. Klijent lokalno računa:
   - identityCommitment = Poseidon(privatniSkalar, hash(jwt.sub))
   - nullifier = Poseidon(identityCommitment, kontekst)
3. Klijent generira ZK dokaz koji dokazuje:
   "Imam validan JWT potpisan od AKD-a
    čiji sub hashira u moj identityCommitment,
    a moj nullifier je jedinstven za ovaj kontekst"
4. Na lanac ide SAMO:
   - ZK dokaz (konstantne veličine, ~200-300K gas za Groth16 verifikaciju)
   - nullifier (32 bajta)
   - adresa novčanika (20 bajtova)
```

**Na lancu nema nikakvih osobnih podataka, čak ni pseudonimnih.**

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 95 | Potpuna privatnost. Nullifier ne otkriva identitet. Različit nullifier po kontekstu = nepovezivost. |
| Korisničko iskustvo (20%) | 40 | ZK generiranje dokaza: ~4s na M1 Pro, ~20s na mobitelu (iz istraživanja 02, Halo2 benchmark). Korisnik mora čekati. |
| Gorivo i mrežna učinkovitost (20%) | 75 | Groth16 verifikacija ~200-300K gas na Gnosisu = ~$0,0003. Jeftinije od RSA (~1,5M gas). |
| Skalabilnost i nadogradivost (20%) | 95 | Isti pristup za glasanje, proračun, sve buduće primjene. Kontekstni nullifier omogućuje per-dApp jedinstvenost. |

**Ponderirani prosjek: 81**

**GDPR status:** ZNAČAJNO USKLAĐENIJA. AEPD (Španjolska) kvalificira ZK dokaze kao "moćne alate pseudonimizacije" koji implementiraju čl. 25 GDPR-a (privatnost od temelja). Nullifier potencijalno izvan GDPR dosega — ali formalno nepotvrđeno od regulatora.

### Opcija D: Potpuno izvanlanačna verifikacija (ODBAČENO)

| Metrika | Ocjena (0-100) | Obrazloženje |
|---------|----------------|--------------|
| Sigurnost i odsutnost povjerenja (40%) | 25 | Centralna točka kvara. |
| Korisničko iskustvo (20%) | 90 | Najjednostavnije. |
| Gorivo i mrežna učinkovitost (20%) | 90 | Minimalan gas. |
| Skalabilnost i nadogradivost (20%) | 40 | SPOF, ne skalira. |

**Ponderirani prosjek: 51**

---

## Odluka (AŽURIRANO v0.5)

**Opcija B (nullifier pristup) je OBAVEZNA za produkciju.** GDPR ne dopušta kompromis.

### Praktični pristup za fazu 1

Ako ZK nullifier nije spreman za dan 1 lansiranja:

1. **Faza 1a (rani pristup, ograničeni korisnici):** Opcija A2 (user-held salt) s jasnim upozorenjima korisnicima, DPIA-jem i savjetovanjem s AZOP-om. Eksplicitna privremena mjera.
2. **Faza 1b (produkcija):** Opcija B (nullifier) mora biti spremna prije šireg lansiranja.
3. **Migracija 1a→1b:** Korisnici iz faze 1a moraju skovati nove SBT-ove pod nullifier sustavom. Stari hash-bazirani zapisi ostaju na lancu ali postaju funkcionalno neaktivni.

### GDPR matrica usklađenosti

| Zahtjev GDPR-a | Opcija A | Opcija A2 (salt) | Opcija B (nullifier) |
|-----------------|----------|-------------------|----------------------|
| Čl. 5(1)(c) — minimizacija podataka | Djelomično | Djelomično | Da |
| Čl. 17 — pravo na brisanje | Ne | Siva zona (uništenje soli) | Da (uklanjanje iz Merkle stabla) |
| Čl. 25 — privatnost od temelja | Ne | Djelomično | Da |
| Čl. 35 — DPIA obvezan | Da | Da | Da (ali manji rizik) |
| Recital 26 — anonimizacija | Ne (hash je pseudonim) | Uvjetno (ovisi o soli) | Potencijalno da |
| EDPB 02/2025 usklađenost | Ne | Siva zona | Da |

---

## Neriješena pitanja (ažurirano)

| # | Pitanje | Status | Kontekst |
|---|---------|--------|----------|
| P1 | Koji ZK sustav koristiti? | Otvoreno | Groth16 (ceremonija ali ~200K gas), PLONK (bez ceremonije ali sporiji), Halo2 (~500K gas, ~4-20s proving). Istraživanje 02 daje benchmark. |
| P2 | Može li se RSA-2048 učinkovito staviti u ZK krug? | Djelomično odgovoreno | Iz istraživanja 02: zkEmail i Anastasia to rade. Deseci milijuna constraintova. Proving na mobitelu: sekunde do desetke sekundi. |
| P3 | Kako riješiti opozivost u nullifier modelu? | Otvoreno | Uklanjanje identityCommitmenta iz Merkle stabla (Safe admin). Ali: kako korisnik dokazuje identitet za novi SBT bez ZK? |
| P4 | Je li hash(OIB) osobni podatak? | **RIJEŠENO: DA.** | EDPB 02/2025, Breyer, WP216, CNIL — jednoglasno. Vidi istraživanje 06. |
| P5 | Je li nullifier osobni podatak? | Otvoreno (novo) | Argumenti za "ne": ne može se reverzirati, kontekstno specifičan. Argumenti za "da": GDPR široka definicija. Nijedno tijelo nije formalno potvrdilo. |
| P6 | Legitimni interes ili privola za pravnu osnovu? | Djelomično odgovoreno (novo) | Istraživanje 06: legitimni interes najperspektivniji. Privola problematična jer se ne može efektivno povući s nepromjenjivog lanca. |
