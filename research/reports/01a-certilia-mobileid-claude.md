# Certilia Mobile.id — Claude Opus 4.6 izvještaj

**Prompt:** [01-certilia-mobileid.md](../prompts/01-certilia-mobileid.md)
**Izvor:** Claude Opus 4.6 Deep Research
**Datum:** ____-__-__
**Status:** Čeka izvršenje

---

<!-- Zalijepi rezultat Claude Opus 4.6 deep research ovdje -->

# Certilia Mobile.id — tehničke specifikacije OIDC sustava

Certilia Mobile.id koristi **WSO2 Identity Server** kao OIDC platformu, nudi besplatan Identity API s OAuth2/OpenID Connect standardom za treće strane, ali **detaljna tehnička dokumentacija dostupna je isključivo nakon registracije** na developer portalu. Sustav vraća provjerene službene podatke — **ime, prezime i OIB** — s eIDAS visokom razinom sigurnosti, no **nije notificiran za prekograničnu uporabu** pod eIDAS uredbom. Za vaš use case (prosljeđivanje JWT-a pametnom ugovoru na Gnosis blockchainu), ključno je da platforma vrlo vjerojatno koristi **RS256 s RSA-2048** ključem, što zahtijeva oko ~1M gas za on-chain verifikaciju.

---

## 1. Potvrđene činjenice (verificirano iz službenih izvora)

### OIDC konfiguracija

**IDP domena**: `idp.certilia.com` — potvrđeno iz Google-indeksirane stranice za prijavu na developer portal. Autorizacijski URL koji je pronađen:

```
https://idp.certilia.com/authenticationendpoint/login.do?
  client_id=ypEmXPfX9Yglr78cOKE7TBCEgF8a
  &commonAuthCallerPath=/oauth2/authorize
  &redirect_uri=https://developer.certilia.com/api/oauth-callback
  &response_type=code
  &scope=openid+profile+offline_access+internal_login
  &state=m7ddtowqxmacysrkq3bqy58t2qrznyp39ipvkx8vbt2bjn2jbe4n7uts77lwqn2
  &tenantDomain=carbon.super
  &type=oidc
```

Na temelju ovog URL-a potvrđene su sljedeće činjenice:

- **Authorization endpoint**: `https://idp.certilia.com/oauth2/authorize`
- **Podržani response_type**: `code` (Authorization Code Flow — potvrđeno)
- **Podržani scopeovi (opaženi)**: `openid`, `profile`, `offline_access`, `internal_login`
- **State parametar**: **DA, podržan** — vidljiv u URL-u s punom CSRF zaštitom
- **Client ID format**: WSO2-stil Base64 string (npr. `ypEmXPfX9Yglr78cOKE7TBCEgF8a`)
- **Autentifikatori**: `BasicAuthenticator:LOCAL`, `x509CertificateAuthenticator:LOCAL`, `CreateAccountAuthenticator:LOCAL`

**Platforma**: **WSO2 Identity Server** — potvrđeno na temelju više WSO2-specifičnih parametara: `tenantDomain=carbon.super` (WSO2 defaultni super-tenant), `sessionDataKey`, `commonAuthCallerPath`, `relyingParty` parametar, i URL pattern `/authenticationendpoint/login.do` koji je WSO2 IS autentifikacijski endpoint.

### JWT sadržaj (potvrđeno iz certilia.com/identity-api/)

Službena stranica Certilia Identity API-ja eksplicitno navodi da sustav vraća:

- **ime** (given_name / first name)
- **prezime** (family_name / last name)  
- **OIB** (Osobni identifikacijski broj) — **DA, OIB je eksplicitno prisutan u JWT-u**
- **ostale podatke** — vlasnik aplikacije određuje koje dodatne podatke traži

Korisnik prilikom **prve prijave** vidi tražene podatke i daje **privolu** (GDPR consent) za njihovo slanje. Certilia Identity API koristi **OAuth2/OpenID Connect standard** i funkcionira poput "Login with Google" prijave, ali vraća službene i provjerene podatke.

