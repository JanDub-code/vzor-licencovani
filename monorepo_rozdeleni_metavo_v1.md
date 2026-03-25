# Monorepo rozdělení pro MetaVO / MetaCentrum

Tento soubor je doplňkem k hlavnímu návrhu:

- **Hlavní architektura:** `finalizovany_navrh_metavo_v5.md`

Tento dokument nerozhoduje o architektuře znovu. Rozvádí pouze to, **jak se uzamčené systémové hranice z hlavního návrhu promítnou do struktury monorepa**. [M1][M2][M3]

---

## 1. Závazná zásada

> **Monorepo je jediný zdroj pravdy pro kontrakty, doménový model, workflow, infrastrukturu, testy a dokumentaci, i když některé runtime komponenty fyzicky běží mimo hlavní cluster nebo mimo správu týmu.** [M1][M2]

To znamená:

- žádné vedlejší „dočasné“ repozitáře pro core služby,
- žádné oddělené repozitáře pro workery, které sdílejí stejný doménový model,
- žádné oddělené repozitáře pro infra definice, pokud se týkají stejného systému,
- žádné „experimentální“ forky kontraktů mimo monorepo.

Výjimky mohou být jen u cizích nebo spravovaných systémů, které tým nevlastní, například institucionální IdP nebo externí API. Ale i tam musí být v monorepu konfigurace, adaptéry, testy a dokumentace rozhraní. [M4][M5]

---

## 2. Co má struktura monorepa sledovat

Struktura repozitáře se neřídí podle technologií samotných. Řídí se podle uzamčených hranic systému z hlavního návrhu:

1. **online vs batch** [M2]
2. **PostgreSQL vs object store vs search vs vector boundary** [M3]
3. **inference contract** [M4]
4. **scheduler adapter** [M2]
5. **consent / audit / evidence model** [M5]
6. **source routing a fallback logika** [M6]

Proto má být monorepo rozdělené podle odpovědností a kontraktů, ne podle toho, co je „backend“ a co je „skript“. 

---

## 3. Doporučená top-level struktura

```text
/
├── apps/
├── services/
├── workers/
├── packages/
├── infra/
├── docs/
├── tests/
├── tools/
├── data-contracts/
└── .github/   (nebo ekvivalent CI/CD složky)
```

---

## 4. Význam jednotlivých částí

## 4.1 `apps/`

Sem patří vstupní aplikace a řídicí rozhraní.

```text
apps/
├── api/                # hlavní FastAPI backend
├── web/                # admin / curator / analyst UI
└── batch-control/      # volitelná tenká control-plane aplikace pro batch operace
```

### `apps/api/`

Musí obsahovat:

- veřejné a interní API endpointy,
- auth integraci,
- request validation,
- rate limiting hooks,
- service class routing,
- enqueue logiku,
- retrieval orchestration,
- volání inference contractu,
- audit a consent enforcement. [M3][M4][M5]

### `apps/web/`

Musí obsahovat:

- správu zdrojů,
- curator/admin workflow,
- consent UI,
- incident UI,
- dohled nad ingestem,
- citace a evidence preview. [M5][M6]

## 4.2 `services/`

Sem patří samostatné běžící služby, které mají jasnou síťovou nebo procesní hranici.

```text
services/
├── inference-gateway/      # jen pokud bude použit guwp
├── retrieval-orchestrator/ # volitelné, pokud se retrieval oddělí od API
├── scheduler-adapter/      # OpenPBS adapter vrstva
└── policy-engine/          # volitelně separovaný policy / enforcement runtime
```

### `services/inference-gateway/`

Tato složka existuje jen pokud bude skutečně implementován `guwp`.

Musí zůstat tenká a držet se definice z hlavního návrhu:

- sjednocení přístupu na `vLLM` a externí API,
- OpenAI-like kontrakt,
- timeouty,
- fallback policy,
- tracing a auditovatelná identifikace volání. [M4]

Pokud `guwp` nebude, inference contract a adaptéry zůstanou v `packages/` a `apps/api/`. To je v souladu s hlavním dokumentem, podle něhož architektura na `guwp` nesmí být existenčně závislá. [M4]

### `services/scheduler-adapter/`

