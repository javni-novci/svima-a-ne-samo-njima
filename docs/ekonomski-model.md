# Ekonomski model i održivost

**Status:** Nacrt — zahtijeva validaciju s kreatorima i korisnicima
**Prioritet:** Važan — utječe na održivost cijelog sustava

---

## 1. Tok sredstava

### Primarni tok: Podcast tržište

```
Korisnik (kupac)
  |
  |  Plaća X EURe za sadržaj
  v
+---------------------------+
|  Razdjelnik plaćanja      |
+---------------------------+
  |                    |
  |  90% (X * 0.9)    |  10% (X * 0.1)
  v                    v
+---------------+    +--------------------+
|  Kreator      |    |  Platformski Safe  |
|  (Novčanik    |    |  (Višepotpisni)    |
|   pod vlast.  |    |                    |
|   skrbi)      |    +----+---------------+
+---------------+         |
                          |  Raspodjela operativnih troškova
                          v
                   +------+------+------+
                   |      |      |      |
                   v      v      v      v
               Platitelj Razvoj  Reviz. Rezerva
               goriva           ugovora
               (30%)   (40%)   (20%)   (10%)
```

### Sekundarni tok: Pretplate (budućnost)

Kreatori mogu ponuditi mjesečne pretplate. Isti mehanizam raspodjele, ali s vremenski zaključanim ugovorom koji automatski isplaćuje mjesečno.

---

## 2. Održivost platitelja goriva

### Problem

Platitelj goriva sponzorira prvu transakciju (kovanje SBT-a) za svakog novog korisnika. Ovo je trošak platforme koji raste linearno s brojem korisnika.

### Procjena troškova

| Parametar | Vrijednost |
|-----------|-----------|
| Trošak kovanja SBT-a na Gnosisu — RSA verif. (istraživanje 02) | ~0,002-0,005 USD (~1,5M gas) |
| Trošak kovanja SBT-a na Gnosisu — ZK nullifier (istraživanje 02) | ~0,0003-0,001 USD (~200-300K gas) |
| Očekivani broj korisnika (godina 1) | 1.000-10.000 |
| Ukupni godišnji trošak platitelja | 10-500 USD |
| Izvor financiranja | 30% od platformske provizije |

**Zaključak:** Na Gnosis lancu troškovi platitelja su zanemarivi za operativni proračun. Čak i pri 100.000 korisnika, godišnji trošak ne prelazi 5.000 USD. Za razliku od Ethereum glavne mreže (50.000-500.000 USD).

**UPOZORENJE:** Ovo NE znači da je financijski rizik platforme zanemariv. Monerium (izdavač EURe) ima MiCA ovlast zamrznuti tokene na bilo kojoj adresi — vidi [pravna-analiza.md](pravna-analiza.md) odjeljak 3 za detalje. Ublažavanje: držati minimalne zalihe u ugovorima (razdjelnik isplaćuje odmah), razmotriti podršku za alternativne stabilne tokene.

### Zaštita od zlouporabe

- Ograničenje: 1 kovanje bez goriva po Certilia identitetu (ne po adresi — adrese se mogu generirati besplatno, ali Certilia identiteti ne).
- Dnevni proračunski limit platitelja: maks. Y xDAI/dan.
- Nadzorna ploča: praćenje potrošnje platitelja u stvarnom vremenu iz Safe-a.

---

## 3. Postotak raspodjele

### Trenutni prijedlog: 90/10

| Usporedba | Udio kreatora | Udio platforme |
|-----------|---------------|----------------|
| **Naš sustav** | **90%** | **10%** |
| YouTube | ~55% | ~45% |
| Spotify (podcasti) | ~50% | ~50% |
| Patreon | 88-95% | 5-12% |
| 0xSplits (Web3 standard) | Konfigurirano | Konfigurirano |

### Otvorena pitanja