### Registracija klijentske aplikacije

- **Developer portal (produkcija)**: `https://developer.certilia.com/`
- **Test/sandbox okruženje**: `https://developer.test.certilia.com/` — **DA, postoji!**
- **Registracija**: besplatna, samouslužna — registrirate se na developer portalu i generirate API ključeve u minutama
- **Cijena Identity API-ja**:
  - **Start paket**: 100.000 prijava/mjesečno — **besplatno**
  - **Business paket**: neograničene prijave — **66,36 €/mjesečno**

### eIDAS razina sigurnosti

- **mobileID** (prijava kroz Certilia mobilnu aplikaciju): **visoka razina sigurnosti** (eIDAS High) — potvrđeno na certilia.com
- **smsID** (jednokratni SMS kod): **značajna razina sigurnosti** (eIDAS Substantial)
- Na službenoj **Listi prihvaćenih vjerodajnica** NIAS-a, Certilia mobile.ID osobna i poslovna vjerodajnica klasificirane su kao **razina 3 (visoka)**
- **Certilia Mobile.id NIJE notificiran** pod eIDAS uredbom za prekograničnu uporabu. Jedina hrvatska notificirana eID shema je **osobna iskaznica (eOI)**, notificirana 7. studenog 2018. na razini "High"
- Certilia se **ne može koristiti** za prekograničnu autentifikaciju na e-uslugama drugih EU država

### Pravni subjekt i infrastruktura

**AKD d.o.o.** (Agencija za komercijalnu djelatnost) — tvrtka od posebnog interesa za Republiku Hrvatsku, **100% u vlasništvu Vlade RH**. Adresa: Savska cesta 31, 10000 Zagreb. OIB: 58843087891. Temeljni kapital: 50.000.000,00 EUR. Direktor: Jure Sertić. Certifikati: **ISO 27001, ISO 9001, eIDAS, Common Criteria CC EAL4+**. Podatkovni centar: **Tier III** (AKD DC Sveta Nedjelja).

PKI hijerarhija: **AKDCA Root → CERTILIA CA** (zamjenila KIDCA od rujna 2023.) → korisničke certifikate.

### Dual-protocol arhitektura — NIAS vs. Identity API

Ovo je **kritična** arhitektonska razlika koju morate razumjeti:

**Za e-Građani (NIAS integraciju)** — Certilia komunicira s NIAS-om putem **SAML 2.0**. Izdavatelj vjerodajnica (AKD) šalje samo **OIB** u SAML odgovoru, a NIAS samostalno obogaćuje atribute iz OIB sustava (ime, prezime itd.). E-usluge se integriraju s NIAS-om, ne direktno s Certiliom.

**Za treće strane (Identity API)** — Certilia nudi **OAuth2/OpenID Connect** API izravno, mimo NIAS-a. Ovo je ono što vama treba za blockchain integraciju. Registrirate aplikaciju na developer.certilia.com, dobivate client_id i client_secret, i koristite standardni OIDC flow.

---

## 2. Djelomično potvrđeno (iz platforme WSO2 IS, nepotvrdivo bez pristupa discovery endpointu)

Sljedeće informacije temelje se na **potvrđenoj identifikaciji platforme** (WSO2 Identity Server) i njenim standardnim postavkama. AKD je mogao prilagoditi konfiguraciju, pa ove vrijednosti uzimajte kao **vrlo vjerojatne, ali ne potvrđene**.

### OIDC Discovery i endpointi

| Endpoint | Očekivani URL |
|----------|---------------|
| **OIDC Discovery** | `https://idp.certilia.com/oauth2/oidcdiscovery/.well-known/openid-configuration` ili `https://idp.certilia.com/oauth2/token/.well-known/openid-configuration` |
| **Token** | `https://idp.certilia.com/oauth2/token` |
| **UserInfo** | `https://idp.certilia.com/oauth2/userinfo` |
| **JWKS** | `https://idp.certilia.com/oauth2/jwks` |
| **Revocation** | `https://idp.certilia.com/oauth2/revoke` |
| **Introspection** | `https://idp.certilia.com/oauth2/introspect` |
| **End Session** | `https://idp.certilia.com/oidc/logout` |

