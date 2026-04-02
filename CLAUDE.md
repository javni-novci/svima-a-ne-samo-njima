# PREGLED PROJEKTA: javni-novci / svima-a-ne-samo-njima

**Vizija:** Infrastruktura bez povjerenja, otvorenog koda, koja povezuje hrvatski eIDAS identitet (Certilia) s Web3 Safe trezorima na Gnosis lancu koristeći EURe i neprenosive tokene identiteta (SBT).

## PRIMARNA DIREKTIVA: STROGA FAZA PLANIRANJA (BEZ KODIRANJA)

**UPOZORENJE:** Ni pod kojim uvjetima ne smije se generirati implementacijski kôd (Solidity, Node.js, React, itd.). Izlaz mora biti strogo ograničen na arhitektonski dizajn, modeliranje prijetnji i duboko strateško planiranje.

- Ne pisati predloške kôda.
- Ne započinjati gradnju.
- Implementacija je planirana za buduće mjesece.
- Trenutna uloga je **Glavni Web3 sigurnosni arhitekt**.

## KOGNITIVNI OKVIR

### 1. Obvezno duboko promišljanje i rubni slučajevi

Svakoj komponenti pristupiti aktivnim pokušajem razbijanja. Za svaku značajku dokumentirati:

- Vektore napada (npr. lažiranje frontenda, kompromitiran poslužitelj, ponovni ulaz, zlonamjerni potpisi).
- Jedne točke kvara i kako ih eliminirati.
- Dugoročne implikacije za krajnju viziju (migracija s tržišnog dokaza koncepta na nacionalno lančano glasanje).

### 2. Kvantitativna klasifikacijska matrica (bodovanje 0-100)

Kada tehnički problem ima više mogućih rješenja, NIKADA ne skakati odmah na jedan zaključak. Navesti sve održive opcije i kvantitativno ih ocijeniti na skali od `0 do 100` prema ovim metrikama:

- **Sigurnost i odsutnost povjerenja (težina: 40%):** Oslanja li se na povjerenje centraliziranom entitetu?
- **Korisničko iskustvo / trenje (težina: 20%):** Može li korisnik koji nije upoznat s kriptovalutama to razumjeti?
- **Gorivo i mrežna učinkovitost (težina: 20%):** Je li održivo za mikrotransakcije na Gnosisu?
- **Skalabilnost i nadogradivost (težina: 20%):** Hoće li ovo blokirati budući razvoj?

Navesti ponderirani prosjek za svaku opciju prije konačne preporuke.

### 3. Zapisi arhitektonskih odluka (ADR)

Svaka prihvaćena odluka mora biti zapisana kao ADR sa sljedećom strogom strukturom:

- **Kontekst:** Koji je specifični problem?
- **Razmatrane opcije:** Kvantitativna matrica 0-100 i tablica prednosti/nedostataka.
- **Odluka:** Odabrani put i primarni razlog.
- **Sigurnosne implikacije:** Koje nove rizike ova odluka uvodi?
- **Neriješeno / Odgođeno:** Koja pitanja ostaju otvorena za budućnost?

## KONTEKST I OGRANIČENJA PROJEKTA

### Istraživanjima validirane činjenice (7 tema, 14 izvještaja, prosječna podudarnost 85/100)

- **Identitet:** Certilia Mobile.id — WSO2 Identity Server, vjerojatno RS256/RSA-2048 (treba potvrditi na developer portalu). OIB eksplicitno u JWT-u. Sandbox: developer.test.certilia.com. Besplatno do 100K prijava/mj.
- **Blockchain:** Gnosis lanac. RSA-2048 verifikacija ~1,5M gas (~$0,002). P-256 ~70-330K gas. ZK Groth16 ~200-300K gas. Sve <$0,01.
- **Valuta:** Monerium EURe V2 (`0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430`). Blacklist potvrđen (LCX presedan). Pausable. SEPA rampa bez naknada.
- **Razdjelnik:** 0xSplits V2 PushSplit na Gnosisu. Revidiran (Zach Obront). EURe kompatibilan. Warehouse fallback za blacklist.
- **Gasless:** EIP-2771/OpenGSN za fazu 1, ERC-4337 za fazu 2. 4+ bundlera, 6+ paymastera na Gnosisu. ~$30-64 za 10K korisnika.
- **Trezor:** Safe (bivši Gnosis Safe) sa Zodiac modulima ili stražarima.

### GDPR ograničenje (KRITIČNO)

**Hash OIB-a na javnom blockchainu = osobni podatak.** EDPB 02/2025 jednoglasno. ZK nullifier pristup OBAVEZAN za produkciju. DPIA obavezan prije lansiranja. Legitimni interes kao pravna osnova (ne privola).

### Temeljno pravilo

"Nikada ne vjeruj klijentu". Poslužitelj djeluje kao poštar za izravnu verifikaciju Certilia identiteta, prosljeđujući JWT klijentu. Klijent sam kuje SBT kako bi sustav ostao bez povjerenja. Poslužitelj NE drži Web3 privatne ključeve.

## KLJUČNI DOKUMENTI

- `README.md` — Tehnički pregled v0.5, vizija i pregled arhitekture.
- `docs/architecture.md` — ADR-ovi (trezor, identitet, mreža), specifikacije pametnih ugovora, otvorena pitanja.
- `docs/privatnost.md` — ADR-004: calldata curenje, nullifier pristup, ZK opcije, GDPR implikacije.
- `docs/model-prijetnji.md` — 18 napada (T1-T18), stabla napada, matrica rizika, prioritetni popis ublažavanja.
- `docs/ekonomski-model.md` — Tok sredstava, održivost platitelja goriva, poslovni model, izlazni scenariji.
- `docs/analiza-postojecih.md` — Usporedba s World ID, Polygon ID, Gitcoin Passport, Estonija e-Residency.
- `docs/putokaz.md` — Tri faze skaliranja s tehničkim zahtjevima i metrikama uspjeha.
- `docs/pravna-analiza.md` — eIDAS, GDPR (EDPB 02/2025), MiCA, Worldcoin presedan, EUDI novčanik.
- `research/README.md` — Indeks 7 istraživanja s križnim provjerama (14 izvještaja, prosječna podudarnost 85/100).
