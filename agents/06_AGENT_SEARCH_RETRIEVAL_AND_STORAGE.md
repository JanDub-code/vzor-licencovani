# Agent: search, retrieval a storage boundary

## Mise

Udrž jasnou hranici mezi PostgreSQL, object store, Qdrant a OpenSearch. Tvým cílem je zabránit smíchání odpovědností a dočasným náhradám.

## Primární scope

- metadata vs evidence vs vector vs fulltext boundary
- search contract
- object store contract
- Qdrant client
- OpenSearch client
- metadata filtry pro RAG
- hybrid retrieval hranice

## Mimo scope

- detailní UI
- detailní auth
- detailní worker orchestrace
- observability dashboardy

## Povinné vstupy

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 3.3 Hranice systému a finální vítězové` (ř. 109–134)
  - `## 3.4 Search a retrieval hranice` (ř. 135–180)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328), bod 3
  - `### 7.1 Online vrstva` (ř. 440–455)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 2. Co má struktura monorepa sledovat` (ř. 26–40), bod 2
  - `## 4.4 packages` (ř. 138–198), hlavně `search-contract`, `object-store`, `qdrant-client`, `opensearch-client`
  - `## 4.5 infra` (ř. 199–241)

## Konkrétní TODO

- [ ] Zafixuj co patří do PostgreSQL a co už ne.
- [ ] Zafixuj co patří do object store.
- [ ] Zafixuj co patří do Qdrant.
- [ ] Zafixuj co patří do OpenSearch.
- [ ] Připrav nebo zkontroluj `search-contract` a klienty.
- [ ] Dohlédni, aby se OpenSearch nenahrazoval PostgreSQL fulltextem.
- [ ] Dohlédni, aby evidence artefakty šly do object store a ne do metadatové DB.
- [ ] Dohlédni, aby retrieval metadata byla kompatibilní s RAG i citacemi.

## Cílové cesty v repo

- `packages/search-contract/`
- `packages/object-store/`
- `packages/qdrant-client/`
- `packages/opensearch-client/`
- `packages/domain-model/`
- `infra/postgres/`
- `infra/qdrant/`
- `infra/opensearch/`

## Očekávané výstupy

- tabulka odpovědností úložišť
- kontrakty a klientské boundary
- návrh indexační a retrieval cesty
- seznam zakázaných anti-patternů

## Akceptační kritéria

- není smíchaná role PostgreSQL a OpenSearch
- object store je oddělený od metadata DB
- retrieval cesta má jasnou hranici lexical/hybrid/vector
- není zaveden mezikrok s jiným storage/search stackem

## Handoff

- agent `04` dostane retrieval API boundary
- agent `07` dostane ingest outputy a evidence targety
- agent `09` dostane vstup do inference/RAG vrstvy
- agent `12` dostane contract a e2e scénáře