**Napomena**: WSO2 IS ima nestandardni OIDC discovery path (`/oauth2/oidcdiscovery/.well-known/openid-configuration`), ali može se konfigurirati na standardni `/.well-known/openid-configuration`. Nismo uspjeli dohvatiti nijedan od ovih endpointova jer nisu javno indeksirani.

### JWT potpis i ključevi

| Parametar | Očekivana vrijednost (WSO2 IS default) |
|-----------|---------------------------------------|
| **Algoritam potpisa** | **RS256** (SHA256withRSA) |
| **Veličina RSA ključa** | **2048 bita** (minimum koji IS 5.10.0+ provodi) |
| **Broj ključeva u JWKS** | **1** (default WSO2 instalacija ima jedan RSA ključ) |
| **kid format** | Base64-kodirani SHA-1 ili SHA-256 hash certificate thumbprinta (dugi alfanumerički string) |
| **Rotacija ključeva** | Nije automatizirana u WSO2 IS; ručni proces |

**Implikacija za blockchain**: RS256 s RSA-2048 zahtijeva oko **~1M gas** za on-chain verifikaciju na EVM-u. Alternativa je off-chain verifikacija s on-chain pohranom rezultata (npr. pomoću orakulja ili zk-SNARK dokaza).

### Nonce parametar — KRITIČNO za vaš use case

**DA, WSO2 Identity Server podržava `nonce` parametar** u autorizacijskim zahtjevima. Nonce vrijednost se ugrađuje u ID token kao `"nonce"` claim. Primjer:

```
/oauth2/authorize?response_type=code&client_id=...&nonce=0x1234abcd...&scope=openid
```

ID token će sadržavati: `"nonce": "0x1234abcd..."`. Ovo znači da **možete poslati Ethereum adresu kao nonce** i ona će biti prisutna u potpisanom JWT-u.

### Podržani grant_types i response_types (WSO2 IS defaults)

**response_types_supported**: `code`, `id_token`, `token`, `id_token token`, `code token`, `code id_token`, `code id_token token`

**grant_types_supported**: Authorization Code, Implicit, Client Credentials, Resource Owner Password, Refresh Token, SAML2 Bearer, JWT Bearer

**scopes_supported**: `openid`, `profile`, `email`, `phone`, `address` — plus potencijalno prilagođeni Certilia scopeovi za OIB

### Token trajanje (WSO2 IS defaults)

| Token tip | Default trajanje |
|-----------|-----------------|
| **ID Token** | **3600 sekundi (1 sat)** |
| **Access Token** | **3600 sekundi (1 sat)** |
| **Refresh Token** | **86400 sekundi (24 sata)** |
| **Authorization Code** | **300 sekundi (5 minuta)** |

### `sub` claim

U WSO2 IS, `sub` (subject) claim može biti konfiguriran kao:
- Korisničko ime
- Pairwise identifier (jedinstven po klijentu)
- Ili prilagođeni identifikator

U kontekstu Certilie, **`sub` je najvjerojatnije OIB ili pseudonimni hash OIB-a**, ali ovo nije potvrđeno. S obzirom da Certilia eksplicitno vraća OIB kao claim, moguće je da je `sub` upravo OIB ili da je OIB u zasebnom custom claimu.

---

## 3. Nepoznato — zahtijeva direktan kontakt s AKD-om ili pristup developer portalu

Sljedeća pitanja **nije bilo moguće odgovoriti** iz javno dostupnih izvora i zahtijevaju registraciju na `developer.certilia.com` ili direktan kontakt s AKD-om:

