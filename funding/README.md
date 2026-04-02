# Financiranje projekta

## Aktivne prilike

### Safe Foundation — Spontaneous Grants

| Detalj | Vrijednost |
|--------|-----------|
| **URL** | https://safefoundation.org/grants |
| **Kontakt** | grants@safefoundation.org |
| **Tip** | Otvoreni prijedlozi, tekuća prijava |
| **Status** | Priprema prijedloga |

**Što traže:**
- Jasan doprinos Safe ekosustavu pametnih računa
- Dobro definiran opseg, isporučevine i vremenski okvir
- Tim ili pojedinac s relevantnim iskustvom
- Otvoreni kôd preferiran (ali ne strogo obavezan)

**Kako se prijaviti:**
1. Poslati prijedlog na grants@safefoundation.org
2. Uključiti: opis projekta, opseg i prekretnice, razrada proračuna, pozadina tima, očekivani utjecaj
3. Zaklada interno pregledava, odgovor unutar 4 tjedna
4. Odobreni grantovi se isplaćuju po prekretnicama vezanim za isporučevine

**Zašto smo dobar kandidat:**
- Koristimo Safe kao platformski trezor (višepotpisni 2/3)
- IIdentityVerifier apstrakcijski sloj proširuje Safe ekosustav
- SBT integracija s Zodiac modulima/stražarima
- 0xSplits V2 integriran sa Safe za raspodjelu sredstava
- Certilia eIDAS identitet → Safe upravljanje = novi use case za Safe ekosustav
- Otvoreni kôd, javna dokumentacija

### Safe Foundation — Ventures

| Detalj | Vrijednost |
|--------|-----------|
| **URL** | https://safefoundation.org/ventures |
| **Kontakt** | investments@safefoundation.org |
| **Tip** | Ulaganja u projekte koji grade na Safe-u |
| **Status** | Razmotriti nakon faze 1a |

**Napomena:** Ventures je za projekte s poslovnim modelom i timom. Relevantno tek kada imamo funkcionalni dokaz koncepta i korisničku bazu.

### Safe{Core} Gas Station

| Detalj | Vrijednost |
|--------|-----------|
| **URL** | https://safefoundation.org/blog/making-every-app-gasless-launching-the-safe-core-multi-chain-gas-station |
| **Tip** | Besplatni gas krediti za projekte koji koriste Safe + ERC-4337 + Pimlico |
| **Vrijednost** | Do $50.000 na Gnosis lancu |
| **Status** | Prijaviti se kada imamo deployani ugovor |

---

## Priprema prijedloga za Safe Grant

### Nacrt strukture prijedloga

```
Naslov: Certilia eIDAS Identity × Safe Treasury — On-chain Identity for EU Citizens

1. Opis projekta
   - Povezivanje hrvatskog eIDAS identiteta (Certilia) s Safe trezorima
   - ZK nullifier pristup za GDPR-usklađenu Sybil zaštitu
   - Podcast tržište kao dokaz koncepta, vizija nacionalnog glasanja

2. Doprinos Safe ekosustavu
   - IIdentityVerifier modul za Safe — proširuje Safe na eIDAS identitete
   - Zodiac Guard koji provjerava SBT prije isplate
   - 0xSplits + Safe integracija za automatsku raspodjelu EURe
   - Otvoreni kôd koji bilo tko u EU može koristiti

3. Opseg i prekretnice
   Prekretnica 1 (mjesec 1-2): Certilia OIDC integracija + CertiliaSBT ugovor
   Prekretnica 2 (mjesec 3-4): ZK nullifier krug + verifikator na Gnosisu
   Prekretnica 3 (mjesec 5-6): Frontend + 0xSplits + Safe integracija
   Prekretnica 4 (mjesec 7-8): Testiranje s korisnicima, revizija, lansiranje

4. Razrada proračuna
   [Popuniti prema potrebama tima]

5. Tim
   [Popuniti]

6. Očekivani utjecaj
   - Prvi EU projekt koji povezuje eIDAS s Safe trezorima
   - GDPR-usklađeni model za lančani identitet
   - Predložak za druge EU države (eIDAS je paneuropski)
```

---

## Buduće prilike za istraživanje

| Izvor | Tip | Relevantnost | URL |
|-------|-----|-------------|-----|
| EU Horizon Europe | Istraživački grant | Digitalna transformacija, e-uprava | horizon-europe.eu |
| EU Digital Europe Programme | Infrastrukturni grant | Blockchain, digitalni identitet | digital-strategy.ec.europa.eu |
| Gnosis DAO | Ekosistemski grant | Projekti na Gnosis lancu | gnosis.io |
| Ethereum Foundation | Istraživački grant | ZK, privatnost, identitet | ethereum.org/grants |
| UNICEF Innovation Fund | Razvojni grant | Otvoreni kôd, javno dobro | unicef.org/innovation |
| Hrvatski fondovi | Nacionalni grant | Digitalna transformacija | strukturnifondovi.hr |

---

## GitHub Sponsors

GitHub Sponsors je uključen putem `.github/FUNDING.yml`. "Sponsor" gumb pojavljuje se na vrhu repozitorija.

Za aktivaciju GitHub Sponsors za organizaciju `javni-novci`:
1. Idi na https://github.com/sponsors/javni-novci
2. Prati upute za postavljanje profila sponzora
3. Definiraj razine sponzorstva (npr. $5/mj, $25/mj, $100/mj)
