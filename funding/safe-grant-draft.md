# Safe Foundation Grant — Nacrt prijedloga

**Status:** Nacrt — zahtijeva popunjavanje tima i proračuna
**Kontakt:** grants@safefoundation.org

---

## 1. Opis projekta

**Certilia eIDAS Identity x Safe Treasury** — otvorena infrastruktura koja povezuje hrvatski eIDAS digitalni identitet (Certilia Mobile.id) s Safe pametnim računima na Gnosis lancu, koristeći ZK nullifier pristup za GDPR-usklađenu Sybil zaštitu i EURe (Monerium) za EUR-denominirane transakcije.

### Problem

Blockchain rješava transparentnost, ali pati od Sybil napada (jedna osoba = N novčanika). Državni identitetski sustavi (eIDAS) imaju savršen identitet, ali su zatvoreni. Ne postoji most između ova dva svijeta koji poštuje GDPR.

### Rješenje

ZK nullifier pristup: korisnik dokazuje "ja sam jedinstven građanin" bez otkrivanja TKO je. Pametni ugovor na Gnosisu verificira ZK dokaz, kuje neprenosivi token identiteta (SBT), a Safe trezor prepoznaje SBT za upravljanje sredstvima.

### Dokaz koncepta

Podcast tržište gdje kreatori primaju EURe automatski (0xSplits V2 PushSplit), a samo verificirani građani mogu sudjelovati. Platforma koristi Safe kao operativni trezor.

### Dugoročna vizija

Ista infrastruktura za transparentno upravljanje javnim proračunom i elektroničko glasanje.

---

## 2. Doprinos Safe ekosustavu

| Komponenta | Doprinos |
|------------|---------|
| **IIdentityVerifier modul** | Novi Safe modul koji povezuje eIDAS identitet s Safe upravljanjem. Proširivo na sve EU države (eIDAS je paneuropski). |
| **Zodiac Guard za SBT** | Guard koji provjerava neprenosivi token identiteta prije odobravanja Safe transakcija. Samo verificirani građani mogu inicirati isplate. |
| **0xSplits + Safe integracija** | Dokumentiran tok: EURe uplata → 0xSplits V2 PushSplit → 90% kreator + 10% Safe trezor. Predložak za bilo koje tržište. |
| **GDPR-usklađeni identitet za Safe** | Prvi Safe modul koji poštuje GDPR putem ZK nullifiera. Predložak za EU projekte koji koriste Safe. |
| **EIP-2771 gasless integracija** | Dokumentiran tok za gasless kovanje SBT-a putem OpenGSN na Gnosisu, kompatibilan sa Safe. |

### Otvoreni kôd

Sav kôd pod MIT licencom. Dokumentacija na hrvatskom i engleskom. Arhitektonske odluke javno dokumentirane (7 ADR-ova, 18 napada u modelu prijetnji, 7 dubinskih istraživanja s križnom provjerom).

---

## 3. Opseg i prekretnice

| Prekretnica | Razdoblje | Isporučevine |
|-------------|-----------|-------------|
| **P1: Certilia integracija** | Mjesec 1-2 | OIDC tok s Certilijom (WSO2 IS), CertiliaSBT ugovor na Gnosis testnetu, IIdentityVerifier v1 (JWT verifikacija) |
| **P2: ZK nullifier** | Mjesec 3-4 | ZK krug za JWT verifikaciju (Circom/Halo2), Groth16 verifikator na Gnosisu, IIdentityVerifier v2 (nullifier), DPIA dokument |
| **P3: Tržište + Safe** | Mjesec 5-6 | Frontend aplikacija, 0xSplits integracija, Safe trezor s Zodiac Guard-om, EIP-2771 gasless kovanje |
| **P4: Lansiranje** | Mjesec 7-8 | Korisničko testiranje, sigurnosna revizija, produkcijsko lansiranje na Gnosis mainnetu |

---

## 4. Razrada proračuna

| Stavka | Procjena | Napomena |
|--------|----------|---------|
| Razvoj (pametni ugovori) | [POPUNITI] | CertiliaSBT, IIdentityVerifier, ZK krug |
| Razvoj (pozadinski sustav) | [POPUNITI] | Node.js OIDC poštar, relayer |
| Razvoj (frontend) | [POPUNITI] | Web aplikacija s novčanikom |
| Sigurnosna revizija | 10.000-30.000 USD | Profesionalni audit pametnih ugovora |
| DPIA i pravne konzultacije | 2.000-5.000 EUR | GDPR, AZOP savjetovanje |
| Infrastruktura (12 mjeseci) | 2.000-5.000 USD | Poslužitelj, RPC čvorovi, Paymaster fond |
| **Ukupno** | **[POPUNITI]** | |

---

## 5. Tim

[POPUNITI — ime, uloga, relevantno iskustvo, linkovi]

---

## 6. Očekivani utjecaj

- **Kratkoročni:** Funkcionalno podcast tržište s eIDAS identitetom na Gnosisu. Dokaz da Safe može upravljati sredstvima verificiranih EU građana.
- **Srednjoročni:** Predložak koji bilo koja EU država može prilagoditi za vlastiti eIDAS identitet + Safe trezor.
- **Dugoročni:** Infrastruktura za transparentno upravljanje javnim proračunom i elektroničko glasanje — sve građeno na Safe ekosustavu.

### Metrike uspjeha

| Metrika | Cilj (12 mjeseci) |
|---------|-------------------|
| Verificirani korisnici (CERTILIA SBT) | 100+ |
| Aktivni kreatori | 10+ |
| EURe promet kroz 0xSplits | 1.000+ EUR |
| Sigurnosni incidenti | 0 |
| GitHub zvjezdice | 50+ |
| Fork-ovi | 5+ |

---

## 7. Postojeća dokumentacija

Projekt već ima opsežnu arhitektonsku dokumentaciju:

- 7 zapisa arhitektonskih odluka (ADR)
- 18 identificiranih napada u modelu prijetnji
- 7 dubinskih istraživanja s križnom provjerom (prosječna podudarnost 85/100)
- GDPR analiza s EDPB 02/2025 smjernicama
- Putokaz s fazama 0/1a/1b/2/3

Sve javno dostupno na: https://github.com/javni-novci/svima-a-ne-samo-njima
