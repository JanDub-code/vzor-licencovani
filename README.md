# Balík instrukcí pro rozdělení práce mezi agenty

Tento ZIP je připravený tak, aby šlo označit jen konkrétní `.md` soubory a agent udělal jen vybranou část práce.

## Jak to používat

1. **Vždy přilož `agents/01_GLOBAL_GUARDRAILS.md`.**
2. K němu přilož jen ty agentní soubory, které odpovídají části projektu, kterou chceš řešit.
3. Pokud chceš, aby agent měl po ruce originální návrh, přilož i soubory ze složky `source/`.
4. Když řešíš víc oblastí najednou, kombinuj více agentních `.md`, ale drž se závislostí v `agents/13_EXECUTION_ORDER_AND_DEPENDENCIES.md`.

## Doporučené rychlé kombinace

### Bootstrap monorepa
- `agents/01_GLOBAL_GUARDRAILS.md`
- `agents/02_AGENT_ARCHITECTURE_BOUNDARIES_AND_MONOREPO.md`
- `agents/10_AGENT_INFRA_PACKAGING_AND_DEPLOYMENT.md`
- `agents/12_AGENT_TESTS_AND_QUALITY_GATES.md`

### API + UI + auth
- `agents/01_GLOBAL_GUARDRAILS.md`
- `agents/04_AGENT_API_UI_AND_AUTH.md`
- `agents/08_AGENT_CONSENT_AUDIT_AND_EVIDENCE.md`

### Queue + workflow + batch
- `agents/01_GLOBAL_GUARDRAILS.md`
- `agents/03_AGENT_ONLINE_BATCH_AND_SCHEDULER.md`
- `agents/05_AGENT_QUEUE_WORKFLOWS_AND_WORKERS.md`

### Search + RAG + storage
- `agents/01_GLOBAL_GUARDRAILS.md`
- `agents/06_AGENT_SEARCH_RETRIEVAL_AND_STORAGE.md`
- `agents/09_AGENT_INFERENCE_AND_SCALING.md`

### Compliance + observability
- `agents/01_GLOBAL_GUARDRAILS.md`
- `agents/08_AGENT_CONSENT_AUDIT_AND_EVIDENCE.md`
- `agents/11_AGENT_OBSERVABILITY_COMPLIANCE_AND_SUPPLY_CHAIN.md`

## Obsah balíku

- `agents/00_INDEX.md` — rychlá mapa všech agentů
- `agents/01_GLOBAL_GUARDRAILS.md` — povinné globální zásady
- `agents/02_AGENT_ARCHITECTURE_BOUNDARIES_AND_MONOREPO.md`
- `agents/03_AGENT_ONLINE_BATCH_AND_SCHEDULER.md`
- `agents/04_AGENT_API_UI_AND_AUTH.md`
- `agents/05_AGENT_QUEUE_WORKFLOWS_AND_WORKERS.md`
- `agents/06_AGENT_SEARCH_RETRIEVAL_AND_STORAGE.md`
- `agents/07_AGENT_INGEST_ROUTING_AND_MULTIMODAL.md`
- `agents/08_AGENT_CONSENT_AUDIT_AND_EVIDENCE.md`
- `agents/09_AGENT_INFERENCE_AND_SCALING.md`
- `agents/10_AGENT_INFRA_PACKAGING_AND_DEPLOYMENT.md`
- `agents/11_AGENT_OBSERVABILITY_COMPLIANCE_AND_SUPPLY_CHAIN.md`
- `agents/12_AGENT_TESTS_AND_QUALITY_GATES.md`
- `agents/13_EXECUTION_ORDER_AND_DEPENDENCIES.md`
- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`

## Poznámka k odkazům na původní soubory

Všechny agentní instrukce odkazují na:
- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`

u každého odkazu je uvedená i sekce a rozsah řádků, aby šlo rychle dohledat původní kontext.
