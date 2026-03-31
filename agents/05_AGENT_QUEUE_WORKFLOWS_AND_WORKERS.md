# Agent: RabbitMQ, Prefect, workflow a workers

## Mise

Postav nebo udrž workflow vrstvu kolem RabbitMQ a Prefect tak, aby workery pracovaly přes sdílené kontrakty a neobsahovaly vlastní skrytou business logiku.

## Primární scope

- RabbitMQ queue model
- Prefect flows
- flow contracts
- worker lifecycle
- retry pravidla
- incident retry cesta
- návaznost na batch adapter

## Mimo scope

- detailní UI
- detailní OIDC integrace
- konkrétní relevance tuning v OpenSearch
- CI supply-chain politika

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `### 1.1 Závazná zásada bez mezikroků` (ř. 12–28), RabbitMQ a Prefect
  - `## 3.3 Hranice systému a finální vítězové` (ř. 109–134), RabbitMQ a Prefect
  - `## 3.6 Multimodální ingest` (ř. 209–228)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), body 4, 5, 8
  - `## 5.1 Fáze A` (ř. 353–376)
  - `## 5.2 Fáze B` (ř. 377–386)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 4.2 services` (ř. 93–114)
  - `## 4.3 workers` (ř. 115–137)
  - `## 4.4 packages` (ř. 138–198), hlavně `packages/prefect-flows/`, `packages/rabbitmq-client/`, `packages/queue-jobs/`
  - `## 5. Doporučená ownership mapa` (ř. 271–288)

## Konkrétní TODO

- [ ] Potvrď RabbitMQ jako jediný queue model.
- [ ] Navrhni nebo doplň topologii front: workflow fronty, priority, DLX, ack/nack.
- [ ] Centralizuj Prefect flows do `packages/prefect-flows/`.
- [ ] Rozděl workery po jedné primární odpovědnosti.
- [ ] Zajisti, že workery jen vykonávají úlohu a neskrývají policy pravidla.
- [ ] Doplň `incident-retry-worker` nebo ekvivalentní retry cestu po vyřešení incidentu.
- [ ] Připrav flow návaznost online -> queue -> worker/batch -> audit -> handoff.

## Cílové cesty v repo

- `packages/prefect-flows/`
- `packages/rabbitmq-client/`
- `packages/queue-jobs/`
- `workers/ingest-worker/`
- `workers/render-worker/`
- `workers/screenshot-worker/`
- `workers/extract-worker/`
- `workers/embed-worker/`
- `workers/vlm-worker/`
- `workers/reindex-worker/`
- `workers/incident-retry-worker/`
- `infra/rabbitmq/`

## Očekávané výstupy

- frontová topologie
- seznam flow a jejich stavů
- worker matrix: vstup, výstup, retry, chyba, incident
- handoff na scheduler adapter, ingest, search a audit

## Akceptační kritéria

- žádná PostgreSQL queue
- žádné flow rozeseté náhodně po workerech
- worker pravidla jsou sdílená v package modulech
- retry a incident stop jsou jasně popsané nebo implementované

## Handoff

- agent `03` dostane požadavky na batch submit/status/cancel
- agent `07` dostane execution path pro ingest a multimodální fallback
- agent `08` dostane incident a audit hooky
- agent `12` dostane contract/integration scénáře
