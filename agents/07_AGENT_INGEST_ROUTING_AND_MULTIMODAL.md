# Agent: multimodální ingest, routing a fallback

## Mise

Dodej nebo zkontroluj ingest tak, aby šel přes řízený routing a bounded fallback, ne přes slepé timeoutové řetězení.

## Primární scope

- source-profile routing
- fallback pořadí
- quality gate
- incident stop
- retry po incidentu
- worker hranice pro ingest/render/screenshot/extract

## Mimo scope

- OIDC
- detailní UI formuláře
- fulltext relevance tuning
- supply-chain a CI

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 2. Co zůstává ze zdrojového zadání` (ř. 47–62)
  - `## 3.6 Multimodální ingest` (ř. 209–228)
  - `## 3.7 Consent, audit a evidence` (ř. 229–258)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), bod 8
  - `### 7.2 Batch vrstva v MetaVO / MetaCentru` (ř. 456–463)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.3 workers` (ř. 115–137)
  - `## 4.4 packages` (ř. 138–198), hlavně `packages/source-routing/`
  - `## 6. Co smí být fyzicky mimo repozitář` (ř. 289–310)

## Konkrétní TODO

- [ ] Zachovej pořadí strategií: API/feed -> HTML -> Rendered DOM -> Screenshot -> Upstream AI provider.
- [ ] Připrav source-profile routing per source.
- [ ] Zaveď bounded fallback s limitem kroků.
- [ ] Zaveď quality gate pro přechod na další strategii.
- [ ] Zaveď incident stop při CAPTCHA/blokaci/licenčním problému.
- [ ] Připrav retry cestu až po vyřešení incidentu.
- [ ] Vymez odpovědnosti workerů `ingest`, `render`, `screenshot`, `extract`.
- [ ] Ulož evidence artefakty do správných storage hranic.

## Cílové cesty v repo

- `packages/source-routing/`
- `workers/ingest-worker/`
- `workers/render-worker/`
- `workers/screenshot-worker/`
- `workers/extract-worker/`
- `workers/incident-retry-worker/`
- `packages/object-store/`
- `packages/domain-model/`

## Očekávané výstupy

- routing pravidla
- fallback policy
- quality score přechody
- incident workflow napojení
- output mapa artefaktů a metadat

## Akceptační kritéria

- není slepý sekvenční fallback bez quality gate
- incidenty skutečně zastaví workflow
- retry je oddělené a auditovatelné
- worker hranice jsou čisté a bez duplikovaných pravidel

## Handoff

- agent `05` dostane flow a worker body
- agent `06` dostane storage/search outputy
- agent `08` dostane incident a evidence hooky
- agent `12` dostane e2e scénáře `source -> ingest -> evidence`
