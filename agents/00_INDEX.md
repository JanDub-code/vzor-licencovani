# Index agentů

## Povinný soubor
- `01_GLOBAL_GUARDRAILS.md` — přikládat vždy

## Agenti podle oblasti

| Soubor | Oblast | Kam typicky sahá |
|---|---|---|
| `02_AGENT_ARCHITECTURE_BOUNDARIES_AND_MONOREPO.md` | architektonické hranice a repo layout | `apps/`, `services/`, `workers/`, `packages/`, `infra/`, `docs/`, `tests/` |
| `03_AGENT_ONLINE_BATCH_AND_SCHEDULER.md` | oddělení online/batch a OpenPBS adapter | `services/scheduler-adapter/`, `infra/apptainer/`, `infra/k8s/`, `workers/` |
| `04_AGENT_API_UI_AND_AUTH.md` | FastAPI, web UI, OIDC | `apps/api/`, `apps/web/`, `packages/auth/`, `infra/keycloak/` |
| `05_AGENT_QUEUE_WORKFLOWS_AND_WORKERS.md` | RabbitMQ, Prefect, workflow a workers | `packages/prefect-flows/`, `packages/queue-jobs/`, `packages/rabbitmq-client/`, `workers/`, `infra/rabbitmq/` |
| `06_AGENT_SEARCH_RETRIEVAL_AND_STORAGE.md` | PostgreSQL vs object store vs Qdrant vs OpenSearch | `packages/search-contract/`, `packages/object-store/`, `packages/qdrant-client/`, `packages/opensearch-client/`, `infra/*` |
| `07_AGENT_INGEST_ROUTING_AND_MULTIMODAL.md` | multimodální ingest a routing | `workers/ingest-worker/`, `workers/render-worker/`, `workers/screenshot-worker/`, `workers/extract-worker/`, `packages/source-routing/` |
| `08_AGENT_CONSENT_AUDIT_AND_EVIDENCE.md` | consent gate, audit, incidenty, evidence | `packages/audit-consent-model/`, `apps/api/`, `apps/web/`, `packages/domain-model/` |
| `09_AGENT_INFERENCE_AND_SCALING.md` | inference contract, vLLM, llama.cpp, cache, service classes | `packages/inference-contract/`, `apps/api/`, `workers/`, `packages/config/`, `infra/*` |
| `10_AGENT_INFRA_PACKAGING_AND_DEPLOYMENT.md` | Docker mirror, K8s, Apptainer, lokální topologie | `infra/local-compose/`, `infra/k8s/`, `infra/apptainer/`, `infra/postgres/`, `infra/redis/`, `infra/qdrant/`, `infra/opensearch/`, `infra/monitoring/` |
| `11_AGENT_OBSERVABILITY_COMPLIANCE_AND_SUPPLY_CHAIN.md` | metriky, logy, tracing, model intake, compliance gate | `packages/observability/`, `infra/monitoring/`, `infra/ci/`, `docs/` |
| `12_AGENT_TESTS_AND_QUALITY_GATES.md` | unit/integration/e2e/contract/load testy | `tests/`, `apps/`, `packages/`, `services/`, `workers/`, `infra/ci/` |
| `13_EXECUTION_ORDER_AND_DEPENDENCIES.md` | pořadí práce a návaznosti | plánování napříč repo |

## Doporučené pravidlo
Když si nejsi jistý, vyber raději méně souborů a drž práci v jedné uzamčené hranici.
