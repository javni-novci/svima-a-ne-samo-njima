# Križna provjera: ERC-4337 apstrakcija računa na Gnosis lancu

**Izvještaji:**
- [03a — Claude Opus 4.6](../reports/03a-erc4337-gnosis-claude.md)
- [03b — Perplexity](../reports/03b-erc4337-gnosis-perplexity.md)

**Prompt:** [03-erc4337-gnosis.md](../prompts/03-erc4337-gnosis.md)

---

## 1. Ukupna ocjena podudarnosti

**Ukupna podudarnost činjenica: 88/100**

Ovo je najviša podudarnost od svih dosadašnjih istraživanja. Oba izvora se slažu na gotovo svim ključnim činjenicama. Razlike su u dubini pokrivenosti i strateškim preporukama.

| Kategorija | Podudarnost | Obrazloženje |
|------------|-------------|--------------|
| EntryPoint adrese i verzije | 95/100 | Identične adrese za v0.6 i v0.7. Claude dodaje v0.8. |
| Bundleri na Gnosisu | 85/100 | Oba navode Pimlico, Biconomy, Particle, ZeroDev. Claude dodaje Etherspot Skandha. |
| Paymaster pružatelji | 85/100 | Oba navode Pimlico, Biconomy, Gelato, Particle, ZeroDev. Claude dodaje Etherspot/Arka i Candide. |
| Alchemy/StackUp status | 100/100 | Oba: Alchemy ne podržava Gnosis AA, StackUp ne/ugašen. |
| Safe{Core} Gas Station | 90/100 | Oba navode program. Claude: $250K+. Perplexity: $50K za Gnosis. |
| Vlastiti Paymaster implementacija | 90/100 | Oba navode eth-infinitism VerifyingPaymaster. Isti pristup. |
| Procjena troškova | 80/100 | Kompatibilni ali različiti parametri. Claude: ~$30 za 10K (1 gwei). Perplexity: ~$64 za 10K (2 gwei). |
| EOA vs Smart Account | 95/100 | Oba: ERC-4337 zahtijeva smart account. EOA ne može koristiti paymaster. |
| Meta-transakcije kao alternativa | 75/100 | Oba spominju. Perplexity puno detaljnija analiza (OpenGSN, EIP-2771, vlastiti relayer). |

---

## 2. Činjenice na kojima se oba izvora slažu (visoko povjerenje)

| # | Činjenica | Povjerenje |
|---|-----------|------------|
| F1 | EntryPoint v0.6: `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` na Gnosisu | Vrlo visoko |
| F2 | EntryPoint v0.7: `0x0000000071727De22E5E9d8BAf0edAc6f37da032` na Gnosisu | Vrlo visoko |
| F3 | Pimlico Alto Bundler aktivan na Gnosisu (v0.6 + v0.7) | Vrlo visoko |
| F4 | Biconomy podržava Gnosis (Bundler + Sponsorship PM + Token PM, v0.7 Nexus) | Vrlo visoko |
| F5 | Particle Network podržava Gnosis s bundlerom i paymasterom | Vrlo visoko |
| F6 | ZeroDev/Kernel podržava Gnosis (dashboard odabir) | Vrlo visoko |
| F7 | Alchemy NE podržava Gnosis AA | Vrlo visoko |
| F8 | StackUp NE podržava Gnosis (ugašen hosted servis) | Vrlo visoko |
| F9 | Safe smart accounti dostupni na Gnosisu | Vrlo visoko |
| F10 | Safe{Core} Gas Station nudi besplatne gas kredite na Gnosisu | Visoko |
| F11 | ERC-4337 zahtijeva smart account — EOA ne može koristiti paymaster | Vrlo visoko |
| F12 | Sponzoriranje 10K korisnika košta desetke xDAI (~$30-$64) | Visoko |
| F13 | eth-infinitism VerifyingPaymaster.sol je referentna implementacija | Vrlo visoko |
| F14 | Pravilo "1 mint po adresi" best practice: kombinacija on-chain require + off-chain paymaster provjera | Visoko |
| F15 | Gelato funkcionira kao relay/meta-tx sustav, ne klasični ERC-4337 | Visoko |

---

## 3. Činjenice koje je pronašao samo jedan izvor

### Samo Claude

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| C1 | EntryPoint v0.8 na Gnosisu (`0x4CE0051A...`) | Srednje-visoko | Nova verzija, specifična za Etherspot Skandha. |
| C2 | Etherspot Skandha bundler (v0.6/v0.7/v0.8, Nethermind) | Srednje-visoko | Relevantno jer Gnosis koristi Nethermind. Open-source. |
| C3 | Candide paymaster (InstaGas, v0.6/v0.7/v0.8) | Srednje | Dodatna opcija. |
| C4 | Pimlico podržava EURe kao ERC-20 paymaster token | Srednje-visoko | Jako relevantno — korisnik bi mogao platiti gas u EURe! |
| C5 | Particle Network bundler potpuno besplatan bez autentikacije | Srednje | Za prototipiranje. |
| C6 | Biconomy "Policies" tab za whitelisting specifičnih funkcija | Visoko | Direktno primjenjivo na naš use case. |

