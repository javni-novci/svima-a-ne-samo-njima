# Analiza postojećih rješenja

**Status:** Istraživačko
**Svrha:** Pozicioniranje projekta u kontekstu postojećih rješenja za decentralizirani identitet

---

## 1. Pregled

Nekoliko projekata rješava problem "dokaza jedinstvenosti" (proof of personhood) na blockchainu. Ni jedan ne rješava specifičan problem koji mi rješavamo: **povezivanje državnog eIDAS identiteta s lančanim upravljanjem sredstvima unutar EU regulatornog okvira.**

---

## 2. Usporedna matrica

| Kriterij | **Naš projekt** | World ID (Worldcoin) | Gitcoin Passport | Polygon ID | Estonija e-Residency |
|----------|-----------------|---------------------|------------------|------------|---------------------|
| **Izvor identiteta** | Državni eIDAS (Certilia) | Biometrija šarenice (Orb) | Kompozitni rezultat (društvene mreže, Web3 aktivnost) | Samovlasništvo (korisnik sam tvori tvrdnje) | Državni digitalni identitet |
| **Pravna valjanost** | Da (eIDAS regulativa) | Ne | Ne | Djelomično (ovisi o izdavaču tvrdnje) | Da (ali ograničeno na e-usluge) |
| **Privatnost** | Trebamo riješiti (vidi privatnost.md) | Upitna (biometrijski podaci) | Niska (društvene mreže su javne) | Visoka (ZK dokazi) | Niska (državna baza) |
| **Sybil otpornost** | Visoka (1 OIB = 1 identitet) | Visoka (1 šarenica = 1 identitet) | Srednja (moguće lažiranje rezultata) | Ovisi o izdavaču tvrdnje | Visoka (državna verifikacija) |
| **Geografski opseg** | Hrvatska (proširivo na EU putem eIDAS) | Globalni | Globalni | Globalni | Estonija (+ e-rezidenti) |
| **Decentralizacija** | Srednja (ovisi o Certilia AKD) | Niska (Worldcoin Foundation kontrolira Orb) | Srednja (Gitcoin kontrolira algoritam bodovanja) | Visoka (korisnik kontrolira tvrdnje) | Niska (potpuno centralizirana) |
| **Blockchain** | Gnosis lanac | Optimism | Ethereum / višelančano | Polygon | Nema (centralna baza) |
| **Token standard** | SBT (ERC-721 neprenosiv) | World ID (vlastiti protokol) | Passport rezultat (off-chain + on-chain atestacija) | Verifiable Credential (W3C standard) | Nema tokena |
| **EU regulatorna usklađenost** | Visoka (eIDAS, MiCA, GDPR) | Niska (biometrija pod GDPR?) | Srednja | Srednja | Visoka |

---

## 3. Detaljne analize

### World ID (Worldcoin)

**Što rade:** Skeniraju šarenicu oka posebnim uređajem ("Orb") i generiraju jedinstveni hash koji služi kao dokaz jedinstvenosti. Korisnik može ZK dokazom potvrditi da je verificiran bez otkrivanja identiteta.

**Što možemo naučiti:**
- ZK pristup privatnosti je izvrstan — nullifier model (Semaphore protokol) je direktno primjenjiv na naš sustav.
- Razdvajanje "dokaza jedinstvenosti" od "identiteta" je pametno — mi trebamo isto.

**Zašto je naš pristup bolji za naš kontekst:**
- Worldcoin zahtijeva fizički uređaj (Orb) — mi koristimo postojeću državnu infrastrukturu (Certilia).
- Worldcoin nema pravnu valjanost — naš identitet je eIDAS usklađen.
- Worldcoin je pod istragom regulatora u više EU država zbog GDPR-a (biometrijski podaci).

**Rizik:** Worldcoin dokazuje da javnost ima otpor prema biometrijskom skeniranju. Naš pristup (korištenje postojećeg državnog identiteta) je društveno prihvatljiviji.

### Gitcoin Passport

**Što rade:** Agregiraju "pečate" (stamps) iz različitih izvora (Twitter/X, GitHub, ENS, BrightID) i računaju kompozitni rezultat pouzdanosti identiteta.

