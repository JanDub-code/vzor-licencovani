# Agent: observability, compliance a supply chain

## Mise

Zajisti, aby observability byla povinná od začátku a aby modely, kontejnery, object store a log backend prošly správným governance/compliance rámcem.

## Primární scope

- metriky
- strukturované logy
- tracing
- korelace request -> job -> inference -> audit
- model intake policy
- artifact allowlist/mirror
- checksumy, SBOM, signing
- compliance gate pro MinIO a Loki
- provozní runbooky a ADR

## Mimo scope

- detailní UI
- detailní business logika workerů
- detailní OpenSearch relevance
- detailní scheduler adapter implementation

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 3.9 Observability, compliance a supply chain` (ř. 276–308)
  - `## 4.2 Musí být navrženo hned, ale lze prohlubovat později` (ř. 329–338)
  - `## 4.3 Může se doplnit až po funkčním jádru` (ř. 339–350)
  - `## 5.3 Fáze C` (ř. 387–401)
  - `### 6.1 Co musí být uvnitř monorepa` (ř. 406–419)
  - `### 7.1 Online vrstva` (ř. 440–455)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.4 packages` (ř. 138–198), hlavně `packages/observability/`
  - `## 4.5 infra` (ř. 199–241), hlavně `infra/monitoring/`, `infra/ci/`
  - `## 4.6 docs` (ř. 242–253)

## Konkrétní TODO

- [ ] Zajisti metriky, strukturované logy a tracing od začátku.
- [ ] Doplň korelační identifikátory napříč requestem, jobem, inference a auditem.
- [ ] Připrav základ dashboardů a alertingu bez čekání na pozdější fázi.
- [ ] Připrav Model Intake Policy.
- [ ] Zaveď allowlist/mirror pro modely, checksumy, SBOM a signing.
- [ ] Zapiš compliance gate pro MinIO a Loki včetně alternativ.
- [ ] Připrav runbooky, ADR a provozní rozhodovací body.

## Cílové cesty v repo

- `packages/observability/`
- `infra/monitoring/`
- `infra/ci/`
- `docs/`
- případně hooky v `apps/`, `services/`, `workers/`

## Očekávané výstupy

- observability baseline
- korelační schema
- model intake policy
- compliance decision log
- seznam povinných provozních dokumentů

## Akceptační kritéria

- observability není odložená na „později“
- supply-chain pravidla nejsou jen text, ale mají konkrétní kontrolní body
- compliance-dependent komponenty mají variantu a rozhodovací workflow
- každý runtime má jasný telemetry a logging kontrakt

## Handoff

- agent `08` dostane návaznost na audit
- agent `10` dostane konkrétní infra požadavky
- agent `12` dostane kontrolní body do CI a testů
