# Agent: architektonické hranice a monorepo

## Mise

Připrav a udrž monorepo strukturu tak, aby přesně odrážela uzamčené hranice systému. Tvým cílem není implementovat celé služby, ale **správně rozdělit vlastnictví, cesty a kontrakty**.

## Primární scope

- top-level struktura monorepa
- rozpad na `apps/`, `services/`, `workers/`, `packages/`, `infra/`, `docs/`, `tests/`
- ownership mapa
- pravidla co může a nemůže být mimo repo
- minimální startovní layout
- návaznost na ostatní agenty

## Mimo scope

- detailní implementace API endpointů
- detailní implementace workerů
- konkrétní search logika
- konkrétní infrastruktura clusteru

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `### 1.2 Monorepo je povinné` (ř. 29–46)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), bod 11
  - `## 6. Monorepo governance` (ř. 402–437)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 1. Závazná zásada` (ř. 11–25)
  - `## 2. Co má struktura monorepa sledovat` (ř. 26–40)
  - `## 3. Doporučená top-level struktura` (ř. 41–55)
  - `## 4. Význam jednotlivých částí` (ř. 56–270)
  - `## 5. Doporučená ownership mapa` (ř. 271–288)
  - `## 6. Co smí být fyzicky mimo repozitář` (ř. 289–310)
  - `## 7. Co se v monorepu nemá dělat` (ř. 311–322)
  - `## 8. Minimální startovní layout` (ř. 323–353)

## Konkrétní TODO

- [ ] Navrhni nebo uprav top-level strom repozitáře podle uzamčených hranic.
- [ ] Připrav seznam povinných složek, které musí vzniknout hned.
- [ ] Rozděl ownership po hranicích, ne po technologiích.
- [ ] Zapiš, co je sdílený package a co je samostatná služba.
- [ ] Vymez, co smí žít mimo repo, ale musí mít adaptery a testy uvnitř repo.
- [ ] Zkontroluj, že žádná core část nekončí v separátním repozitáři.
- [ ] Připrav handoff soubory/cesty pro ostatní agenty.

## Očekávané výstupy

- finální strom adresářů
- mapa ownershipu
- seznam povinných kontraktů a jejich umístění
- seznam cross-cutting package modulů
- stručný seznam „co sem nepatří“

## Cílové cesty v repo

- `/apps/`
- `/services/`
- `/workers/`
- `/packages/`
- `/infra/`
- `/docs/`
- `/tests/`

## Akceptační kritéria

- repo layout odpovídá systémovým hranicím
- každá důležitá hranice má jasné místo v repo
- nejsou zavedené ad hoc složky typu `scripts/temporary`, `experimental-auth`, `legacy-search`
- ownership je čitelný a nevytváří konflikt mezi agenty

## Handoff

Po dokončení předej:
- agentovi `03` služby a hranici scheduler adapteru
- agentovi `04` umístění `apps/api`, `apps/web`, `packages/auth`
- agentovi `05` umístění `packages/prefect-flows`, `packages/queue-jobs`, `workers/`
- agentovi `06` umístění search/storage package modulů
- agentovi `10` umístění celé `infra/`
- agentovi `12` umístění `tests/`
