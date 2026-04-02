# Deep Research: 0xSplits i razdjelnici plaćanja na Gnosis lancu

## Kontekst za istraživača

Trebam mehanizam za automatsku raspodjelu EURe (Monerium stabilni token) na Gnosis lancu: 90% kreatoru, 10% platformskom Safe trezoru. Razmatram 0xSplits ili vlastitu implementaciju.

## Pitanja na koja trebam odgovore

### 1. 0xSplits na Gnosisu

- Je li 0xSplits v2 deployran na Gnosis lancu (chain ID 100)?
- Ako da, koje su adrese ugovora?
- Koja je verzija? (v1, v2?)
- Podržava li 0xSplits ERC-20 tokene (konkretno EURe) ili samo nativni token (xDAI)?
- Kako funkcionira distribucija? Push (automatski pri uplati) ili Pull (primatelji sami povlače)?
- Koliko goriva troši kreiranje novog splita?
- Koliko goriva troši distribucija?
- Je li 0xSplits revidiran? Tko ga je revidirao?
- Postoji li 0xSplits SDK za JavaScript/TypeScript?

### 2. Alternativni razdjelnici

- Postoje li drugi protokoli za raspodjelu sredstava na Gnosisu?
- Superfluid (streaming plaćanja) — podržava li Gnosis? Relevantno za pretplate?
- Sablier — podržava li Gnosis?
- Drips — podržava li Gnosis?
- OpenZeppelin PaymentSplitter — je li dovoljno za naše potrebe?

### 3. EURe (Monerium) specifičnosti

- Je li EURe standardni ERC-20 na Gnosisu?
- Ima li EURe posebne funkcije (mint/burn/freeze)?
- Koja je adresa EURe ugovora na Gnosis lancu?
- Postoje li poznata ograničenja pri korištenju EURe s pametnim ugovorima?
- Monerium API — postoji li ulazna rampa za SEPA → EURe na Gnosisu?
- Može li Monerium zamrznuti tokene na specifičnim adresama? Ako da, pod kojim uvjetima?

### 4. Vlastita implementacija

- Ako 0xSplits nije dostupan na Gnosisu, koliko je složeno napisati vlastiti razdjelnik?
- Koji su minimalni zahtjevi? (ERC-20 approve/transferFrom, fiksni udjeli)
- Sigurnosna razmatranja za vlastiti razdjelnik?
- Postoje li revidirane minimalne implementacije razdjelnika koje bismo mogli koristiti kao temelj?

## Format odgovora

Tablica usporedbe svih opcija:

| Opcija | Gnosis podrška | ERC-20 podrška | Revidirana | Gorivo (kreiranje) | Gorivo (distribucija) | Složenost integracije |
|--------|---------------|----------------|------------|--------------------|-----------------------|----------------------|

Sav izlaz na **hrvatskom jeziku**.
