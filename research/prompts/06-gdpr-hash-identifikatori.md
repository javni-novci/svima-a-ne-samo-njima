# Deep Research: GDPR i hashirani identifikatori na blockchainu

## Kontekst za istraživača

Gradim sustav na Gnosis blockchainu koji pohranjuje `keccak256(hash(OIB))` (ili sličan hash osobnog identifikatora) u pohrani pametnog ugovora kao zaštitu od Sybil napada (jedna osoba = jedan identitet). Ovaj hash je **trajno zapisan na javnom, nepromjenjivom blockchainu**.

Trebam razumjeti pravni status ovakvih hashiranih identifikatora prema GDPR-u i primjenjivom hrvatskom zakonu.

## Pitanja na koja trebam odgovore

### 1. Hash kao osobni podatak

- Je li kriptografski hash osobnog identifikatora (poput OIB-a) "osobni podatak" prema GDPR-u (Članak 4(1))?
- Presuda Breyer (C-582/14) — što točno kaže o pseudonimnim identifikatorima?
- EDPB (European Data Protection Board) smjernice — postoje li specifične smjernice o hashiranim podacima?
- AZOP (Agencija za zaštitu osobnih podataka) — postoje li hrvatski specifični stavovi?
- Razlika između pseudonimizacije i anonimizacije prema GDPR-u — gdje hash pada?
- Je li keccak256 hash jednosmjeran dovoljno da bude "anonimizacija" ili je uvijek samo "pseudonimizacija"?

### 2. Blockchain i pravo na brisanje (Članak 17)

- Mogu li se podaci na blockchainu brisati? Kako GDPR tretira nepromjenjive sustave?
- Postoji li sudska praksa ili mišljenja regulatora specifično o blockchainu i pravu na brisanje?
- Francuska CNIL smjernice o blockchainu (2018) — što kažu?
- EDPB ili nacionalni regulatori — postoje li novije smjernice (2023-2026)?
- Može li "soljeni hash" (hash s tajnom soli poznatom samo korisniku) riješiti problem brisanja? (Ako korisnik uništi sol, hash postaje de facto nepovratan.)

### 3. DPIA (Procjena utjecaja na zaštitu podataka)

- Kada je DPIA obavezan prema GDPR-u (Članak 35)?
- Je li naš sustav (lančano pohranjivanje pseudonimnih identifikatora) kandidat za obvezni DPIA?
- Postoji li predložak DPIA-ja za blockchain projekte?
- Što DPIA mora sadržavati za naš specifični slučaj?

### 4. Pravna osnova obrade (Članak 6)

- Koja pravna osnova je najprikladnija za naš sustav?
  - Privola (Članak 6(1)(a)) — korisnik pristaje na kovanje SBT-a?
  - Legitimni interes (Članak 6(1)(f)) — sprečavanje Sybil napada?
  - Izvršavanje ugovora (Članak 6(1)(b)) — korisnik želi pristup platformi?
- Ako koristimo privolu, može li se privola povući? Što tada — ne možemo brisati hash s blockchaina.

### 5. Nullifier kao alternativa

- ZK nullifier pristup: na lancu nema pseudonimnog identifikatora, samo kriptografski dokaz jedinstvenosti. Je li ovo GDPR-usklađenije?
- Postoje li pravna mišljenja o ZK dokazima i GDPR-u?
- Semaphore protokol / World ID — jesu li se oni suočili s GDPR pitanjima? Kako su ih riješili?

### 6. Hrvatski kontekst

- AZOP — postoje li objavljeni stavovi o blockchainu i osobnim podacima?
- Zakon o provedbi Opće uredbe o zaštiti podataka (NN 42/18) — specifičnosti za Hrvatsku?
- Postoje li hrvatski projekti koji pohranjuju hashirane identifikatore na blockchain? Kako su riješili GDPR?

### 7. Relevantne presude i mišljenja

Pronađi i sažmi ključne:
- CJEU presude o pseudonimizaciji i anonimizaciji
- CNIL blockchain smjernice
- ICO (UK) smjernice o hashiranju osobnih podataka
- Bilo koji EDPB dokument koji spominje blockchain, DLT ili hashirane identifikatore

## Format odgovora

1. **Pravni zaključak** — jasna izjava: je li naš hash osobni podatak ili nije
2. **Relevantne presude i smjernice** — tablica s nazivom, datumom, ključnim zaključkom
3. **Preporuke za naš projekt** — konkretne mjere koje moramo poduzeti
4. **Izvori** — službeni dokumenti, linkovi na presude

Sav izlaz na **hrvatskom jeziku**.
