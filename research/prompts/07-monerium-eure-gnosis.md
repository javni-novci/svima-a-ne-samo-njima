# Deep Research: Monerium EURe na Gnosis lancu

## Kontekst za istraživača

Koristim EURe (Monerium) kao primarnu valutu na Gnosis lancu za podcast tržište. EURe je MiCA-usklađeni stabilni token vezan za euro. Trebam razumjeti tehničke mogućnosti i regulatorna ograničenja.

## Pitanja na koja trebam odgovore

### 1. Tehnička specifikacija na Gnosisu

- Koja je adresa EURe ugovora na Gnosis lancu (chain ID 100)?
- Je li EURe standardni ERC-20? Ima li proširenja?
- Ima li EURe `mint()`, `burn()`, `freeze()`, `blacklist()` funkcije u ugovoru?
- Tko ima ovlast pozivati privilegirane funkcije (freeze, blacklist)?
- Je li EURe ugovor nadogradiv (proxy)?
- Koliki je ukupni opticaj (total supply) EURe na Gnosisu?
- Koji je dnevni volumen transakcija EURe na Gnosisu?

### 2. Monerium API i ulazna rampa

- Postoji li Monerium API za programsku konverziju SEPA → EURe?
- Može li korisnik izravno poslati SEPA uplatu i primiti EURe na Gnosis adresi?
- Koji je KYC postupak za Monerium? Koliko traje?
- Postoji li sandbox/testno okruženje za Monerium API?
- Kolika je naknada za SEPA → EURe konverziju?
- Kolika je naknada za EURe → SEPA konverziju (povlačenje u banku)?
- Postoji li Monerium SDK za JavaScript/TypeScript?

### 3. Zamrzavanje i regulatorne ovlasti

- Može li Monerium zamrznuti EURe na specifičnoj adresi? Pod kojim uvjetima?
- Je li Monerium obvezan zamrznuti sredstva na zahtjev regulatora (HANFA, ECB, nacionalni sud)?
- Postoji li javno dostupan popis zamrznutih adresa?
- Koliko je zamrzavanja bilo do sada na Gnosisu?
- Ako Monerium zamrzne adresu razdjelnika plaćanja, što se događa s tokovima sredstava?
- MiCA (Regulation 2023/1114) — koji specifični članci daju pravo zamrzavanja?

### 4. Integracija s Safe i pametnim ugovorima

- Može li Safe primati i slati EURe na Gnosisu bez posebne konfiguracije?
- Funkcionira li EURe s `approve()` + `transferFrom()` uzorkom (potrebno za razdjelnike plaćanja)?
- Postoje li poznati problemi s EURe i pametnim ugovorima?
- Postoji li Safe aplikacija (Safe App) za Monerium?

### 5. Alternative EURe-u

- Koji drugi EUR stabilni tokeni postoje na Gnosis lancu?
  - EURS (Stasis)?
  - agEUR (Angle Protocol)?
  - cEUR?
- Postoji li USDC ili DAI na Gnosis lancu kao rezervna opcija?
- Usporedba: regulatorni status, likvidnost, mogućnost zamrzavanja

### 6. Gnosis Pay i Safe{Wallet}

- Gnosis Pay (Visa kartica) — koristi li EURe? Kako?
- Safe{Wallet} na Gnosisu — ima li posebnu Monerium integraciju?
- Postoji li veza između Gnosis Pay infrastrukture i naše namjene?

## Format odgovora

1. **Tehničke činjenice** — adresa ugovora, ABI, funkcije
2. **Regulatorna situacija** — zamrzavanje, MiCA, KYC
3. **Integracijske mogućnosti** — API, SDK, Safe
4. **Rizici** — zamrzavanje, ovisnost o Moneriumu
5. **Alternative** — tablica usporedbe stabilnih tokena na Gnosisu

Sav izlaz na **hrvatskom jeziku**.