### Samo Perplexity

| # | Činjenica | Razina povjerenja | Komentar |
|---|-----------|-------------------|---------|
| P1 | OpenGSN službeno podržava Gnosis (RelayHub, Forwarder adrese) | Visoko | Relevantna alternativa za meta-transakcije. |
| P2 | MetaMask Smart Accounts Kit podržava Gnosis | Srednje | Novi proizvod, može biti koristan. |
| P3 | Pimlico besplatni plan: 1M credits/mj (~1.300 UserOps), bez mainnet paymastera | Visoko | Važno ograničenje — besplatni plan ne uključuje sponzorstvo na mainnetu! |
| P4 | ERC-4337 overhead po UserOp: ~40K gas u idealnom slučaju, ~50-150K realno | Visoko | Kvantitativna procjena overhead-a. |
| P5 | Gelato ima ~20% gas premium na besplatnom planu | Srednje-visoko | Skriveni trošak. |
| P6 | Detaljna usporedba ERC-4337 vs EIP-2771 vs vlastiti relayer | — | Strateška analiza, ne činjenica. |

---

## 4. Strateška razlika: ERC-4337 vs meta-transakcije

Oba izvora prepoznaju da za naš specifičan use case (jedna gasless funkcija, 1x po korisniku) postoji alternativa ERC-4337:

| Pristup | Claude stav | Perplexity stav |
|---------|------------|-----------------|
| **ERC-4337 (puni stack)** | Preporučuje Gelato Relay kao najbrži start, ERC-4337 s Pimlico/Biconomy za standardiziraniji put | Opisuje kao kompleksnije rješenje koje ima smisla ako planiraš AA za druge značajke |
| **EIP-2771 / OpenGSN** | Kratko spominje | Detaljno opisuje, navodi Gnosis adrese, preporučuje za jednostavnost |
| **Vlastiti relayer** | Ne spominje eksplicitno | Eksplicitno preporučuje kao "daleko najjednostavnije" za jednu funkciju |

**Ova razlika je arhitektonski značajna.** Vlastiti relayer ili EIP-2771 eliminira potrebu za smart accountom — korisnik ostaje EOA (obični MetaMask novčanik). To pojednostavljuje cijeli tok.

---

## 5. Zaključci za projekt

### Potvrđeno (možemo se osloniti)

- ERC-4337 infrastruktura na Gnosisu je **zrela** (4+ bundlera, 6+ paymastera).
- Troškovi su **zanemarivi** (~$30-$64 za 10K korisnika).
- Safe smart accounti rade na Gnosisu s punom AA podrškom.
- Safe{Core} Gas Station nudi besplatne gas kredite.
- Biconomy omogućuje precizno whitelisting funkcija za sponzorstvo.
- Pimlico podržava EURe kao gas token (ERC-20 paymaster).

### Nova arhitektonska odluka potrebna (ADR)

| Opcija | Složenost | Prednost | Nedostatak |
|--------|-----------|----------|------------|
| **A: Vlastiti relayer** | Niska | Najjednostavnije, korisnik ostaje EOA | Custom kôd, nema standarda |
| **B: EIP-2771 / OpenGSN** | Srednja | Standardizirano, korisnik ostaje EOA, OpenGSN na Gnosisu | Relayer infrastruktura |
| **C: ERC-4337 s Pimlico/Biconomy** | Visoka | Standardizirano, ekosustav alata, buduća proširivost | Zahtijeva smart account, kompleksniji UX |
| **D: Gelato Relay** | Srednja | Brz start, Safe integracija | Vendor lock-in, 20% gas premium |

**Ovo zahtijeva ADR s matricom bodovanja prema CLAUDE.md mandatu.**

### Utjecaj na arhitekturu

| Nalaz | Utjecaj | Dokument za ažuriranje |
|-------|---------|----------------------|
| Pimlico EURe paymaster | Korisnik bi mogao platiti gas u EURe — eliminira potrebu za xDAI | ekonomski-model.md, architecture.md |
| EOA vs Smart Account | Ako odaberemo meta-transakcije, korisnik ostaje EOA — jednostavniji UX ali manje AA značajki | architecture.md ADR potreban |
| Safe Gas Station krediti | Do $50K besplatnog gas sponzorstva na Gnosisu | ekonomski-model.md |
| Biconomy function whitelisting | Možemo ograničiti sponzorstvo na točno `claimCertiliaSBT()` | model-prijetnji.md (T6 ublažavanje) |

---

## 6. Metodološke napomene

| Aspekt | Claude Opus 4.6 | Perplexity |
|--------|-----------------|------------|
| **Pristup** | Široki pregled svih pružatelja s tehničkim detaljima | Duboka analiza s jasnim strateškim preporukama |
| **Prednost** | Više pružatelja pokriveno (Etherspot, Candide), specifičniji API detalji | Realnija procjena složenosti, uvažava meta-tx alternativu |
| **Slabost** | Podcjenjuje složenost ERC-4337 za jednostavan use case | Propušta neke pružatelje (Etherspot, Candide) |
| **Idealan za** | Tehničku evaluaciju pružatelja | Stratešku arhitektonsku odluku |
