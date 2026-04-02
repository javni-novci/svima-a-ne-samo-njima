# Model prijetnji — proširena analiza

**Status:** Živi dokument — ažurirati pri svakoj arhitektonskoj promjeni
**Prioritet:** Kritičan

---

## 1. Klasifikacija prijetnji po ozbiljnosti

### Legenda

| Razina | Utjecaj | Primjer |
|--------|---------|---------|
| **Kritična** | Gubitak sredstava ili masovna krađa identiteta | Kompromis ključa AKD-a, greška u ugovoru |
| **Visoka** | Deanonimizacija korisnika ili cenzura | Calldata curenje, selektivno opozivanje |
| **Srednja** | Degradacija usluge ili financijski gubitak platforme | Iscrpljivanje platitelja, zaobilaženje provizije |
| **Niska** | Neugodnost za pojedinačnog korisnika | Istek JWT-a, privremena uskrata usluge |

---

## 2. Potpuna matrica napada

### Kritična ozbiljnost

| # | Napad | Vektor | Ublažavanje | Rezidualni rizik |
|---|-------|--------|-------------|------------------|
| T1 | Greška u pametnom ugovoru | Pogreška u JWT parsiranju, RSA verifikaciji ili SBT logici | Formalna verifikacija kritičnih dijelova. Profesionalna revizija prije produkcije. Ograničenje početnog proračuna u trezoru. | Srednji. Čak i revidirani ugovori mogu imati greške (vidi Wormhole hack, Euler hack). |
| T5 | Kompromis Certilia ključa | AKD privatni ključ procurjeli ili ukraden | `updateCertiliaPublicKey()` upravljan od Safe višepotpisnog. Skupno opozivanje sumnjivih SBT-ova. Nadzor JWKS krajnje točke za neočekivane promjene. | Srednji. Prozor izloženosti od kompromisa do detekcije. Napadač može skovati neograničen broj lažnih SBT-ova u tom prozoru. |
| T13 | Monerium zamrzavanje EURe | Monerium (kao regulirani izdavač) može zamrznuti sredstva na bilo kojoj adresi prema MiCA regulativi | Diverzifikacija stabilnih tokena (podržati i USDC, DAI kao alternativu). Držati minimalne zalihe u ugovorima — kreatori primaju sredstva odmah kroz razdjelnik. | Visok. Ne možemo tehnički spriječiti regulatornu akciju. Jedino ublažavanje je minimizirati izloženost. |

### Visoka ozbiljnost

| # | Napad | Vektor | Ublažavanje | Rezidualni rizik |
|---|-------|--------|-------------|------------------|
| T2 | Zamjena nonce-a | Zlonamjerni/hakiran poslužitelj podmeće tuđu adresu kao nonce | Klijent verificira nonce u URL-u prije preusmjeravanja (korak 4). Hardverski novčanik kao zadnja obrana. | Nizak. Ovisi o integritetu frontenda. |
| T9 | Napad na opskrbni lanac frontenda | Zlonamjerni npm paket, otmica CDN-a | SRI sažeci. Provjera nonce-a na strani klijenta. Hardverski novčanik ne izlaže privatne ključeve frontendu. | Srednji. Frontend može prikazati lažne podatke ali ne može potpisati transakcije bez korisnikovog odobrenja. |
| T10 | Vremenski napad na JWT (NOVO) | Napadač presretne JWT u prijenosu poslužitelj→klijent (MITM ili XSS) i iskuje SBT na svoju adresu prije korisnika | JWT nonce je vezan za korisnikovu adresu. Čak i ako napadač presretne JWT, pametni ugovor provjerava `nonce == msg.sender`. Napadač ne može koristiti tuđi JWT. | Vrlo nizak. Nonce vezanje potpuno eliminira ovaj vektor. |
| T11 | Prisluškivanje calldata (NOVO) | Bilo tko čita transakcijski calldata na lancu i doznaje osobne podatke iz JWT-a | Selektivno prosljeđivanje (vidi privatnost.md — Opcija A). Dugoročno: nullifier pristup (Opcija B). | Visok bez mitigacije. Nizak s Opcijom A. Zanemariv s Opcijom B. |
| T12 | Cenzura od strane Safe potpisnika (NOVO) | Potpisnici selektivno opozivaju SBT-ove ili odbijaju rotaciju ključeva | Povećati kvorum potpisnika. Uključiti neovisne potpisnike (npr. predstavnike zajednice). Vremensko zaključavanje na opozivanje (48h odgoda, moguć prigovor). | Srednji. Fundamentalni problem upravljanja — tko nadzire nadzornike? |

### Srednja ozbiljnost

| # | Napad | Vektor | Ublažavanje | Rezidualni rizik |
|---|-------|--------|-------------|------------------|
| T3 | Ponovno korištenje JWT-a | Napadač presretne JWT i pokušava drugi mint | `usedJWTs[hash]` mapa u ugovoru. JWT `exp` tvrdnja. | Vrlo nizak. |
| T4 | Sybil napad | Jedna osoba, više novčanika, više SBT-ova | `identityClaimed[hash(sub)]` mapa u ugovoru. | Nizak. Ovisi o jedinstvenosti `sub` tvrdnje. |
| T6 | Iscrpljivanje platitelja | Spam korisničkih operacija za trošenje fonda platitelja | Ograničenje: 1 kovanje po Certilia identitetu (ne po adresi). Dnevni proračunski limit. | Nizak. Trošak napada > korist za napadača. |
| T14 | Kreator nestaje (NOVO) | Kreator primi sredstva pa obriše sadržaj ili nestane | Vremensko zaključavanje isplata (48h). Sustav reputacije temeljen na lančanoj povijesti. Za veće iznose: uvjetno oslobađanje (escrow). | Srednji. Ne možemo spriječiti nekoga da potroši vlastite EURe. |