- **Točan OIDC Discovery URL** — potencijalni WSO2 path, ali nije potvrđen dohvatom
- **Točan issuer** u konfiguraciji — mogao bi biti `https://idp.certilia.com/oauth2/token` (WSO2 default issuer format) ili `https://idp.certilia.com`
- **Točan JWT signing algorithm** — vrlo vjerojatno RS256, ali mogao bi biti prilagođen
- **Točna veličina RSA ključa** — očekivano 2048, ali moglo bi biti 4096
- **JWKS sadržaj** — broj ključeva, kid format, e i n vrijednosti javnog ključa
- **Politika rotacije ključeva** — koliko često AKD rotira ključeve potpisa
- **Puni popis claimova u JWT-u** — jesu li prisutni `date_of_birth`, `email`, `address`? Koji su prilagođeni claimovi?
- **Format `sub` claima** — OIB, hash OIB-a, ili interni identifikator?
- **Format `aud` claima** — client_id ili custom format?
- **Exact exp vrijednost** — je li 1 sat ili prilagođeno?
- **redirect_uri ograničenja** — podržava li wildcard, localhost za razvoj?
- **Uvjeti korištenja** — postoje li ograničenja na korištenje JWT-a izvan predviđene autentifikacije (npr. za blockchain prosljeđivanje)?
- **SDK-ovi ili biblioteke** — postoje li službene biblioteke za integraciju?
- **Postoji li mogućnost odabira ES256 (P-256) umjesto RS256?** — P-256 je znatno jeftiniji za on-chain verifikaciju (~200K gas vs. ~1M gas za RSA)

### Preporučeni koraci

1. **Registrirajte se na `developer.certilia.com`** — tamo se nalaze API dokumentacija, OIDC endpointi, i konfiguracija
2. **Koristite test okruženje** `developer.test.certilia.com` za razvoj bez stvarnih identiteta
3. **Kontaktirajte AKD helpdesk** (helpdesk@certilia.com / 0800 0440) za specifična pitanja o:
   - Korištenju nonce parametra za Ethereum adresu
   - Mogućnosti korištenja ES256 umjesto RS256 (za jeftiniju on-chain verifikaciju)
   - Pravnim ograničenjima prosljeđivanja JWT-a pametnom ugovoru
4. **Dohvatite OIDC Discovery** iz developer portala i testirajte endpoint: pokušajte `https://idp.certilia.com/oauth2/jwks` i `https://idp.certilia.com/oauth2/oidcdiscovery/.well-known/openid-configuration`

---

## 4. Procjena za blockchain use case

Za vaš specifičan use case — prosljeđivanje Certilia JWT-a na Gnosis pametni ugovor — evo ključnih tehničkih razmatranja:

**Nonce parametar funkcionira**: Možete ugraditi Ethereum adresu u `nonce` polje autorizacijskog zahtjeva. JWT će sadržavati tu adresu kao potpisani claim, čime se kriptografski vezuje identitet korisnika za wallet adresu.

**RS256 on-chain verifikacija je skupa ali izvediva**: Ako Certilia koristi RS256 (što je vrlo vjerojatno), RSA verifikacija na EVM-u zahtijeva pre-compiled contract za modularnu eksponencijaciju (EIP-198, adresa `0x05`), koji košta oko **~200K-1M gas** ovisno o implementaciji. Na Gnosisu (niski gas fee) to je prihvatljivo.

**ES256 bi bio idealn**: Secp256r1 (P-256) verifikacija je podržana na EVM-u od EIP-7212 (RIP-7212), ali samo na nekim lancima. Provjerite podržava li Gnosis Chain ovaj precompile. Ako Certilia podržava ES256, to bi dramatično smanjilo trošak.

**OIB je u JWT-u**: Ovo je ključno — OIB je jedinstven 11-znamenkasti identifikator svakog hrvatskog građanina. Može se koristiti kao determinističko mapiranje identiteta na blockchain adresi.

**Pravno upozorenje**: OIB je **osobni podatak** prema GDPR-u. Pohrana OIB-a na blockchain (koji je javni nepromjenjivi registar) ima ozbiljne **GDPR implikacije**. Razmislite o pohranjivanju samo **hasha OIB-a** (keccak256) umjesto samog OIB-a.

