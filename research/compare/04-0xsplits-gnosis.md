# Križna provjera: 0xSplits i razdjelnici plaćanja na Gnosis lancu

**Izvještaji:**
- [04a — Claude Opus 4.6](../reports/04a-0xsplits-gnosis-claude.md)
- [04b — Perplexity](../reports/04b-0xsplits-gnosis-perplexity.md)

**Prompt:** [04-0xsplits-gnosis.md](../prompts/04-0xsplits-gnosis.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 91/100**

Dosad najviša podudarnost. Oba izvora se slažu na gotovo svemu. Razlike su uglavnom u dubini detalja, ne u činjenicama.

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| 0xSplits na Gnosisu (V1 + V2) | 95/100 | Oba potvrđuju. Claude ima V1 adresu, oba navode V2 Warehouse na chainId 100. |
| ERC-20 / EURe podrška | 95/100 | Oba: da, standardni ERC-20. Oba upozoravaju na ograničenja (fee-on-transfer, rebasing — ne odnosi se na EURe). |
| Push vs Pull model | 95/100 | Identičan opis V2 PushSplit/PullSplit arhitekture. |
| Revizije | 95/100 | Identično: V1 Shipyard, V2 Zach Obront (Sherlock/Spearbit). |
| SDK | 90/100 | Oba navode `@0xsplits/splits-sdk`. Claude daje primjer koda. |
| EURe adrese na Gnosisu | 95/100 | Identične adrese: V1 `0xcB444...`, V2 `0x420CA...` |
| EURe freeze/blacklist | 95/100 | Oba: da, Monerium može zamrznuti. Oba navode LCX hack primjer. |
| Monerium SEPA rampa | 85/100 | Oba opisuju SEPA→EURe→Safe tok. Claude detaljniji o API-ju. |
| Superfluid | 90/100 | Oba: na Gnosisu, ali overkill za 90/10. Zahtijeva Super Token wrapping. |
| Sablier | 85/100 | Claude: na Gnosisu ali za streaming. Perplexity: isto, s više detalja (24+ mreža). |
| Drips | 95/100 | Oba: NIJE na Gnosisu. |
| OZ PaymentSplitter | 85/100 | Oba navode. Claude naglašava da je uklonjen iz OZ v5. Perplexity detaljnija o pull modelu. |
| Gas troškovi | 75/100 | Razlika u procjenama (vidi dolje). |

---

## 2. Gas procjene — jedina značajnija razlika

| Operacija | Claude procjena | Perplexity procjena | Komentar |
|-----------|----------------|---------------------|---------|
| CreateSplit (2 primatelja) | ~100-150K gas | ~1,2M gas (citira Etherscan) | Perplexity citira stvarnu transakciju, ali za V1 s možda više primatelja. Za 2 primatelja Claude je vjerojatno bliži istini. |
| Distribucija (2 primatelja, ERC-20) | ~80-120K gas | Linearna, ali bez specifične brojke za 2 primatelja | Nema proturječja — oboje kažu da je za 2 primatelja jeftino. |

**Zaključak:** Razlika u gas procjenama nije kritična jer su oba reda veličine zanemarivi na Gnosisu (<$0,001).

---

## 3. Činjenice na kojima se oba izvora slažu (visoko povjerenje)

| # | Činjenica | Povjerenje |
|---|-----------|------------|
| F1 | 0xSplits V1 i V2 deployani na Gnosis Chain (chainId 100) | Vrlo visoko |
| F2 | V2 koristi CREATE2 determinističko postavljanje (iste adrese na svim lancima) | Vrlo visoko |
| F3 | V2 PushSplit šalje sredstva odmah; fallback u Warehouse ako transfer ne uspije | Vrlo visoko |
| F4 | 0xSplits nema protokolnih naknada | Vrlo visoko |
| F5 | EURe je standardni ERC-20 s 18 decimala, kompatibilan s 0xSplits | Vrlo visoko |
| F6 | EURe V1: `0xcB444e90D8198415266c6a2724b7900fb12FC56E`, V2: `0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430` | Vrlo visoko |
| F7 | Monerium može zamrznuti adrese (blacklist); LCX hack 2022 kao precedent | Vrlo visoko |
| F8 | V1 revidirao Shipyard, V2 revidirao Zach Obront | Vrlo visoko |
| F9 | Ugovori su nenadogradivi (non-upgradeable) | Visoko |
| F10 | SDK: `@0xsplits/splits-sdk` (viem-based, TS/JS) | Vrlo visoko |
| F11 | Superfluid na Gnosisu, ali zahtijeva wrapping u Super Token — overkill za 90/10 | Visoko |
| F12 | Sablier na Gnosisu, ali za streaming/vesting — neprimjenjivo za instantni split | Visoko |
| F13 | Drips NIJE na Gnosisu | Visoko |
| F14 | OZ PaymentSplitter: pull model, funkcionalan ali uklonjen iz OZ v5 | Visoko |
| F15 | Monerium SEPA rampa integrirana sa Safe na Gnosisu | Visoko |
| F16 | EURe V1 prosljeđuje na V2, emitira dvostruke Transfer evente | Visoko |

---

## 4. Činjenice koje je pronašao samo jedan izvor

### Samo Claude

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| C1 | V1 SplitMain adresa: `0x2ed6c4B5dA6378c7897AC67Ba9e43102Feb694EE` | Visoko | Specifična adresa s 73+ ERC-20 tokena. |
| C2 | EURe podržava ERC-2612 `permit()` za gasless odobrenja | Srednje-visoko | Relevantno za UX — korisnik ne treba zasebni approve tx. |
| C3 | EURe nema transfer hookova — nema rizika od reentrancy | Visoko | Važno za sigurnosnu analizu razdjelnika. |
| C4 | Monerium `recover` funkcija za preseljenje s izgubljenog novčanika | Srednje | Nova informacija, utječe na model prijetnji. |
| C5 | SDK primjer koda za kreiranje PushSplit-a na Gnosisu | — | Praktično korisno za implementaciju. |

### Samo Perplexity

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| P1 | 0xSplits subgraph postoji za Gnosis | Visoko | Važno za indeksiranje i praćenje raspodjela. |
| P2 | V2 audit upozorava: >1.100 primatelja može premašiti blok gas limit | Visoko | Nije relevantno za nas (2 primatelja), ali dobro za poznavanje. |
| P3 | Monerium tri uloge: Owner (Safe multisig), Admin, System | Visoko | Detaljnija analiza governance strukture. |
| P4 | Sablier na 24+ mreža uključujući Gnosis | Srednje-visoko | Više detalja o Sablieru. |

---

## 5. Zaključci za projekt

### Potvrđeno — odluka za ADR Q6

Oba izvora jednoznačno potvrđuju: **0xSplits V2 PushSplit je optimalno rješenje za naš use case.**

| Kriterij | 0xSplits V2 | OZ PaymentSplitter | Vlastiti razdjelnik |
|----------|-------------|--------------------|--------------------|
| Na Gnosisu | Da | Da (ali OZ v5 uklonio) | Da |
| Revidiran | Da (Zach Obront) | Da (OZ ekosustav) | Ne |
| EURe blacklist handling | Da (fallback u Warehouse) | Ne (revert blokira sve) | Moguće (ali moraš sam) |
| SDK | Da (zreo) | Ne | Ne |
| Složenost | Srednja | Niska | Srednja-visoka |
| Protokolne naknade | 0% | 0% | 0% |

**Preporuka:** 0xSplits V2 PushSplit za fazu 1. OZ PaymentSplitter je prihvatljiv fallback ako se pojave problemi s 0xSplits.

### Utjecaj na arhitekturu

| Nalaz | Utjecaj | Dokument za ažuriranje |
|-------|---------|----------------------|
| 0xSplits V2 PushSplit + Warehouse fallback | Rješava EURe blacklist problem iz model-prijetnji.md (T13) | architecture.md (ADR Q6 riješen), model-prijetnji.md |
| EURe permit() (ERC-2612) | Korisnik može odobriti razdjelnik bez zasebne approve tx | architecture.md UX tok |
| Monerium SEPA rampa + Safe integracija | Ulazna rampa za fazu 1 je riješena | ekonomski-model.md |
| Monerium recover funkcija | Novi vektor — Monerium može premjestiti sredstva. Treba T16? | model-prijetnji.md |
| EURe V1→V2 dvostruki eventi | Koristiti ISKLJUČIVO V2 adresu za integraciju | architecture.md |

---

## 6. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Konkretan i akcijski — SDK primjer, specifične adrese, gas procjene | Dokumentarni — citira službene izvore, audit izvještaje, tokendizajn |
| **Prednost** | Odmah upotrebljiv za implementaciju | Sveobuhvatan kontekst, bolja pokrivenost alternativa |
| **Slabost** | Gas procjene bez citata | Nema SDK primjera koda |
| **Pouzdanost** | Visoka — većina tvrdnji provjerljiva | Visoka — svaka tvrdnja ima izvor |
