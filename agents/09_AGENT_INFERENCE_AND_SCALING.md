# Agent: inference contract a škálování

## Mise

Udrž jednotný inference contract a škálovací model, který odděluje front-door throughput od reálné generativní kapacity. Nesmí vzniknout přímé napojení aplikace na backend bez contract vrstvy.

## Primární scope

- inference contract
- mapování online a batch backendů
- vLLM vs llama.cpp + GGUF
- timeouty, retry, fallback
- tracing identifikace volání
- L1/L2 cache
- service classes
- async slow-path
- admission control

## Mimo scope

- detailní UI
- konkrétní OpenSearch index mappingy
- detailní consent tabulky
- CI signing pipeline

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 3.3 Hranice systému a finální vítězové` (ř. 109–134), vLLM, llama.cpp, Redis
  - `## 3.5 Inference contract` (ř. 181–208)
  - `## 3.8 Škálovací model` (ř. 259–275)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), body 7 a 10
  - `### 7.1 Online vrstva` (ř. 440–455)
  - `### 7.2 Batch vrstva` (ř. 456–463)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 2. Co má struktura monorepa sledovat` (ř. 26–40), bod 3
  - `## 4.4 packages` (ř. 138–198), hlavně `packages/inference-contract/`, `packages/config/`
  - `## 4.7 tests` (ř. 254–270)

## Konkrétní TODO

- [ ] Udrž jedinou vstupní boundary `API / orchestrátor -> inference contract -> backend`.
- [ ] Doplň request/response model, metadata, timeouty a fallback pravidla.
- [ ] Rozliš online backend (`vLLM`) a batch backend (`llama.cpp + GGUF`) bez změny contractu.
- [ ] Připrav tracing a auditovatelnou identifikaci volání.
- [ ] Zaveď L1 cache v API procesu.
- [ ] Zaveď Redis jako L2 cache a rate-limit store.
- [ ] Připrav service classes `FAST_PATH`, `STANDARD`, `SLOW_PATH`.
- [ ] Připrav async režim pro delší odpovědi a batch-like úlohy.
- [ ] Připrav admission control a kvóty na gateway/API vrstvě.

## Cílové cesty v repo

- `packages/inference-contract/`
- `packages/config/`
- `apps/api/`
- `workers/embed-worker/`
- `workers/vlm-worker/`
- `services/scheduler-adapter/`
- `infra/redis/`

## Očekávané výstupy

- contract schémata
- backend mapping
- cache a service-class návrh
- risk list pro inference throughput
- handoff pro API, workflow a testy

## Akceptační kritéria

- aplikace nejde napřímo na backend mimo contract vrstvu
- online a batch inference se liší backendem, ne contractem
- škálování nerozlišuje jen RPS, ale i inference capacity
- Redis je L2 a rate-limit store, L1 zůstává v procesu

## Handoff

- agent `04` dostane API boundary
- agent `05` dostane workflow napojení
- agent `06` dostane retrieval -> generation vstup
- agent `10` dostane infra požadavky na Redis a inference runtime
- agent `12` dostane contract/load scénáře