| # | Pitanje | Implikacije |
|---|---------|-------------|
| E1 | Treba li postotak biti fiksan u ugovoru ili upravljan kroz DAO? | Fiksni = jednostavnije, pouzdanije. DAO upravljan = fleksibilnije, ali otvara napadački vektor (glasanje za promjenu postotka u korist jedne strane). |
| E2 | Treba li postojati razlika u postotku za male vs. velike kreatore? | Poticaj za male kreatore, ali komplicira logiku razdjelnika. |
| E3 | Kako spriječiti zaobilaženje platforme? | Nakon prvog kontakta, kreator i kupac mogu dogovoriti izravno plaćanje. Jedini odgovor: platforma mora kontinuirano pružati vrijednost (alati za distribuciju, analitika, SBT verifikacija). |

---

## 4. Poslovni model platforme

### Prihodi

| Izvor | Opis | Prioritet |
|-------|------|-----------|
| Provizija na prodaju sadržaja | 10% od svake transakcije | Primarni |
| Pretplate kreatora (premium alati) | Mjesečna naknada za napredne značajke | Sekundarni (budućnost) |
| Brendirani SBT-ovi za organizacije | Tvrtke plaćaju za vlastite verifikacijske tokene | Istraživačko (budućnost) |

### Rashodi

| Trošak | Opis | Procjena (god. 1) |
|--------|------|-------------------|
| Platitelj goriva | Sponzorirano kovanje SBT-a | 10-500 USD |
| Node.js poslužitelj | OIDC poštar (minimalan) | 50-200 USD/mj. |
| Revizija pametnih ugovora | Profesionalna sigurnosna revizija | 10.000-50.000 USD (jednokratno) |
| DPIA (procjena utjecaja na zaštitu podataka) | Obvezno prema EDPB 02/2025 (istraživanje 06) | 1.000-5.000 EUR |
| Pravne konzultacije (GDPR, MiCA) | AZOP savjetovanje, HANFA upit o CASP licenci | 1.000-3.000 EUR |
| Razvoj | Timski troškovi | Ovisno o modelu (volonteri vs. plaćeni) |

### Potencijalni besplatni resursi (iz istraživanja 03)

- **Safe{Core} Gas Station:** Do $50.000 besplatnih gas kredita na Gnosisu za projekte koji koriste Safe + ERC-4337 + Pimlico.
- **Pimlico besplatni plan:** 1M credits/mj (~1.300 UserOps). Dovoljno za rani pristup.
- **Monerium SEPA rampa:** Bez naknada za SEPA ↔ EURe konverziju (trenutno).

### Kritična točka

**Revizija pametnih ugovora** je daleko najveći jednokratni trošak. Opcije:
- Tražiti financiranje od EU fondova (digitalna transformacija / e-uprava).
- Pokrenuti javno crowdfunding kampanju (koristiti vlastitu infrastrukturu kao dokaz koncepta).
- Početi s ograničenim proračunom u Safe-u i ručnom revizijom zajednice, pa profesionalna revizija za fazu 2.
- **Prijaviti se za Safe{Core} Gas Station** za gas kredite na Gnosisu.

---

## 5. Izlazni scenariji za korisnike

### Kreator želi otići s platforme

- Novčanik je pod vlastitom skrbi — kreator zadržava sve zaradene EURe.
- SBT ostaje na novčaniku — identitet nije vezan za platformu.
- Kreator gubi pristup platformskim alatima, ali ne gubi ništa na lancu.

### Korisnik želi povrat

- Platformski Safe može implementirati vremensko zaključavanje: isplata kreatoru se odgađa 48h, unutar kojih korisnik može zatražiti povrat.
- Povrate odobrava Safe višepotpisni (ili Zodiac modul s pravilima).

### Platforma prestane postojati

- Svi SBT-ovi ostaju na lancu — identiteti su trajni.
- Sva sredstva u novčanicima kreatora ostaju netaknuta.
- Razdjelnici plaćanja nastavljaju raditi autonomno — nema centralnog prekidača.
- **Ovo je ključna prednost nad Web2 platformama.**
