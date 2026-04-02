# Križna provjera: Izvori financiranja projekta

**Izvještaji:**
- [08a — Claude Opus 4.6](../reports/08a-financiranje-claude.md)
- [08b — Perplexity](../reports/08b-financiranje-perplexity.md)

**Prompt:** [08-financiranje-projekt.md](../prompts/08-financiranje-projekt.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 72/100**

Niža podudarnost nego kod tehničkih istraživanja — očekivano jer je ovo istraživanje šireg opsega s više spekulativnih elemenata. Claude je značajno širi (60+ izvora, konkretni iznosi i rokovi), Perplexity je konzervativniji (samo potvrđeni izvori, mnoge kategorije ostavio prazne).

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| Blockchain fondacije (Safe, Gnosis, EF) | 85/100 | Oba navode iste ključne programe. Claude ima više detalja (iznosi, rokovi). |
| EU fondovi (CERV, NGI, Horizon) | 75/100 | Oba navode CERV. Claude pronašao NGI Zero Commons Fund i Horizon Cluster 2 koje Perplexity ne navodi. |
| Hrvatski fondovi | 40/100 | **Velika razlika.** Claude detaljno (HAMAG-BICRO, ZICER, HBOR, UBIK). Perplexity samo UBIK. |
| VC-jevi i acceleratori | 30/100 | Claude detaljan (Variant, Fabric, 1kx, Alliance DAO). Perplexity eksplicitno odbija popuniti bez potvrđenih izvora. |
| Hackathoni | 75/100 | Oba navode ETHGlobal i Safe hackathone. Slični datumi. |
| Crowdfunding | 60/100 | Claude navodi Giveth, Octant, Drips, Juicebox. Perplexity samo Gitcoin. |
| Nagrade i natjecanja | 65/100 | Oba navode EBC Startup Battle i PBW. Claude širi. |

---

## 2. Ključne razlike u pristupu

| Aspekt | Claude | Perplexity |
|--------|--------|------------|
| **Broj izvora** | 60+ | ~25 potvrđenih |
| **Iznosi** | Konkretni ($5K-$8M) | Samo gdje javno navedeno, inače "nije potvrđeno" |
| **Rokovi** | Specifični datumi (29.4., 1.6., 23.9.) | Rijetko — "nije u izvatku" |
| **Pouzdanost** | Srednja — neke brojke mogu biti zastarjele ili spekulativne | Visoka — samo potvrđeno iz izvora |
| **Praktičnost** | Vrlo visoka — vremenski plan mjesec po mjesec | Srednja — sugerira "drugu rundu" istraživanja za praznine |

**Zaključak:** Claude je korisniji za AKCIJSKI PLAN, Perplexity za PROVJERU ČINJENICA. Koristiti Claude listu kao radnu, Perplexity za validaciju ključnih rokova i iznosa.

---

## 3. Izvori na kojima se oba slažu (visoko povjerenje)

| # | Izvor | Tip | Iznos | Oba potvrđuju |
|---|-------|-----|-------|---------------|
| F1 | GECO (Gnosis Ecosystem Fund) | Grant | $5K-$100K | Da — oba navode GitHub repo, isti raspon |
| F2 | Safe Grants Program | Grant (wave-based) | €500K ukupno za Wave 1 | Da — oba citiraju isti Mirror post |
| F3 | Safe{Core} Gas Station | Gas krediti | Do $550K kumulativno | Da — oba navode Gnosis Chain komponentu |
| F4 | Safe Ventures | Investicija | Nije specificirano | Da — oba navode safefoundation.org/ventures |
| F5 | Ethereum Foundation ESP | Grant (Wishlist/RFP) | Varijabilno | Da — oba navode novu strukturu (studeni 2025.) |
| F6 | CERV — Citizens' engagement | Grant | Min. €75K, budžet €30M | Da — oba navode kao visoko relevantan |
| F7 | ETHGlobal hackathoni | Hackathon | Varijabilne nagrade | Da — oba navode Cannes, Lisbon, NY |
| F8 | UBIK | Ekosustav | N/A (networking) | Da — oba navode ubik.hr |
| F9 | EBC Startup Battle | Natjecanje | Varijabilno | Da — oba navode |
| F10 | Paris Blockchain Week | Startup natjecanje | $10M fond | Da — oba navode |

---

## 4. Izvori koje je pronašao samo Claude (zahtijevaju verifikaciju)

| # | Izvor | Iznos | Rok | Verifikacija potrebna |
|---|-------|-------|-----|-----------------------|
| C1 | **NGI Zero Commons Fund** | €5K-€50K | 1.6.2026. | Provjeriti nlnet.nl/commonsfund — je li 13. poziv aktivan? |
| C2 | **Horizon Europe Cluster 2** | €2-4M | 23.9.2026. | Provjeriti EC Funding & Tenders portal |
| C3 | **Horizon CL4 Open Internet Stack** | €3-8M | 15.4.2026. | Provjeriti — Claude navodi "vrlo kratak rok" |
| C4 | **HAMAG-BICRO PoC** | €50K-€150K | 2026. | Provjeriti eKohezija |
| C5 | **HAMAG-BICRO Inovacijski vaučeri** | €4,87M ukupno | 31.12.2026. | Provjeriti eKohezija |
| C6 | **ZICER Startup Factory** | Dio €300K | Proljeće 2026. | Provjeriti zicer.hr |
| C7 | **ZICER Seed Fond** | Do €200K | Rolling | Provjeriti zicer.hr |
| C8 | **World Foundation** | Do $50K | Rolling | Provjeriti airtable forme |
| C9 | **Variant Fund** | $250K-$750K | Rolling | Provjeriti projects@variant.fund |
| C10 | **Fabric Ventures** | $500K-$5M | Rolling | Provjeriti x@fabric.vc |
| C11 | **Greenfield Capital** | Do €5M+ | Rolling | Provjeriti greenfieldcapital.com |
| C12 | **Fil Rouge Capital** | Do €500K | Rolling | Provjeriti filrougecapital.com — Zagreb! |
| C13 | **Giveth** | QF ~$91K pool | Odmah | Provjeriti giveth.io |
| C14 | **Octant** | $10K-$50K po epochu | V2 u razvoju | Provjeriti octant.build |
| C15 | **GovTech4All** | Inkubacija | Kontinuirano | Provjeriti interoperable-europe.ec.europa.eu |
| C16 | **Alliance DAO** | $500K | 11.5.2026. | Provjeriti alliance.xyz/apply |
| C17 | **SafeDAO (OBRA framework)** | ~€24K prosječno | Rolling | Provjeriti forum.safe.global |
| C18 | **EBSI Blockchain Sandbox** | Nefinancijski | 4. kohorta 2026. | Provjeriti hub.ebsi.eu |

### Izvori koje je pronašao samo Perplexity

| # | Izvor | Iznos | Komentar |
|---|-------|-------|---------|
| P1 | Creative Europe Cooperation Projects | Do €200K (small) / €1M (medium) | Rok 5.5.2026. — Claude ga navodi ali kao Innovation Lab |
| P2 | IPFS Utility Grants | $5K-$25K | Claude navodi Filecoin ali ne IPFS specifično |
| P3 | UN:BLOCK Pitch Competition | €500K prize pool | Claude ne navodi |
| P4 | OpenCivics Consortium Round | QF round | Povijesno, ali relevantan kanal |

---

## 5. Top 10 — konsenzus između izvora

Na temelju OBA izvještaja, ovo je konsolidirani top 10:

| Rang | Program | Iznos | Rok | Oba izvora? |
|------|---------|-------|-----|-------------|
| 1 | **GECO (Gnosis Ecosystem Fund)** | $5K-$100K | Rolling | Da |
| 2 | **Safe Grants Program** | €500K pool (wave) | Wave-based | Da |
| 3 | **CERV Citizens' engagement** | Min. €75K | ~Travanj 2026. | Da |
| 4 | **EF ESP (Wishlist/RFP)** | Varijabilno | Rolling | Da |
| 5 | **NGI Zero Commons Fund** | €5K-€50K | 1.6.2026.? | Samo Claude — VERIFICIRATI |
| 6 | **Gitcoin Grants** | $5K-$50K QF | Periodično | Oba (ali bez rokova) |
| 7 | **HAMAG-BICRO (PoC / Vaučeri)** | €50K-€150K | 2026. | Samo Claude — VERIFICIRATI |
| 8 | **ETHGlobal hackathoni** | $2K-$20K bounty | Kontinuirano | Da |
| 9 | **Giveth + Octant** | $10K-$80K/god | Odmah | Samo Claude — VERIFICIRATI |
| 10 | **Horizon Europe Cluster 2** | €2-4M | 23.9.2026.? | Samo Claude — VERIFICIRATI |

---

## 6. Hitne akcije (konsenzus)

Oba izvora se slažu na sljedećem redoslijedu:

1. **Odmah:** Kreirati Giveth profil, Gitcoin Grants profil, GECO prijedlog na GitHubu
2. **Ovaj mjesec:** CERV prijava (ako je rok travanj), Safe Grant prijedlog (već pripremljen)
3. **Sljedeći mjesec:** NGI Zero Commons Fund (ako je aktivan), HAMAG-BICRO vaučeri
4. **Paralelno:** ETHGlobal hackathoni za bounty prihode i vidljivost

---

## 7. Metodološke napomene

| Aspekt | Claude | Perplexity |
|--------|--------|------------|
| **Pristup** | Maksimalistički — pokriva sve, daje konkretne brojke i rokove | Minimalistički — samo potvrđeni izvori, eksplicitno odbija nagađati |
| **Prednost** | Akcijski korisno, vremenski plan, kontakt emailovi | Pouzdano — svaka tvrdnja ima izvor |
| **Slabost** | Neke brojke i rokovi mogu biti netočni ili zastarjeli | Propušta mnoge relevantne izvore koje ne može potvrditi |
| **Pouzdanost** | Srednja za specifične brojke, visoka za postojanje programa | Visoka za sve navedeno |

**Preporuka:** Koristiti Claude listu kao RADNU MAPU, ali svaki rok i iznos **verificirati na službenim stranicama** prije prijave. Perplexity pomaže identificirati koje Claude tvrdnje su nepotvrđene.
