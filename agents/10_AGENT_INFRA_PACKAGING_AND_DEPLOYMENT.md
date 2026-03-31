# Agent: infra, packaging a deployment

## Mise

Připrav infrastrukturu tak, aby lokální Compose byl věrný mirror finální architektury a produkce zůstala rozdělena na K8s online vrstvu a OpenPBS/Apptainer batch vrstvu.

## Primární scope

- `infra/local-compose/`
- `infra/k8s/`
- `infra/apptainer/`
- lokální topologie služeb
- wiring pro PostgreSQL, RabbitMQ, Redis, Qdrant, OpenSearch, object store, OIDC, monitoring
- sizing rozdíly mezi local/integration/prod bez změny hranic

## Mimo scope

- detailní obchodní logika workerů
- detailní UI obrazovky
- detailní contract schémata
- právní texty consentu

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 3.2 Packaging` (ř. 80–108)
  - `## 3.3 Hranice systému a finální vítězové` (ř. 109–134)
  - `## 3.9 Observability, compliance a supply chain` (ř. 276–308)
  - `## 5.1 Fáze A` (ř. 353–376)
  - `## 5.2 Fáze B` (ř. 377–386)
  - `## 5.3 Fáze C` (ř. 387–401)
  - `### 6.1 Co musí být uvnitř monorepa` (ř. 406–419)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.5 infra` (ř. 199–241)
  - `## 8. Minimální startovní layout` (ř. 323–353)

## Konkrétní TODO

- [ ] Udrž stejnou topologii komponent mezi local a prod.
- [ ] Doplň local compose pro všechny povinné komponenty.
- [ ] Rozděl online deployment do K8s/Cloud a batch do OpenPBS + Apptainer.
- [ ] Zajisti, že lokálně se liší jen sizing a typ instance, ne kontrakt ani hranice.
- [ ] Připrav wiring pro PostgreSQL, RabbitMQ, Redis, Qdrant, OpenSearch, object store, OIDC, Prometheus, Grafana, Loki.
- [ ] Zapiš compliance-dependent varianty pro object store a log backend.
- [ ] Připrav secrets a konfiguraci tak, aby šly hardenovat ve fázi C.

## Cílové cesty v repo

- `infra/local-compose/`
- `infra/k8s/`
- `infra/apptainer/`
- `infra/postgres/`
- `infra/rabbitmq/`
- `infra/redis/`
- `infra/qdrant/`
- `infra/opensearch/`
- `infra/keycloak/`
- `infra/monitoring/`
- `infra/ci/`

## Očekávané výstupy

- compose topologie
- k8s deployment/service/ingress návrh
- apptainer image a submit wrapper návrh
- config matrix local/integration/prod
- seznam compliance toggle bodů

## Akceptační kritéria

- lokální stack je věrná miniatura finální architektury
- není tam alternativní MVP stack
- produkční packaging nemění systémové hranice
- batch a online deployment jsou oddělené

## Handoff

- agent `03` dostane infrastrukturu pro scheduler/batch
- agent `04` dostane OIDC a app wiring
- agent `05` dostane RabbitMQ/Prefect wiring
- agent `09` dostane Redis a inference runtime wiring
- agent `11` dostane monitoring/logging/supply-chain body
- agent `12` dostane integrační prostředí pro testy
