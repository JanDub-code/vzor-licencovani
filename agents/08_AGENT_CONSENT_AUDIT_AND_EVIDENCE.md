# Agent: consent, audit, evidence a incidenty

## Mise

Dodej nebo zkontroluj hard Consent Gate, append-only audit a evidenční model tak, aby šlo systém právně a provozně obhájit.

## Primární scope

- UI hard gate
- API enforcement
- append-only audit
- consent records
- evidence artefakty
- incident lifecycle
- role curator/admin návaznosti

## Mimo scope

- scheduler adapter
- detailní hybrid retrieval tuning
- K8s deployment manifesty
- load tuning

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 2. Co zůstává ze zdrojového zadání` (ř. 47–62)
  - `## 3.4 Search a retrieval hranice` (ř. 135–180), hlavně object store
  - `## 3.7 Consent, audit a evidence` (ř. 229–258)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), bod 9
  - `## 5.2 Fáze B` (ř. 377–386), consent enforcement a audit trail
  - `### 6.1 Co musí být uvnitř monorepa` (ř. 406–419)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 2. Co má struktura monorepa sledovat` (ř. 26–40), bod 5
  - `## 4.1 apps` (ř. 58–92)
  - `## 4.4 packages` (ř. 138–198), hlavně `packages/audit-consent-model/`
  - `## 4.6 docs` (ř. 242–253)

## Konkrétní TODO

- [ ] Zajisti UI hard gate před scrapingem, screeningem a citlivou analýzou.
- [ ] Zajisti API enforcement přes `consent_token` nebo `ConsentRecord`.
- [ ] Zajisti append-only audit s verzí textu, scope, časem, identitou, IP a zdrojem/datasetem.
- [ ] Doplň nebo zkontroluj tabulku `consent_record`.
- [ ] Zajisti povinné checkboxy a jejich verzování.
- [ ] Připrav vazbu incidentů na evidence artefakty a retry workflow.
- [ ] Zkontroluj, že screenshoty, PDF a DOM snapshoty jsou dohledatelné přes audit/citace.

## Cílové cesty v repo

- `packages/audit-consent-model/`
- `packages/domain-model/`
- `apps/api/`
- `apps/web/`
- `packages/object-store/`
- `docs/` (policy, playbook, ADR)

## Očekávané výstupy

- consent model
- audit event model
- incident/evidence vazby
- API a UI enforcement body
- seznam povinných dat polí a retention vazeb

## Akceptační kritéria

- bez souhlasu nejde spustit zakázaná akce
- audit je append-only a dohledatelný
- incidenty jsou navázané na artefakty a retry
- evidence artefakty jsou jednoznačně dohledatelné

## Handoff

- agent `04` dostane API/UI enforcement body
- agent `05` dostane incident retry hooky
- agent `07` dostane incident stop pravidla
- agent `11` dostane compliance a governance návaznosti
- agent `12` dostane contract/e2e scénáře