**Što možemo naučiti:**
- Kompozitni pristup je zanimljiv za buduće faze — možemo kombinirati Certilia SBT s drugim izvorima verifikacije za jačanje rezultata.
- Njihov pristup atestacijama (EAS — Ethereum Attestation Service) je standardiziran i mogao bi zamijeniti naš prilagođeni SBT.

**Zašto je naš pristup bolji za naš kontekst:**
- Gitcoin Passport se lako lažira — moguće je kupiti "pečate" ili koristiti socijalni inženjering.
- Nema pravne valjanosti — ne može se koristiti za glasanje ili upravljanje javnim sredstvima.
- Rezultat je subjektivan i ne-binarni — naš SBT je binarni (verificiran ili nije).

### Polygon ID

**Što rade:** Implementiraju W3C Verifiable Credentials standard s ZK dokazima. Korisnik drži tvrdnje u svom novčaniku i selektivno ih otkriva.

**Što možemo naučiti:**
- W3C Verifiable Credentials standard je relevantan za eIDAS 2.0 / EUDI novčanik.
- Njihov pristup selektivnom otkrivanju (selective disclosure) je direktno primjenjiv — korisnik može dokazati "stariji sam od 18" bez otkrivanja datuma rođenja.
- Circom ZK krugovi za verifikaciju potpisa su otvoreni i mogli bi biti temelj za naš ZK pristup.

**Zašto je naš pristup drugačiji:**
- Polygon ID ne specificira izvor tvrdnji — bilo tko može izdati tvrdnju. Mi imamo konkretan, reguliran izvor (AKD Certilia).
- Polygon ID je agnostičan prema regulativi — mi smo eksplicitno dizajnirani za EU/eIDAS kontekst.

### Estonija e-Residency

**Što rade:** Estonija izdaje digitalni identitet svima (uključujući ne-rezidente) koji omogućuje pristup estonskim e-uslugama, osnivanje tvrtki i digitalno potpisivanje.

**Što možemo naučiti:**
- Estonija dokazuje da je **državni digitalni identitet moguć i skalabilan** — 100.000+ e-rezidenata.
- Njihov X-Road sustav (razmjena podataka između državnih sustava) je inspiracija za interoperabilnost.
- KSI blockchain (Keyless Signature Infrastructure) koriste za integritet podataka — ne za financijske transakcije.

**Zašto je naš pristup bolji za naš kontekst:**
- Estonija koristi centraliziranu infrastrukturu — nema lančanog upravljanja sredstvima.
- Estonija ne rješava problem transparentnosti proračuna — digitalizirali su birokraciju, ne demokratizirali financije.
- Naš projekt koristi decentralizirane trezore (Safe) umjesto centralnih baza podataka.

---

## 4. Naša jedinstvena pozicija

```
                    Pravna valjanost (eIDAS)
                           ^
                           |
                    Naš    |
                   projekt *
                           |
        Polygon ID *       |         * Estonija
                           |
     ---+----------+-------+-------+----------+--->
        |          |               |          |
     Potpuno     Srednje        Srednje    Potpuno
     decentra-   decentra-     centrali-  centrali-
     lizirano    lizirano      zirano     zirano
                           |
                           |
        Gitcoin    *       |
        Passport           |
                           |
                   World   |
                   ID  *   |
                           v
                    Bez pravne valjanosti
```

**Naš projekt zauzima jedinstven prostor:** dovoljno decentraliziran da bude trustless, ali s pravnom valjanošću koju nijedan čisto kripto projekt nema.

---

## 5. Tehnologije za potencijalno usvajanje

| Tehnologija | Izvor | Primjena u našem projektu |
|-------------|-------|--------------------------|
| Semaphore nullifier protokol | World ID / PSE | Privatnost glasanja (faza 3) |
| EAS (Ethereum Attestation Service) | Gitcoin / EAS | Alternativa prilagođenom SBT-u — standardiziraniji pristup |
| Circom ZK krugovi za potpise | Polygon ID / iden3 | ZK verifikacija JWT potpisa off-chain |
| W3C Verifiable Credentials | Polygon ID / EU standardi | Kompatibilnost s EUDI novčanikom (eIDAS 2.0) |
| X-Road interoperabilnost | Estonija | Inspiracija za višedržavnu eIDAS podršku |
