# Križna provjera: GDPR i hashirani identifikatori na blockchainu

**Izvještaji:**
- [06a — Claude Opus 4.6](../reports/06a-gdpr-hash-claude.md)
- [06b — Perplexity](../reports/06b-gdpr-hash-perplexity.md)

**Prompt:** [06-gdpr-hash-identifikatori.md](../prompts/06-gdpr-hash-identifikatori.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 93/100**

Nova najviša ocjena. Oba izvora dolaze do **identičnog pravnog zaključka** i citiraju iste ključne izvore. Razlike su samo u dubini pojedinih aspekata.

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| Hash OIB-a = osobni podatak | 100/100 | Identičan zaključak, iste presude i smjernice. |
| Breyer presuda (C-582/14) | 100/100 | Oba citiraju identično. |
| EDPB 02/2025 o blockchainu | 100/100 | Oba citiraju istu ključnu rečenicu: "hash će se smatrati osobnim podatkom" |
| WP216 — hashing nije anonimizacija | 100/100 | Identičan zaključak. |
| CNIL blockchain smjernice (2018) | 95/100 | Oba: keyed hash + uništavanje ključa kao "funkcionalno brisanje". |
| Pravo na brisanje — fundamentalni sukob | 95/100 | Oba: blockchain nepromjenjivost je u direktnom sukobu s čl. 17. |
| DPIA je obvezan | 95/100 | Oba: da, bez iznimke. |
| Legitimni interes kao pravna osnova | 90/100 | Oba preporučuju. Claude detaljniji (trodijelni test, Worldcoin presedan). |
| Privola problematična za neizbrisivi blockchain | 95/100 | Oba: povlačenje privole nemoguće ispuniti na nepromjenjivom lancu. |
| ZK nullifier kao bolja alternativa | 90/100 | Oba preporučuju. Claude detaljniji (Semaphore, usporedna tablica). |
| AZOP nema blockchain smjernice | 85/100 | Claude: eksplicitno kaže "nema". Perplexity: "Blockchain smjernica" u godišnjem izvješću, ali tekst nedostupan. |
| OIB je osobni podatak po hrvatskom pravu | 95/100 | Oba: nedvosmisleno. |
| Nullifier — pravni status neriješen | 85/100 | Oba: nema presude o ZK dokazima, ali potencijalno izvan GDPR dosega. |

---

## 2. Jednoglasni pravni zaključak

**Oba izvora su JEDNOGLASNA na ključnom pitanju:**

> `keccak256(OIB)` pohranjen na javnom blockchainu **jest osobni podatak** prema GDPR-u.

Ovo nije razlika mišljenja — ovo je pravna činjenica potkrijepljena:
- GDPR čl. 4(1) i Recital 26
- CJEU Breyer (C-582/14)
- WP29 WP216 (Opinion 05/2014)
- EDPB Smjernice 01/2025 o pseudonimizaciji
- EDPB Smjernice 02/2025 o blockchainu
- CNIL blockchain smjernice (2018)
- ICO smjernice o anonimizaciji (UK)
- AEPD/EDPS dokument o hash funkciji

**Implikacija za projekt:** Opcija A iz privatnost.md (selektivno prosljeđivanje s hash(sub) u pohrani) je **GDPR problematična** čak i za fazu 1.

---

## 3. Činjenice na kojima se oba izvora slažu

| # | Činjenica | Povjerenje |
|---|-----------|------------|
| F1 | Hash OIB-a je pseudonimizirani osobni podatak, ne anonimizirani | Vrlo visoko |
| F2 | Brute-force na OIB (10^11 vrijednosti) je trivijalan — minuta do sati | Vrlo visoko |
| F3 | EDPB 02/2025 eksplicitno: hash na blockchainu = osobni podatak | Vrlo visoko |
| F4 | Nesoljeni hash NIJE dostatan za javni blockchain (EDPB 02/2025) | Vrlo visoko |
| F5 | Keyed hash + uništavanje ključa je "bliže brisanju" ali ne garantira usklađenost | Vrlo visoko |
| F6 | Nepromjenjivost blockchaina je u direktnom sukobu s čl. 17 (pravo na brisanje) | Vrlo visoko |
| F7 | Tehnička nemogućnost NE opravdava neusklađenost s GDPR-om | Vrlo visoko |
| F8 | DPIA je obvezan za ovaj projekt (novi tech + neizbrisivi podaci + identifikatori) | Vrlo visoko |
| F9 | Privola je najslabija pravna osnova za neizbrisive blockchain zapise | Vrlo visoko |
| F10 | Legitimni interes je najperspektivnija pravna osnova | Visoko |
| F11 | ZK nullifier pristup je značajno usklađeniji s GDPR-om | Vrlo visoko |
| F12 | Niti jedan regulator još nije formalno potvrdio da je nullifier anoniman | Visoko |
| F13 | Worldcoin regulatorne istrage su relevantan presedan | Visoko |
| F14 | OIB je nedvosmisleno osobni podatak po hrvatskom pravu (čl. 20 NN 42/18) | Vrlo visoko |
| F15 | AZOP nema eksplicitne blockchain smjernice | Visoko |

---

## 4. Činjenice koje je pronašao samo jedan izvor

### Samo Claude

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| C1 | CJEU SRB (C-413/23 P, rujan 2025) — "relativni pristup" za pseudonimizirane podatke | Srednje-visoko | Nova presuda, relevantna za "tko je voditelj obrade" pitanje. |
| C2 | EDPB 01/2025 preporučuje argon2 umjesto brzih hasheva | Srednje | Tehnički relevantan detalj. |
| C3 | AEPD PoC za brisanje podataka hard forkom na privatnom Ethereum lancu | Srednje | Zanimljivo ali neprimjenjivo na javni lanac. |
| C4 | EU Blockchain Observatory (2018): "Ne postoji GDPR-usklađeni blockchain" | Visoko | Jaka izjava, relevantna za DPIA argumentaciju. |
| C5 | AZOP direktor Zdravko Vukić je potpredsjednik EDPB-a | Srednje-visoko | Znači da AZOP prati EDPB mainstream. |
| C6 | Worldcoin kazne i zabrane u 8+ jurisdikcija (detaljni popis) | Visoko | Korisno za model prijetnji. |
| C7 | Perfectly hiding commitment > keyed hash > enkripcija > nesoljeni hash (EDPB hijerarhija) | Visoko | Ključno za dizajn odluku. |

### Samo Perplexity

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| P1 | AEPD/EDPS zajednički dokument specifično o hash funkciji i osobnim podacima | Visoko | Još jedan autoritativan izvor. |
| P2 | AZOP godišnje izvješće spominje "Blockchain smjernicu" | Srednje | Nepoznato sadržajem, ali AZOP je svjestan teme. |
| P3 | Kazne B2 Kapitalu i EOS Matrixu za povrede osobnih podataka u HR | Srednje-visoko | Domaći kontekst za regulatorni rizik. |
| P4 | Detaljna praktična shema za user-held salt pristup | — | Strateški koristan. |
| P5 | Preporuka prethodnog savjetovanja s AZOP-om | Visoko | Pragmatičan korak. |

---

## 5. Zaključci za projekt

### Potvrđeno — zahtijeva hitnu akciju

**Ovo istraživanje fundamentalno mijenja prioritete projekta.**

1. **Opcija A (faza 1: hash(sub) u pohrani) je GDPR problematična.** Oba izvora jednoglasni. Ne možemo ovo ignorirati.
2. **Nullifier pristup (Opcija B) mora biti prioritet, ne "budućnost".** Pomak iz faze 2 u fazu 1, ili barem paralelni razvoj.
3. **DPIA je obvezan PRIJE produkcije.** Procjena troška iz pravna-analiza.md (1.000-5.000 EUR) je realistična.
4. **Legitimni interes** je pravna osnova, ne privola.
5. **Konzultacija s AZOP-om** preporučena prije lansiranja.

### Nova arhitektonska odluka

Na temelju oba izvještaja, preporuka za privatnost.md se mijenja:

| Prije (v0.4) | Poslije (v0.5) |
|---------------|----------------|
| Opcija C hibridni: A za fazu 1, B za fazu 2 | **Opcija B (nullifier) od faze 1**, ili minimalno: soljeni hash s user-held salt + funkcionalnim blokiranjem |

Ako ZK nullifier nije spreman za fazu 1 (složenost), minimalni prihvatljivi pristup je:
- `hash(user_held_salt + sub)` — salt poznat samo korisniku
- Ugovor NE čuva salt
- Korisnik može "obrisati" identitet uništavanjem salta
- DPIA dokumentira ovaj pristup i preostali rizik
- Prethodno savjetovanje s AZOP-om

### Utjecaj na dokumente

| Dokument | Potrebna promjena |
|----------|------------------|
| privatnost.md | Preporuka se mijenja: Opcija B prioritetna, Opcija A samo s user-held salt |
| architecture.md | `identityClaimed` mapiranje mora podržavati funkcionalno blokiranje |
| model-prijetnji.md | Novi T16: GDPR regulatorni rizik (kazna do 20M EUR / 4% prometa) |
| ekonomski-model.md | Dodati trošak DPIA (1-5K EUR) i pravne konzultacije |
| putokaz.md | ZK nullifier pomaknut u fazu 1 ili rani fazu 2 |
| pravna-analiza.md | Integrirati nalaze oba izvještaja (EDPB 02/2025, trodijelni test) |

---

## 6. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Pravna analiza s tehničkim implikacijama, detaljan pregled presuda | Strukturirani pravni pregled s praktičnim preporukama |
| **Prednost** | EDPB hijerarhija rješenja (perfectly hiding > keyed hash > ...), Worldcoin detalji | AEPD/EDPS hash dokument, praktične preporuke za dizajn, AZOP kontekst |
| **Slabost** | Neke presude možda fabricirane (C-413/23 P iz "rujna 2025" — treba verificirati) | Manje tehničkih detalja o nullifier implementaciji |
| **Pouzdanost** | Visoka za pravne zaključke, srednja za specifične datume presuda | Visoka za sve aspekte |

**Oba izvještaja su izuzetno kvalitetna.** Zajedno tvore najjači argument za hitnu promjenu pristupa privatnosti u projektu.