Musí obsahovat:

- OpenPBS-first adapter,
- submit/status/cancel rozhraní,
- mapování batch job metadata,
- zabalení parametrů pro Apptainer/Singularity joby,
- jednotné API pro batch orchestrace. [M2]

Tohle je tvrdá systémová hranice a nemá se rozpustit po workerech.

## 4.3 `workers/`

Sem patří dlouho běžící nebo dávkové pracovní procesy.

```text
workers/
├── ingest-worker/
├── render-worker/
├── screenshot-worker/
├── extract-worker/
├── embed-worker/
├── vlm-worker/
├── reindex-worker/
└── incident-retry-worker/
```

### Zásady pro `workers/`

- každý worker má jednu primární odpovědnost,
- worker nesmí skrývat business pravidla, která patří do sdílených package modulů,
- worker nesmí mít vlastní lokální interpretaci consent nebo audit modelu,
- worker používá stejné kontrakty jako API a orchestrace. [M3][M5][M6]

## 4.4 `packages/`

Sem patří sdílené knihovny, kontrakty a doménové moduly.

```text
packages/
├── domain-model/
├── inference-contract/
├── search-contract/
├── audit-consent-model/
├── source-routing/
├── observability/
├── config/
├── auth/
├── queue-jobs/
├── object-store/
├── qdrant-client/
├── opensearch-client/
├── rabbitmq-client/
├── prefect-flows/
└── shared-utils/
```

### Co sem patří povinně

#### `packages/domain-model/`

- entity jako source, document, chunk, citation, incident, consent,
- serializační a validační schémata,
- verzování schémat.

#### `packages/inference-contract/`

- jednotný request/response model pro inference,
- mapování na online a batch backendy,
- fallback a timeout policy,
- telemetry metadata. [M4]

#### `packages/audit-consent-model/`

- consent payloady,
- audit eventy,
- append-only log model,
- enforcement helpery. [M5]

#### `packages/source-routing/`

- source-profile routing,
- fallback pravidla,
- quality gate,
- incident stop pravidla. [M6]

#### `packages/prefect-flows/`

- centrální definice flow,
- retry policy,
- orchestration wrappers,
- flow contracts pro online i batch návaznosti. [M7]

Tady je důležité, aby Prefect flows nebyly rozkopané po náhodných worker repozitářích. Patří do jednoho sdíleného prostoru.

## 4.5 `infra/`

Sem patří všechny infra definice a lokální/prod packaging artefakty.

```text
infra/
├── local-compose/
├── k8s/
├── apptainer/
├── rabbitmq/
├── opensearch/
├── qdrant/
├── postgres/
├── redis/
├── keycloak/
├── monitoring/
└── ci/
```

### `infra/local-compose/`

Tato složka je kritická, protože podle hlavního návrhu je **lokální Docker mirror jediná povolená výjimka z produkčního balení**. Musí proto obsahovat stejnou topologii komponent jako cílová architektura v malém. [M1][M2]

### `infra/k8s/`

Sem patří:

- deploymenty,
- services,
- ingress,
- secrets wiring,
- autoscaling pravidla,
- policy a network rules.

### `infra/apptainer/`

Sem patří:

- definice image pro batch workloady,
- packaging skripty,
- OpenPBS submit wrappers,
- reference job templaty. [M2]

## 4.6 `docs/`

Sem patří:

- hlavní architektonický dokument,
- tento monorepo dokument,
- ADRs,
- provozní runbooky,
- compliance rozhodnutí,
- model intake policy,
- incident response playbook.

## 4.7 `tests/`

```text
tests/
├── unit/
├── integration/
├── e2e/
├── contract/
└── load/
```

### Zásady

- contract testy musí hlídat inference contract, scheduler adapter contract a consent/audit contract, [M2][M4][M5]
- e2e testy musí ověřovat kompletní tok source -> ingest -> evidence -> retrieval -> answer -> citation,
- load testy musí rozlišovat front-door throughput a inference throughput. [M8]

## 4.8 `data-contracts/`

Doporučená samostatná složka pro:

- JSON Schema / OpenAPI / protobuf / pydantic contracts,
- event schémata,
- export/import formáty,
- verze schémat mezi API, workers a orchestrace.

