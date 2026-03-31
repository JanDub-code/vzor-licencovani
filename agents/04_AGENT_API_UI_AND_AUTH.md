# Agent: API, web UI a auth

## Mise

Dodej nebo udrž vstupní aplikace a autentizaci tak, aby odpovídaly finálním hranicím: FastAPI + web UI + OIDC bez lokálních auth zkratek.

## Primární scope

- `apps/api/`
- `apps/web/`
- OIDC integrace a auth boundary
- request validation
- service class routing hooky
- consent enforcement hooky
- audit hooky
- základní UI pro správu zdrojů a incidentů

## Mimo scope

- implementace scheduler adapteru
- detailní multimodální ingest
- fulltext/hybrid relevance tuning
- supply-chain pipeline

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `### 1.1 Závazná zásada bez mezikroků` (ř. 12–28)
  - `## 3.2 Packaging` (ř. 80–108)
  - `## 3.3 Hranice systému a finální vítězové` (ř. 109–134), auth a Redis
  - `## 3.7 Consent, audit a evidence` (ř. 229–258)
  - `## 3.8 Škálovací model` (ř. 259–275)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), body 6, 9, 10
  - `### 7.1 Online vrstva` (ř. 440–455)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.1 apps` (ř. 58–92)
  - `## 4.4 packages` (ř. 138–198), hlavně `packages/auth/`
  - `## 5. Doporučená ownership mapa` (ř. 271–288)

## Konkrétní TODO

- [ ] Udrž nebo doplň OIDC boundary bez lokální JWT náhrady.
- [ ] Připrav API endpointy tak, aby uměly auth, rate limit hooky, enqueue, retrieval orchestration a volání inference contractu.
- [ ] Doplň UI pro správu zdrojů, consent, incidenty a preview evidence/citací.
- [ ] Napoj consent enforcement na API i UI.
- [ ] Připrav service class routing hooky (`FAST_PATH`, `STANDARD`, `SLOW_PATH`).
- [ ] Zajisti, že API neimplementuje business pravidla bokem mimo shared packages.
- [ ] Připrav jasné rozhraní pro komunikaci s workflow, search a audit vrstvou.

## Cílové cesty v repo

- `apps/api/`
- `apps/web/`
- `packages/auth/`
- `packages/config/`
- `packages/audit-consent-model/`
- `infra/keycloak/` nebo jiná OIDC konfigurace bez změny kontraktu

## Očekávané výstupy

- seznam endpointů / UI obrazovek / middleware hooků
- auth integration plan nebo implementace
- kontrola kde přesně sedí consent gate
- mapování API na ostatní kontrakty

## Akceptační kritéria

- auth je OIDC-first
- UI i API respektují hard consent gate
- API neobsahuje ad hoc alternativní auth ani dočasný queue/search model
- je jasné, kudy request teče k retrieval a inference contractu

## Handoff

- agent `05` dostane body napojení enqueue/orchestrace
- agent `06` dostane body napojení retrieval/search
- agent `08` dostane místa, kde musí být consent a audit enforcement
- agent `09` dostane vstup do inference contractu
