# Deep Research: Certilia Mobile.id tehničke specifikacije

## Kontekst za istraživača

Gradim otvoreni sustav koji koristi hrvatski eIDAS digitalni identitet (Certilia Mobile.id, izdaje AKD — Agencija za komercijalnu djelatnost) za lančanu verifikaciju identiteta na Gnosis blockchainu. Trebam ČINJENIČNE, provjerljive informacije — ne spekulacije.

Certilia Mobile.id je OIDC (OpenID Connect) pružatelj koji omogućuje hrvatskim građanima prijavu na e-usluge koristeći mobitel i biometriju. Ja ga želim koristiti na nestandardan način: proslijediti JWT koji Certilia izda pametnom ugovoru na blockchainu koji će ga kriptografski verificirati.

## Pitanja na koja trebam odgovore

### 1. OIDC konfiguracija (KRITIČNO)

- Koja je Certilia OIDC Discovery URL? (obično `/.well-known/openid-configuration`)
- Koji je `issuer` u konfiguraciji?
- Koji su podržani `response_types`, `grant_types`, `scopes`?
- Podržava li Certilia **prilagođeni `nonce` parametar** u autorizacijskom zahtjevu? (Ovo je apsolutno kritično — trebam poslati Ethereum adresu kao nonce.)
- Podržava li `state` parametar za CSRF zaštitu?
- Koji je format povratnog URL-a (redirect_uri)?

### 2. JWT potpis (KRITIČNO)

- Koji algoritam koristi Certilia za potpisivanje JWT-ova? (RSA-256, RS256, ES256/P-256, PS256, ili nešto drugo?)
- Koja je veličina ključa? (RSA-2048, RSA-4096, P-256 krivulja?)
- Gdje je JWKS (JSON Web Key Set) krajnja točka za dohvat javnog ključa?
- Koliko ključeva je obično u JWKS-u? (Jedan ili više — bitno za rotaciju.)
- Koliko često se ključevi rotiraju?
- Koji je format `kid` (Key ID) u JWT zaglavlju?

### 3. JWT sadržaj (KRITIČNO)

- Koje tvrdnje (claims) sadrži JWT? Konkretno:
  - `sub` — što je ovo? Hash OIB-a? Interni identifikator? Pseudonim? Je li deterministički jedinstven po građaninu (isti `sub` uvijek za istu osobu)?
  - `name`, `given_name`, `family_name` — jesu li prisutni?
  - `email` — je li prisutan?
  - `date_of_birth` — je li prisutan?
  - OIB — je li ikada eksplicitno prisutan u JWT-u?
- Koji je tipični `exp` (istek) JWT-a? 5 minuta? 1 sat?
- Koji je `aud` (publika) format?

### 4. Registracija klijenta

- Kako se registrira aplikacija kao "pouzdajuća strana" (relying party) kod AKD-a?
- Postoji li postupak samoposlužne registracije ili zahtijeva formalni ugovor s AKD-om?
- Postoji li **testno/sandbox okruženje** za razvoj bez pravih identiteta?
- Koji su uvjeti korištenja? Postoje li ograničenja namjene (npr. zabrana korištenja JWT-a izvan predviđene autentikacije)?
- Koliki je trošak registracije?

### 5. eIDAS razina osiguranja

- Na kojoj eIDAS razini osiguranja je Certilia? (niska / značajna / visoka)
- Je li Certilia notificirana prema eIDAS regulativi za prekograničnu upotrebu?
- Može li se koristiti za autentikaciju na e-usluge drugih EU država?

### 6. Tehnička infrastruktura

- Koji je pravni entitet iza Certilije? (AKD, FINA, ili drugo?)
- Koristi li Certilia standardnu OIDC biblioteku (Keycloak, IdentityServer, vlastito)?
- Postoji li javna API dokumentacija? Ako da, URL?
- Postoje li poznati SDK-ovi ili biblioteke za integraciju?

## Format odgovora

Molim strukturiraj odgovor ovako:

1. **Potvrđene činjenice** — samo ono što je verificirano iz službenih izvora (URL-ovi!)
2. **Djelomično potvrđeno** — informacije iz neslužbenih ali pouzdanih izvora
3. **Nepoznato / Zahtijeva direktan kontakt s AKD-om** — pitanja na koja nisi mogao naći odgovor
4. **Relevantni linkovi** — sve službene stranice, dokumentacija, API krajnje točke

Sav izlaz na **hrvatskom jeziku**.
