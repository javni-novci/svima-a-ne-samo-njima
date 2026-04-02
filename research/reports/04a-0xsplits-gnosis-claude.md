# 0xSplits i razdjelnici — Claude Opus 4.6 izvještaj

**Prompt:** [04-0xsplits-gnosis.md](../prompts/04-0xsplits-gnosis.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# Razdjelnici plaćanja za EURe na Gnosis Chain

**0xSplits V2 je implementiran na Gnosis Chain (chain ID 100) i predstavlja optimalno rješenje za dijeljenje EURe uplata u omjeru 90/10.** Protokol u potpunosti podržava ERC-20 tokene, uključujući EURe (Monerium), ne naplaćuje nikakve protokolne naknade, auditiran je od strane nezavisnih sigurnosnih istraživača, a troškovi gas-a na Gnosis Chainu su zanemarivi — ispod $0.001 po transakciji. Uz dostupan JavaScript/TypeScript SDK (`@0xsplits/splits-sdk`), integracija s postojećim web3 aplikacijama je izravna. EURe je standardni ERC-20 token s **18 decimala, bez naknada na transfer**, ali s mogućnošću zamrzavanja adresa od strane Moneriuma kao reguliranog izdavatelja elektroničkog novca — što zahtijeva odgovarajuću obradu grešaka u pametnim ugovorima.

---

## 0xSplits je spreman za korištenje na Gnosis Chainu

Obje verzije protokola — **V1 i V2** — implementirane su na Gnosis Chain. V1 `SplitMain` ugovor dostupan je na adresi `0x2ed6c4B5dA6378c7897AC67Ba9e43102Feb694EE` i već sadrži EURe među **73+ ERC-20 tokena** kojima upravlja. V2 koristi CREATE2 determinističko postavljanje, što znači da su adrese ugovora identične na svim podržanim lancima.

**V2 je preporučena verzija za nove integracije** i nudi dva modela distribucije. **PullSplit** (zadani) prenosi sredstva u centralizirani `SplitsWarehouse` (ERC-6909 kompatibilan), odakle primatelji povlače svoju dobit. **PushSplit** šalje sredstva izravno na adrese primatelja u trenutku pozivanja `distribute()` funkcije — za slučaj korištenja 90/10 s dva primatelja, ovo je najjednostavniji pristup. Ako transfer prema jednom primatelju ne uspije (npr. zbog blackliste), sredstva se preusmjeravaju u Warehouse.

Za kreiranje 90/10 splita putem SDK-a, postupak je sljedeći:

```typescript
import { SplitV2Client } from '@0xsplits/splits-sdk'
const splitsClient = new SplitV2Client({ chainId: 100, publicClient, walletClient })
await splitsClient.createSplit({
  recipients: [
    { address: '0xCREATOR', percentAllocation: 90.0 },
    { address: '0xPLATFORMA', percentAllocation: 10.0 }
  ],
  distributorFeePercent: 0,
  splitType: SplitV2Type.Push
})
```

**Troškovi gas-a na Gnosis Chainu su praktički nula.** Kreiranje splita troši otprilike **100.000–150.000 gas jedinica**, a distribucija ERC-20 tokena s dva primatelja oko **80.000–120.000 gas jedinica**. Uz trenutne cijene gas-a na Gnosis Chainu (ispod 0.001 Gwei), to znači manje od **$0.001 po operaciji**. Gas se plaća u xDAI-u, čija je vrijednost vezana uz USD, pa nema volatilnosti troškova.

SDK ekosustav je zreo: `@0xsplits/splits-sdk` (v6.4.1+, ~1.600 tjednih preuzimanja, MIT licenca), `@0xsplits/splits-sdk-react` za React hookove, te `@0xsplits/splits-kit` za gotove UI komponente. Svi paketi koriste `viem` biblioteku.

**Revizije (auditi):** V1 je revidiran od strane **Shipyard Software**, a V2 od strane **Zacha Obronta** (Lead Senior Watson na Sherlocku, Lead Security Researcher na Spearbitu). Dodatno, Swapper/Oracle moduli prošli su Sherlock natjecateljsku reviziju. Svi ugovori su otvorenog koda, verificirani on-chain i **nenadogradivi** (non-upgradeable).

---

## EURe na Gnosis Chainu ima posebnosti koje valja razumjeti

EURe (Monerium EUR) na Gnosis Chainu koristi **dvije ugovorne adrese** zbog migracije V1→V2:

- **V2 (preporučena):** `0x420CA0f9B9b604cE0fd9C18EF134C705e5Fa3430` — ~18.3M EURe u optjecaju, ~30.500 vlasnika
- **V1 (zastarjela):** `0xcB444e90D8198415266c6a2724b7900fb12FC56E` — još funkcionalna, ali prosljeđuje sve transakcije na V2 i emitira **dvostruke Transfer evente**, što može zbuniti indeksere

Za nove integracije obavezno koristiti **V2 adresu**. Token je potpuno ERC-20 kompatibilan s **18 decimala**, podržava ERC-2612 `permit()` za gasless odobrenja, a V2 ugovor koristi standardne OpenZeppelin obrasce bez transfer hookova — **nema rizika od reentrancy napada** pri standardnim operacijama.

Ključna specifičnost EURe-a za payment splitter implementacije je **blacklist/freeze funkcionalnost**. Monerium kao regulirani izdavatelj elektroničkog novca (EMI, licenciran od Centralne banke Islanda, MiCA kompatibilan) ima mogućnost zamrzavanja adresa u dva slučaja: **potencijalna sigurnosna prijetnja** ili **zahtjev zakona, regulacije ili sudskog naloga** nadležnog tijela. U praksi, Monerium je 2022. godine zamrznuo **611.000 EURe** povezanih s hakom LCX burze. Validator ugovor (`BlacklistValidatorUpgradeable`) provjerava svaki transfer — ako je pošiljatelj ILI primatelj na crnoj listi, transakcija se **odbija (revert)**.

**Implikacija za payment splitter:** Ako bilo koja adresa u lancu distribucije (sam splitter ugovor, kreator ili platforma) bude stavljena na crnu listu, transfer prema toj adresi neće uspjeti. 0xSplits V2 PushSplit ovo djelomično rješava preusmjeravanjem neuspjelih transfera u Warehouse. Za custom implementacije, **obavezan je try/catch ili pull obrazac**.

Monerium nudi potpunu **SEPA on-ramp integraciju** za Gnosis Chain: korisnik dobiva osobni Web3 IBAN, šalje EUR putem SEPA-e, i EURe se automatski minta na povezanu adresu. Off-ramp funkcionira obrnuto — potpis poruke iz walleta aktivira burn EURe-a i slanje EUR-a na bankovni račun. API (`@monerium/sdk`) koristi OAuth2 autentifikaciju i podržava Safe wallete putem ERC-1271. **Integracija s Gnosis Safe-om je duboka** — Monerium je službeni partner Safe-a, s integracijom u Safe{Core} SDK putem "Onramp Kit-a".

---

## Alternativni protokoli nude ograničene prednosti

**Superfluid** je implementiran na Gnosis Chainu i nudi **Instant Distribution Agreement (IDA/GDA)** koji podržava proporcionalno dijeljenje s fiksnim udjelima — dodjelite 9 jedinica kreatoru i 1 jedinicu platformi, te aktivirate instantnu distribuciju. Gas trošak je fiksan neovisno o broju primatelja. Međutim, Superfluid zahtijeva **wrapping ERC-20 tokena u Super Tokene**, što dodaje dodatni korak složenosti. Protokol je relevantan ako platforma planira i streaming plaćanja (npr. pretplate), ali za jednostavno 90/10 dijeljenje predstavlja nepotrebnu kompleksnost.

**Sablier** je prisutan na Gnosis Chainu, ali je dizajniran za vremenski baziranu distribuciju (vesting, payroll streaming) — **nije prikladan za instantno proporcionalno dijeljenje uplata**.

**Drips (bivši Radicle Drips)** ima odličan Splits modul s fiksnim omjerima, ali **nije implementiran na Gnosis Chainu** — dostupan je samo na Ethereum mainnetu, Sepolia testnetu i Filecoinu.

**OpenZeppelin PaymentSplitter** je najjednostavnija opcija za self-deployment: konstruktor prima niz adresa i udjela (`[kreator, trezor], [90, 10]`), koristi pull obrazac, i podržava ERC-20 tokene od verzije v4.6. Potpuno je auditiran kao dio OpenZeppelin biblioteke. **Međutim, uklonjen je iz OpenZeppelin Contracts v5.0** — kod iz v4.9.6 je stabilan i funkcionalan, ali više se ne održava. Za nove projekte koji žele ostati na najnovijim verzijama OZ biblioteke, ovo zahtijeva kopiranje koda.

---

## Custom splitter zahtijeva minimalan napor

Ako postoje specifični razlozi za izradu vlastitog rješenja (npr. posebna logika za EURe blacklist handling), minimalni ERC-20 splitter za dva primatelja zahtijeva **30-40 linija Solidity koda**. Ključne komponente su: `SafeERC20` za sigurne transfere, `ReentrancyGuard` za zaštitu od reentrancy napada, `immutable` varijable za adrese primatelja i token, te izračun udjela u **basis pointima** (9000/1000 od 10000).

Za rounding pravilnost, prvo se izračuna manji udio (`balance * 1000 / 10000`), a zatim se ostatak dodijeli većem primatelju (`balance - treasuryAmount`), čime se eliminira zaostali "dust". Funkciju `distribute()` trebao bi moći pozvati **bilo tko** — uključujući botove ili same primatelje — bez potrebe za access controlom.

Za platforme s više kreatora preporučuje se **factory pattern s EIP-1167 minimal proxy klonovima**. Svaki klon troši samo ~**45.000–60.000 gas jedinica** za deployment (nasuprot 500.000–800.000 za puni ugovor), iako su na Gnosis Chainu oba troška ispod $0.001. OpenZeppelin `Clones.sol` biblioteka pruža gotovu implementaciju.

**Procjena razvoja:** iskusni Solidity developer može implementirati, testirati i deployati custom splitter u **1-2 radna dana**. Foundry je preporučeni testing framework, s mogućnošću forka Gnosis mainnet-a za testiranje s pravim EURe tokenom. Formalni audit zahtijeva dodatnih 2-4 tjedna i $5.000–$20.000+.

---

## Usporedna tablica svih opcija

| Opcija | Gnosis podrška | ERC-20 podrška | Auditiran | Gas (kreiranje) | Gas (distribucija) | Složenost integracije |
|--------|---------------|----------------|-----------|-----------------|--------------------|-----------------------|
| **0xSplits V2** | ✅ Da | ✅ Potpuna | ✅ Zach Obront (Spearbit) | ~100K–150K gas (<$0.001) | ~80K–120K gas (<$0.001) | **Niska** — SDK + UI |
| **0xSplits V1** | ✅ Da | ✅ Potpuna | ✅ Shipyard Software | ~80K–120K gas (<$0.001) | ~70K–80K gas (<$0.001) | Niska-Srednja |
| **OZ PaymentSplitter** | ✅ Self-deploy | ✅ (v4.6+) | ✅ OpenZeppelin auditi | ~300K gas (<$0.001) | ~50K–80K gas (<$0.001) | **Vrlo niska** — ali uklonjen iz v5 |
| **Superfluid IDA** | ✅ Da | ✅ Uz wrapping | ✅ Višestruki auditi | ~200K+ gas (<$0.001) | ~100K+ gas (<$0.001) | Srednja-Visoka |
| **Sablier** | ✅ Da | ✅ Da | ✅ Da | N/P | N/P | **Nije primjenjiv** — samo streaming |
| **Drips** | ❌ Ne | ✅ Da | ✅ Da | N/P | N/P | **Nije dostupan** na Gnosis |
| **Custom splitter** | ✅ Self-deploy | ✅ Da | ❌ Zahtijeva audit | ~500K–800K gas (<$0.001) | ~80K–120K gas (<$0.001) | Niska (30-40 linija koda) |
| **Custom + EIP-1167 klonovi** | ✅ Self-deploy | ✅ Da | ❌ Zahtijeva audit | ~45K–60K gas/klon (<$0.001) | ~80K–120K gas (<$0.001) | Srednja (factory pattern) |

---

## Zaključak i preporuka

**0xSplits V2 PushSplit je optimalan izbor** za dijeljenje EURe uplata 90/10 na Gnosis Chainu. Kombinacija dokazanog protokola bez protokolnih naknada, gotovog SDK-a, zanemarivog gas troška i neovisnog audita čini ga nadmoćnim u odnosu na alternative. Jedini aspekt koji zahtijeva pažnju jest **EURe blacklist mehanizam** — PushSplit automatski preusmjerava neuspjele transfere u Warehouse, što pruža ugrađenu zaštitu.

Za platforme koje žele potpunu kontrolu bez ovisnosti o vanjskim protokolima, **custom EIP-1167 factory splitter** s OpenZeppelin bibliotekama predstavlja razumnu alternativu koja se može implementirati u 1-2 dana. Ključni sigurnosni obrasci — SafeERC20, ReentrancyGuard, immutable varijable i basis point aritmetika — dovoljni su za siguran rad.

Posebno vrijedi istaknuti da **Monerium-Safe integracija** omogućava izravnu SEPA on-ramp na Safe trezor platforme, čime se zatvara cijeli platni ciklus: EUR → SEPA → EURe (mint na Gnosis) → Payment Splitter (90/10) → kreator + platforma → eventualni off-ramp natrag u EUR. Cijena cjelokupnog on-chain dijela tog procesa na Gnosis Chainu je ispod jednog centa.