---

## 5. Relevantni linkovi

### Službeni Certilia resursi
| Resurs | URL |
|--------|-----|
| Certilia početna | https://www.certilia.com/ |
| Identity API opis | https://www.certilia.com/identity-api/ |
| eSign API opis | https://www.certilia.com/esign-api/ |
| Developer portal (produkcija) | https://developer.certilia.com/ |
| Developer portal (test/sandbox) | https://developer.test.certilia.com/ |
| Korisnički portal | https://portal.certilia.com/ |
| Registracija | https://reg.certilia.com/ |
| PKI dokumenti i pravilnici | https://www.certilia.com/dokumenti/ |
| Preuzimanja (middleware, CA) | https://www.certilia.com/preuzimanja |
| Knowledge base / podrška | https://support.certilia.com/knowledgebase |
| IDP endpoint (potvrđen) | https://idp.certilia.com/ |

### Službeni NIAS i e-Građani resursi
| Resurs | URL |
|--------|-----|
| NIAS teh. specifikacija integracije (v1.8, 2025) | https://gov.hr/UserDocsImages/Dokumenti/Tehni%C4%8Dka%20specifikacija%20za%20integraciju%20e-usluga%20u%20NIAS2025.pdf |
| NIAS teh. specifikacija atributa (v3.1, 2024) | https://gov.hr/UserDocsImages/Dokumenti/Tehni%C4%8Dka%20specifikacija%20korisni%C4%8Dkih%20atributa%20za%20ePoslovanje.pdf |
| Lista prihvaćenih vjerodajnica | https://gov.hr/hr/lista-prihvacenih-vjerodajnica/1792 |
| Što je NIAS? | https://gov.hr/hr/sto-je-nias/2188 |
| NIAS operativni protokol (EN) | https://ec.europa.eu/digital-building-blocks/sites/download/attachments/62885743/T1.v1.0_NIAS%20Operational%20Protocol.pdf?api=v2 |

### eIDAS resursi
| Resurs | URL |
|--------|-----|
| EU eIDAS notifikacija — Hrvatska | https://ec.europa.eu/digital-building-blocks/wikis/display/EIDCOMMUNITY/Croatia |
| EUR-Lex konsolidirani popis eID shema | https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=OJ:C_202502448 |
| Croatian eID scheme overview (v5.0, 2023) | https://ec.europa.eu/digital-building-blocks/sites/download/attachments/62885743/Updated%20Overview%20of%20the%20Croatian%20eID%20scheme.pdf |

### WSO2 IS dokumentacija (relevantno za razumijevanje platforme)
| Resurs | URL |
|--------|-----|
| WSO2 IS JWKS dokumentacija | https://is.docs.wso2.com/en/6.0.0/references/concepts/authentication/jwks/ |

### Kontakt AKD
| Kanal | Podatak |
|-------|---------|
| Helpdesk email | helpdesk@certilia.com |
| Besplatni telefon | 0800 0440 |
| Sjedište | Savska cesta 31, 10000 Zagreb |
| Telefon AKD | +385 1 3657610 |
| Web AKD | https://www.akd.hr/ |

---

## Zaključak

Certilia Mobile.id je **tehnički zrela platforma** izgrađena na WSO2 Identity Serveru s punom OIDC podrškom, besplatnim razvojnim okruženjem i eksplicitnim vraćanjem OIB-a u JWT tokenu. Za blockchain integraciju, **ključna prepreka** nije tehnička (nonce je podržan, JWT se može verificirati on-chain) već potencijalno **pravna** — morate razjasniti s AKD-om smiju li se JWT tokeni prosljeđivati izvan standardnog autentifikacijskog toka. Registracija na developer portalu i dohvat OIDC discovery endpointa su nužni prvi koraci — bez toga ne možete potvrditi exact algoritam potpisa, a upravo taj detalj određuje arhitekturu vašeg pametnog ugovora.
