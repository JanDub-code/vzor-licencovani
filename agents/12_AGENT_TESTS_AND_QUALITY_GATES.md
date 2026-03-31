# Agent: testy a quality gates

## Mise

Zajisti, aby testovací strategie hlídala kontrakty, e2e tok a load chování přesně podle uzamčených hranic systému.

## Primární scope

- unit testy
- integration testy
- e2e testy
- contract testy
- load testy
- quality gates v CI
- smoke testy pro externí nebo spravované závislosti

## Mimo scope

- detailní UI design
- detailní cluster operations
- právní formulace consent textu
- relevance tuning modelů

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 3.5 Inference contract` (ř. 181–208)
  - `## 3.6 Multimodální ingest` (ř. 209–228)
  - `## 3.7 Consent, audit a evidence` (ř. 229–258)
  - `## 3.8 Škálovací model` (ř. 259–275)
  - `## 5.1 Fáze A` (ř. 353–376)
  - `## 5.2 Fáze B` (ř. 377–386)
  - `## 5.3 Fáze C` (ř. 387–401)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.7 tests` (ř. 254–270)
  - `## 6. Co smí být fyzicky mimo repozitář` (ř. 289–310)

## Konkrétní TODO

- [ ] Rozděl testy na `unit`, `integration`, `e2e`, `contract`, `load`.
- [ ] Doplň contract testy pro inference contract, scheduler adapter a consent/audit contract.
- [ ] Doplň e2e tok `source -> ingest -> evidence -> retrieval -> answer -> citation`.
- [ ] Doplň load scénáře, které rozlišují front-door throughput a inference throughput.
- [ ] Doplň smoke testy pro spravované externí závislosti.
- [ ] Doplň quality gates do CI tak, aby bránily porušení architektonických hranic.

## Cílové cesty v repo

- `tests/unit/`
- `tests/integration/`
- `tests/e2e/`
- `tests/contract/`
- `tests/load/`
- `infra/ci/`

## Očekávané výstupy

- test matrix
- minimální povinné scénáře
- CI gate seznam
- mapování testů na architektonické hranice

## Akceptační kritéria

- testy hlídají kontrakty, ne jen happy-path
- load testy nerozbijí rozdíl mezi RPS a generativní kapacitou
- e2e scénář obsahuje evidence a citace
- externí spravované služby mají smoke/health contract

## Handoff

- vrací se všem doménovým agentům jako kontrolní vrstva
- zvlášť informuje agenty `03`, `08`, `09` o contract testech
