# Agent: online vs batch a scheduler adapter

## Mise

Udrž tvrdé oddělení mezi online vrstvou a batch vrstvou. Navrhni nebo implementuj scheduler adapter a batch packaging bez porušení finální architektury.

## Primární scope

- oddělení online runtime vs batch runtime
- OpenPBS-first scheduler adapter
- Apptainer/Singularity packaging pro batch
- napojení batch jobů na orchestrace a workery
- definice rozhraní submit/status/cancel

## Mimo scope

- UI obrazovky
- detailní search schémata
- detailní consent model
- detailní observability dashboardy

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 3.1 Provozní model` (ř. 65–79)
  - `## 3.2 Packaging` (ř. 80–108)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), body 1 a 2
  - `## 5.1 Fáze A` (ř. 353–376)
  - `## 5.2 Fáze B` (ř. 377–386)
  - `## 5.3 Fáze C` (ř. 387–401)
  - `### 7.2 Batch vrstva v MetaVO / MetaCentru` (ř. 456–463)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.2 services` (ř. 93–114)
  - `## 4.3 workers` (ř. 115–137)
  - `## 4.5 infra` (ř. 199–241)

## Konkrétní TODO

- [ ] Vymez přesně, co běží online a co jde přes batch.
- [ ] Připrav kontrakt scheduler adapteru: submit/status/cancel.
- [ ] Ujasni, jak se batch metadata přenáší z orchestrace do OpenPBS.
- [ ] Připrav obal pro Apptainer joby a reference šablony.
- [ ] Zajisti, že online služby nikdy neběží jako batch job.
- [ ] Připrav integrační body pro Prefect a workery.
- [ ] Sepiš limity a selhání: retry, cancel, stav jobu, incidenty.

## Cílové cesty v repo

- `services/scheduler-adapter/`
- `infra/apptainer/`
- `infra/k8s/`
- `workers/`
- případně `packages/config/` a `packages/shared-utils/`

## Očekávané výstupy

- rozhraní scheduler adapteru
- mapování job statusů
- packaging pravidla pro batch kontejnery
- handoff pro workery a workflow vrstvu
- seznam zakázaných zkratek a co nahradit nesmí

## Akceptační kritéria

- adapter je OpenPBS-first
- není tam Slurm hardlock
- online a batch cesty jsou čitelné a oddělené
- lokální prostředí používá stejnou logiku, jen menší balení

## Handoff

- agent `05` dostane kontrakt pro napojení Prefect flow na batch
- agent `09` dostane napojení batch inference backendu
- agent `10` dostane požadavky na `infra/apptainer/` a `infra/k8s/`
- agent `12` dostane kontrakty pro contract/integration testy
