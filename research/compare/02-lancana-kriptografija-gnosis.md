# Križna provjera: Lančana RSA/P-256 verifikacija na Gnosis lancu

**Izvještaji:**
- [02a — Claude Opus 4.6](../reports/02a-lancana-kriptografija-gnosis-claude.md)
- [02b — Perplexity](../reports/02b-lancana-kriptografija-gnosis-perplexity.md)

**Prompt:** [02-lancana-kriptografija-gnosis.md](../prompts/02-lancana-kriptografija-gnosis.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 62/100**

Oba izvora se slažu na kvalitativnim zaključcima (izvedivo je, Gnosis je jeftin, biblioteke postoje), ali imaju **kritično proturječje** u kvantitativnim procjenama goriva za RSA-2048.

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| RSA-2048 gas trošak | **15/100** | **KRITIČNO PROTURJEČJE.** Claude: ~25-35K gas. Perplexity: ~1,5-2,5M gas. Razlika od ~50x. |
| P-256 gas trošak (softverski) | 85/100 | Oba u rangu 70-330K gas, ovisno o biblioteci. |
| RIP-7212 status na Gnosisu | 95/100 | Oba kažu: nije dostupan. |
| Gnosis gas cijena | 90/100 | Oba kažu: ~1 gwei, ~$0,002 za 2M gas. |
| OpenZeppelin RSA postojanje | 95/100 | Oba navode OZ RSA v5.1+. |
| FreshCryptoLib P-256 | 95/100 | Oba navode ~69K gas s prekomputacijom, ~205K bez. |
| JWT parsiranje pristup | 85/100 | Oba preporučuju off-chain parsiranje, on-chain samo potpis. |
| ZK alternativa gas | 80/100 | Oba: Groth16 ~200-300K gas on-chain. |
| Postojeći projekti | 70/100 | Oba navode SocialLock. Perplexity dodaje Pixel, Anastasia, Automata. |
| Preporuka za projekt | 50/100 | Claude preporučuje RSA (jer ga smatra jeftinim). Perplexity preporučuje P-256 (jer je stvarno jeftiniji). |

---

## 2. KRITIČNO PROTURJEČJE: RSA-2048 gas trošak

Ovo je najvažniji nalaz usporedbe i zahtijeva razrješenje.

| Izvor | RSA-2048 gas procjena | Temelj procjene |
|-------|----------------------|-----------------|
| **Claude** | ~25.000-35.000 gas | Teoretski izračun: modexp (~5.461 gas po EIP-2565) + SHA-256 + PKCS#1 padding + overhead |
| **Perplexity** | ~1.490.000-2.500.000 gas | RareSkills benchmark (stvarno mjerenje), SocialLock projekt (~1.780.000 gas za puni JWT tok) |

### Analiza proturječja

**Perplexity je vjerojatno u pravu.** Razlozi:

1. **Perplexity citira stvarna mjerenja** (RareSkills benchmark, SocialLock test), dok Claude radi **teoretski izračun** koji podcjenjuje overhead.
2. **Modexp sam po sebi košta ~5.461 gas**, ali RSA verifikacija zahtijeva puno više:
   - SHA-256 hash izvornih podataka
   - PKCS#1 v1.5 padding provjera (usporedba bajtova u Solidity — skupa operacija)
   - Calldata za 256-bajtni potpis + javni ključ
   - Solidity logika za koordinaciju svega
3. **SocialLock** (stvarni projekt koji radi Google JWT RSA-256 verifikaciju) mjeri **~1,78M gas** — ovo je empirijski dokaz.
4. Claude sam navodi SocialLock u svom izvještaju ali ga ne koristi za kalibriranje svojih gas procjena — unutarnje proturječje.

### Utjecaj na projekt

| Scenarij | Gas | Trošak na Gnosisu (1 gwei) | Prihvatljivo? |
|----------|-----|---------------------------|---------------|
| Claude procjena (RSA) | ~30K | ~$0,00003 | Da |
| Perplexity procjena (RSA) | ~1,5-2,5M | ~$0,0015-0,0025 | **Da — i dalje zanemarivo!** |
| P-256 (softverski) | ~70-330K | ~$0,00007-0,0003 | Da |
| ZK Groth16 | ~200-300K | ~$0,0002-0,0003 | Da |

**Ključni zaključak:** Čak i ako je Claude u krivu za ~50x, RSA verifikacija na Gnosisu i dalje košta manje od jednog centa. **Izvedivost nije upitna.** Ali P-256 je objektivno jeftiniji ako Certilia podržava ES256.

---

## 3. Činjenice na kojima se oba izvora slažu (visoko povjerenje)

| # | Činjenica | Povjerenje |
|---|-----------|------------|
| F1 | OpenZeppelin ima RSA (v5.1+) i P256 biblioteke, obje aktivno održavane | Vrlo visoko |
| F2 | Modexp precompile (0x05) dostupan na Gnosisu po EIP-198/EIP-2565 | Vrlo visoko |
| F3 | RIP-7212 (P-256 precompile) NIJE dostupan na Gnosisu (stanje travanj 2026.) | Vrlo visoko |
| F4 | P-256 softverska verifikacija: ~70-330K gas ovisno o biblioteci | Visoko |
| F5 | FreshCryptoLib s prekomputacijom: ~69K gas, deployment ~3,2M gas | Visoko |
| F6 | Gnosis gas cijena: ~1 gwei, ~$0,002 za 2M gas | Vrlo visoko |
| F7 | JWT se ne treba parsirati on-chain — off-chain parsiranje, on-chain samo potpis | Visoko |
| F8 | Groth16 ZK verifikacija on-chain: ~200-300K gas | Visoko |
| F9 | SocialLock: puni JWT RSA verifikacijski tok troši ~1,78M gas | Visoko |
| F10 | adria0/SolRsaVerify je zastarjelo, autor preporučuje OpenZeppelin | Visoko |
| F11 | RSA-PSS (Probabilistic Signature Scheme) NIJE podržan u Solidity bibliotekama | Visoko |
| F12 | Blok gas limit na Gnosisu može primiti transakcije od 2-5M gas bez problema | Visoko |

---

## 4. Činjenice koje je pronašao samo jedan izvor

### Samo Claude

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| C1 | RSA-2048 verifikacija košta ~25-35K gas | **Nisko** | Proturječi benchmarku. Vjerojatno podcjenjuje overhead izvan modexp-a. |
| C2 | Noir framework za ZK RSA verifikaciju | Srednje | Relevantno ali nedovoljno detaljno. |
| C3 | OZ RSA specifično: `pkcs1Sha256` i `pkcs1Sha256Raw` API | Srednje-visoko | Specifični API nazivi, vjerodostojno za OZ. |

### Samo Perplexity

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| P1 | Pixel wallet deployran na Gnosisu s P-256 (čisti Solidity, ~500K-1M gas) | Srednje-visoko | Realan projekt, referentna točka za P-256 na Gnosisu. |
| P2 | Anastasia: ZK verifikacija X.509 certifikata (Noir + UltraHonk) | Srednje | Istraživački projekt, relevantan za eIDAS use case. |
| P3 | Automata DCAP attestation: Solidity verifikatori za Intel SGX/TDX | Srednje | Sličan arhitekturalni uzorak (hardverska atestacija + ZK). |
| P4 | Halo2 ZK proving: ~4s na M1 Pro, ~20s na mobitelu | Srednje | Korisno za procjenu UX-a nullifier pristupa. |
| P5 | RareSkills RSA-2048 benchmark: prosječno ~1,49M gas | Visoko | Stvarno mjerenje, pouzdaniji od teoretskog izračuna. |
| P6 | Obvious Wallet P-256: ~330K gas, deployment ~590K gas | Srednje-visoko | Još jedna biblioteka za usporedbu. |

---

## 5. Razlike u preporukama

| Aspekt | Claude preporuka | Perplexity preporuka |
|--------|-----------------|---------------------|
| **Preferirani algoritam** | RSA (jer ga smatra jeftinim: ~25-35K) | P-256 (jer je stvarno jeftiniji: ~70-330K vs ~1,5M) |
| **JWT pristup** | Off-chain parsiranje, on-chain samo potpis | Isto — off-chain parsiranje |
| **ZK preporuka** | Noir framework, Groth16 | Circom/Halo2/Risc0, Groth16 |
| **Gnosis izvedivost** | "Potpuno izvedivo, zanemariva cijena" | "Potpuno izvedivo, zanemariva cijena" |

---

## 6. Zaključci za projekt

### Potvrđeno (možemo se osloniti)

- Lančana verifikacija JWT potpisa je **potpuno izvediva** na Gnosisu za bilo koji algoritam.
- Trošak je **ispod jednog centa** za sve pristupe — razlika između algoritama je akademska u kontekstu troškova.
- **OpenZeppelin v5.1+** je zlatni standard za i RSA i P-256.
- **Off-chain JWT parsiranje** je preporučeni pristup — na lanac samo potpis i minimalni podaci.
- **ZK alternativa** je izvediva i konkurentna (~200-300K gas za Groth16).

### Zahtijeva razrješenje

| Prioritet | Pitanje | Kako razriješiti |
|-----------|---------|-----------------|
| KRITIČNO | Stvarni gas trošak RSA-2048 s OpenZeppelin v5.1+ na Gnosisu | Deployati OZ RSA na Gnosis testnet i izmjeriti. RareSkills benchmark sugerira ~1,5M ali možda je OZ optimizirao. |
| VISOKO | Koji algoritam koristi Certilia? (RS256 vs ES256) | Vidi istraživanje 01 — registracija na developer portal. |
| SREDNJE | Je li FreshCryptoLib s prekomputacijom (~69K gas) produkcijski spreman? | Provjeriti audit status, deployment na Gnosisu. |

### Utjecaj na arhitekturu

| Nalaz | Utjecaj | Dokument za ažuriranje |
|-------|---------|----------------------|
| RSA trošak je vjerojatno ~1,5M, ne ~25K | Nije blokirajuće (i dalje <$0,01 na Gnosisu), ali utječe na procjenu paymaster proračuna | ekonomski-model.md |
| P-256 je ~5-20x jeftiniji od RSA | Ako Certilia podržava ES256, treba ga preferirati. Ali Q1 (algoritam potpisa) mora biti riješen prvo. | architecture.md (ovisno o Q1) |
| ZK pristup (~200-300K gas) je konkurentan s P-256 softverskim | ZK postaje opcija čak i za fazu 1 ako trebamo privatnost | privatnost.md |
| RSA-PSS nije podržan | Ako Certilia koristi PSS padding umjesto PKCS#1 v1.5, trebamo custom implementaciju ili ZK | Blokirajuće za Q1 |
| Pixel wallet radi P-256 na Gnosisu | Dokaz da P-256 softverska verifikacija radi u produkciji na Gnosisu | architecture.md |

---

## 7. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Teoretski izračun iz specifikacija (EIP-2565 formula) | Empirijski podaci iz benchmarkova i projekata |
| **Prednost** | Čist i strukturiran pregled opcija | Realniji gas brojevi, više referentnih projekata |
| **Slabost** | **Ozbiljno podcjenjuje RSA gas (~50x)** — teoretski izračun ne uhvati overhead | Ponekad citira izvore koji ne odgovaraju tvrdnji (neke reference su nebitne) |
| **Pouzdanost gas procjena** | Niska za RSA, srednja za P-256 | Visoka za RSA (benchmark), srednja-visoka za P-256 |

**Preporuka:** Za gas procjene, **Perplexity je pouzdaniji** jer citira stvarna mjerenja. Claude je korisniji za pregled biblioteka i API-ja. Konačnu potvrdu daje samo vlastito mjerenje na Gnosis testnet-u.