To je vhodné hlavně proto, aby se systémové kontrakty neukryly v implementaci jednoho frameworku.

## 4.9 `tools/`

Sem patří pomocné utility:

- migrace,
- seedery,
- validační skripty,
- benchmark harness,
- lokální bootstrap,
- kontrola checksumů modelů,
- SBOM generátory a release pomocníci. [M9]

---

## 5. Doporučená ownership mapa

```text
apps/api                    -> platform/backend owner
apps/web                    -> product/ui owner
services/scheduler-adapter  -> platform/hpc owner
services/inference-gateway  -> inference/platform owner
workers/*                   -> data-pipeline owner
packages/domain-model       -> architecture/platform owner
packages/inference-contract -> inference/platform owner
packages/audit-consent-model-> compliance/platform owner
infra/*                     -> platform/devops owner
docs/*                      -> architecture owner
```

Ownership nemusí znamenat oddělené týmy. Znamená to, že je vždy jasné, kdo hlídá kterou hranici.

---

## 6. Co smí být fyzicky mimo repozitář

Mimo repozitář mohou fyzicky žít tyto systémy:

- institucionální OIDC provider,
- spravovaný PostgreSQL,
- spravovaný OpenSearch,
- spravovaný object store,
- MetaVO OpenPBS infrastruktura,
- externí modelové API.

Ale v monorepu musí zůstat:

- klientské adaptéry,
- konfigurační šablony,
- healthcheck/logické smoke testy,
- integrační kontrakty,
- provozní dokumentace,
- fallback pravidla. [M2][M3][M4]

---

## 7. Co se v monorepu nemá dělat

- nerozdělovat core workers do samostatných repozitářů,
- nedržet infra definice bokem „jen u ops týmu“,
- neskrývat contract schémata uvnitř jedné aplikace,
- netvořit zvláštní experimentální repo pro alternativní auth/search/queue stack,
- nepoužívat lokální prototypy s jinými systémovými hranicemi než ve finální architektuře.

Tohle přímo navazuje na hlavní zásadu: neodkládají se rozhodnutí o hranicích systému. [M1]

---

## 8. Minimální startovní layout

Pokud je potřeba začít rychle, minimální doporučená kostra je tato:

```text
/
├── apps/
│   ├── api/
│   └── web/
├── workers/
│   ├── ingest-worker/
│   ├── embed-worker/
│   └── vlm-worker/
├── packages/
│   ├── domain-model/
│   ├── inference-contract/
│   ├── audit-consent-model/
│   ├── source-routing/
│   └── prefect-flows/
├── infra/
│   ├── local-compose/
│   ├── k8s/
│   └── apptainer/
├── docs/
└── tests/
```

Tato kostra je dost malá na rychlý start a současně už respektuje finální hranice systému. [M1][M2][M3][M4][M5][M6]

---

## 9. Reference uvnitř návrhu

- **[M1]** `finalizovany_navrh_metavo_v5.md`, sekce **1. Cíl a závazné zásady**
- **[M2]** `finalizovany_navrh_metavo_v5.md`, sekce **3.1 Provozní model** a **3.2 Packaging**
- **[M3]** `finalizovany_navrh_metavo_v5.md`, sekce **3.3 Hranice systému a finální vítězové** a **3.4 Search a retrieval hranice**
- **[M4]** `finalizovany_navrh_metavo_v5.md`, sekce **3.5 Inference contract a guwp**
- **[M5]** `finalizovany_navrh_metavo_v5.md`, sekce **3.7 Consent, audit a evidence**
- **[M6]** `finalizovany_navrh_metavo_v5.md`, sekce **3.6 Multimodální ingest: řízený routing, ne slepý fallback**
- **[M7]** `finalizovany_navrh_metavo_v5.md`, sekce **3.3 Hranice systému a finální vítězové** a reference **[R6][R8][R9]** pro Prefect
- **[M8]** `finalizovany_navrh_metavo_v5.md`, sekce **3.8 Škálovací model: 1M requestů != 1M synchronních LLM generací**
- **[M9]** `finalizovany_navrh_metavo_v5.md`, sekce **3.9 Observability, compliance a supply chain**