### Niska ozbiljnost

| # | Napad | Vektor | Ublažavanje | Rezidualni rizik |
|---|-------|--------|-------------|------------------|
| T1 | Kompromitiran poslužitelj (uskrata usluge) | Napadač preuzme Node.js poslužitelj | Poslužitelj nema ključeve. Jedini utjecaj: korisnici ne mogu započeti novu Certilia autentikaciju dok se poslužitelj ne obnovi. Postojeći SBT-ovi i sredstva ostaju netaknuti. | Nizak. |
| T8 | Izgubljen mobitel / istekla iskaznica | Korisnik gubi pristup Certilia identitetu | `revokeSBT()` putem Safe višepotpisnog. Ponovna verifikacija s novim novčanikom. | Nizak. |
| T15 | Istek JWT-a prije kovanja (NOVO) | Korisnik presporo potpiše transakciju kovanja, JWT istekne (obično 5 min) | Jasno korisničko sučelje s odbrojavanjem. Mogućnost ponovnog pokretanja toka bez ponovne autentikacije (koristiti isti auth code za novi JWT, ako Certilia dopušta). | Nizak. Neugodnost, ne sigurnosni problem. |

---

## 3. Stabla napada za kritične scenarije

### Stablo napada: Krađa identiteta

```
Cilj: Skovati SBT s tuđim identitetom
|
+-- 1. Nabaviti JWT s tuđim sub-om
|   +-- 1a. Kompromitirati Certilia ključ --> Skovati lažni JWT
|   |   (Ublažavanje: rotacija ključa, nadzor JWKS)
|   +-- 1b. Presresti tuđi JWT u prijenosu
|   |   (Ublažavanje: nonce == msg.sender, napadač ne može koristiti)
|   +-- 1c. Socijalni inženjering na korisnika
|       (Ublažavanje: korisnik nikada ne dijeli JWT, edukacija)
|
+-- 2. Zaobići nonce provjeru
|   +-- 2a. Podmetnuti svoju adresu kao nonce (zamjena nonce-a)
|   |   (Ublažavanje: klijent verificira nonce u URL-u, korak 4)
|   +-- 2b. Kompromitirati frontend da preskoči provjeru
|       (Ublažavanje: SRI, hardverski novčanik, edukacija)
|
+-- 3. Zaobići on-chain provjeru
    +-- 3a. Pronaći grešku u RSA/P-256 verifikaciji u Solidity-u
    |   (Ublažavanje: formalna verifikacija, revizija)
    +-- 3b. Pronaći grešku u JWT parsiranju
        (Ublažavanje: korištenje provjerenih biblioteka, revizija)
```

### Stablo napada: Krađa sredstava

```
Cilj: Ukrasti EURe iz sustava
|
+-- 1. Napasti platformski Safe
|   +-- 1a. Kompromitirati kvorum potpisnika
|   |   (Ublažavanje: hardverski novčanici, geografska distribucija)
|   +-- 1b. Iskoristiti Zodiac modul
|   |   (Ublažavanje: revizija modula, ograničenja dopuštenja)
|   +-- 1c. Socijalni inženjering na potpisnike
|       (Ublažavanje: vremensko zaključavanje, obavijesti)
|
+-- 2. Napasti razdjelnik plaćanja
|   +-- 2a. Promijeniti adrese primatelja
|   |   (Ublažavanje: nepromjenjive adrese ili admin = Safe)
|   +-- 2b. Promijeniti udjele raspodjele
|       (Ublažavanje: fiksni udjeli u ugovoru ili admin = Safe)
|
+-- 3. Napasti korisnikov novčanik izravno
    +-- 3a. Ukrasti privatni ključ (phishing, zlonamjerni softver)
    |   (Ublažavanje: izvan našeg opsega — preporuka hardverski novčanik)
    +-- 3b. Podmetnuti lažnu transakciju za potpis
        (Ublažavanje: čitljivi potpisi, EIP-712)
```

---

## 4. Matrica rizika (vjerojatnost x utjecaj)

```
Utjecaj
  ^
  |
K |                              T5 (Certilia ključ)
r |                    T13       T1 (greška ugovora)
i |          (Monerium)
t |
i |
č |
n |---------------------------------------------------
V |                    T12       T9 (opskrbni lanac)
i |          (cenzura)           T11 (calldata)
s |
o |
k |---------------------------------------------------
S |  T14               T4       T2 (nonce swap)
r |  (kreator)         (Sybil)
e |
d |---------------------------------------------------
N |  T15               T6       T3 (JWT replay)
i |  (istek JWT)       (plat.)  T8 (izgubljen mob.)
z |                              T10 (vremenski)
a |
k +------+--------+--------+--------+--->
         Niska    Srednja   Visoka   Vjerojatnost
```

---

## 5. Prioritetni popis ublažavanja

| Prioritet | Ublažavanje | Napadi koje pokriva | Složenost |
|-----------|-------------|--------------------|-----------| 
| 1 | Riješiti calldata curenje (privatnost.md) | T11 | Srednja |
| 2 | Profesionalna revizija pametnih ugovora | T1, T5 (djelomično) | Visoka (trošak) |
| 3 | Klijentska provjera nonce-a (korak 4) | T2, T9 (djelomično) | Niska |
| 4 | Vremensko zaključavanje na Safe operacije | T12, T14 | Niska |
| 5 | Nadzor Certilia JWKS krajnje točke | T5 | Niska |
| 6 | Diverzifikacija stabilnih tokena | T13 | Srednja |
| 7 | Nullifier pristup za glasanje | T11 (potpuno) | Visoka |
