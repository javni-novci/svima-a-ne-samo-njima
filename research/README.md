# Dubinska istraživanja

Svaka tema istražena iz dva (ili više) neovisna izvora radi križne provjere činjenica. Nakon što su oba izvora gotova, izrađuje se usporedba u `compare/` mapi.

## Status

| # | Tema | Prioritet | Prompt | Claude Opus 4.6 | Perplexity | Usporedba |
|---|------|-----------|--------|-----------------|------------|-----------|
| 01 | Certilia Mobile.id specifikacije | BLOKIRAJUĆE | [prompt](prompts/01-certilia-mobileid.md) | [gotovo](reports/01a-certilia-mobileid-claude.md) | [gotovo](reports/01b-certilia-mobileid-perplexity.md) | [82/100](compare/01-certilia-mobileid.md) |
| 02 | Lančana RSA/P-256 verifikacija | BLOKIRAJUĆE | [prompt](prompts/02-lancana-kriptografija-gnosis.md) | [gotovo](reports/02a-lancana-kriptografija-gnosis-claude.md) | [gotovo](reports/02b-lancana-kriptografija-gnosis-perplexity.md) | [62/100](compare/02-lancana-kriptografija-gnosis.md) |
| 03 | ERC-4337 Paymaster na Gnosisu | BLOKIRAJUĆE | [prompt](prompts/03-erc4337-gnosis.md) | [gotovo](reports/03a-erc4337-gnosis-claude.md) | [gotovo](reports/03b-erc4337-gnosis-perplexity.md) | [88/100](compare/03-erc4337-gnosis.md) |
| 04 | 0xSplits i razdjelnici | VAŽNO | [prompt](prompts/04-0xsplits-gnosis.md) | [gotovo](reports/04a-0xsplits-gnosis-claude.md) | [gotovo](reports/04b-0xsplits-gnosis-perplexity.md) | [91/100](compare/04-0xsplits-gnosis.md) |
| 05 | eIDAS 2.0 / EUDI novčanik | VAŽNO | [prompt](prompts/05-eidas2-eudi.md) | [gotovo](reports/05a-eidas2-eudi-claude.md) | [gotovo](reports/05b-eidas2-eudi-perplexity.md) | [85/100](compare/05-eidas2-eudi.md) |
| 06 | GDPR i hashirani identifikatori | VAŽNO | [prompt](prompts/06-gdpr-hash-identifikatori.md) | [gotovo](reports/06a-gdpr-hash-claude.md) | [gotovo](reports/06b-gdpr-hash-perplexity.md) | [93/100](compare/06-gdpr-hash.md) |
| 07 | Monerium EURe na Gnosisu | VAŽNO | [prompt](prompts/07-monerium-eure-gnosis.md) | [gotovo](reports/07a-monerium-eure-claude.md) | [gotovo](reports/07b-monerium-eure-perplexity.md) | [92/100](compare/07-monerium-eure.md) |
| 08 | Izvori financiranja projekta | STRATEŠKO | [prompt](prompts/08-financiranje-projekt.md) | [čeka](reports/08a-financiranje-claude.md) | [čeka](reports/08b-financiranje-perplexity.md) | — |

## Struktura

```
research/
├── README.md              ← ovaj indeks
├── prompts/               ← promptovi za pokretanje istraživanja
│   └── XX-tema.md
├── reports/               ← sirovi izvještaji iz AI alata
│   ├── XXa-tema-claude.md
│   ├── XXb-tema-perplexity.md
│   └── XXc-tema-[drugi-alat].md    ← opcionalno, ako koristiš više alata
└── compare/               ← križne provjere i ocjene podudarnosti
    ├── PREDLOZAK.md       ← predložak za nove usporedbe
    └── XX-tema.md
```

## Postupak

1. Kopiraj sadržaj prompta u odgovarajući AI alat
2. Zalijepi rezultat u odgovarajuću report datoteku (zamijeni HTML komentar)
3. Ažuriraj datum i status u zaglavlju reporta
4. Ponovi za drugi (i treći, ako želiš) alat
5. Kada su svi izvori za istu temu gotovi, napravi usporedbu u `compare/` koristeći predložak
6. Ažuriraj tablicu iznad: zamijeni `čeka` s `gotovo` i dodaj ocjenu podudarnosti

## Dosadašnji rezultati križnih provjera

| # | Tema | Podudarnost | Ključni nalaz |
|---|------|-------------|---------------|
| 01 | Certilia Mobile.id | **82/100** | Slažu se na temeljnim činjenicama (AKD, OIDC, OIB u JWT-u, cijena, eIDAS). Razilaze se na tehničkim detaljima (algoritam potpisa, nonce podrška) koje Claude izvodi iz WSO2 IS identifikacije a Perplexity ostavlja nepoznatima. |
| 02 | Lančana RSA/P-256 verifikacija | **62/100** | **Kritično proturječje:** RSA-2048 gas — Claude kaže ~25-35K, Perplexity kaže ~1,5-2,5M (50x razlika). Perplexity pouzdaniji (citira benchmarkove). Oba se slažu: izvedivo na Gnosisu, P-256 jeftiniji, ZK alternativa održiva. |
| 03 | ERC-4337 na Gnosisu | **88/100** | Najviša podudarnost. Oba potvrđuju zrelu infrastrukturu (4+ bundlera, 6+ paymastera). Ključna strateška razlika: Perplexity preporučuje razmotriti meta-transakcije (EIP-2771/OpenGSN) kao jednostavniju alternativu za naš use case. |
| 04 | 0xSplits i razdjelnici | **91/100** | Dosad najviša. 0xSplits V2 PushSplit potvrđen na Gnosisu, revidiran, EURe kompatibilan, SDK zreo. EURe blacklist fallback u Warehouse rješava T13. ADR Q6 može se zatvoriti. |
| 05 | eIDAS 2.0 / EUDI | **85/100** | 20 zajedničkih činjenica. P-256 primarni algoritam (potvrđuje naš dizajn). Nema EUDI nullifiera — moramo sami. SD-JWT-VC format za online (ne klasični JWT). Certilia koegzistira. ARF verzija nepotvrđena (Claude v2.8.0 vs Perplexity v1.2.0). |
| 06 | GDPR i hashirani identifikatori | **93/100** | **JEDNOGLASNI:** hash OIB-a = osobni podatak. DPIA obvezan. Privola neprihvatljiva za neizbrisivi blockchain. ZK nullifier značajno usklađeniji. Fundamentalno mijenja prioritete: nullifier mora biti u fazi 1, ne fazi 2. |
| 07 | Monerium EURe | **92/100** | 25 zajedničkih činjenica. EURe optimalan za EUR platformu. Blacklist potvrđen (LCX presedan). 0xSplits V2 kompatibilan. SEPA rampa bez naknada. Gnosis Pay battle-tested. Pausable = novi rizik. |
