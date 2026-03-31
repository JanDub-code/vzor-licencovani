# Pořadí práce a závislosti

Tento soubor použij, když chceš pustit více agentů a nechceš, aby si lezli do stejné hranice.

## Doporučené pořadí

### Fáze 1 — uzamčení kostry
1. `01_GLOBAL_GUARDRAILS.md`
2. `02_AGENT_ARCHITECTURE_BOUNDARIES_AND_MONOREPO.md`
3. `10_AGENT_INFRA_PACKAGING_AND_DEPLOYMENT.md`

### Fáze 2 — kontrakty a runtime hranice
4. `03_AGENT_ONLINE_BATCH_AND_SCHEDULER.md`
5. `05_AGENT_QUEUE_WORKFLOWS_AND_WORKERS.md`
6. `06_AGENT_SEARCH_RETRIEVAL_AND_STORAGE.md`
7. `09_AGENT_INFERENCE_AND_SCALING.md`
8. `08_AGENT_CONSENT_AUDIT_AND_EVIDENCE.md`

### Fáze 3 — vstupní aplikace a ingest
9. `04_AGENT_API_UI_AND_AUTH.md`
10. `07_AGENT_INGEST_ROUTING_AND_MULTIMODAL.md`

### Fáze 4 — kontrolní a provozní vrstva
11. `11_AGENT_OBSERVABILITY_COMPLIANCE_AND_SUPPLY_CHAIN.md`
12. `12_AGENT_TESTS_AND_QUALITY_GATES.md`

## Co může běžet paralelně

### Bezpečně paralelně
- `03` + `06`
- `05` + `08`
- `09` + `11`
- `04` až po základním dokončení `02`, `06`, `08`, `09`
- `12` může začít dřív, ale plnou matrici dokončí až po `03`, `08`, `09`

### Nespouštět paralelně bez koordinace
- `02` a kdokoliv, kdo přesouvá cesty v repo
- `05` a `07`, pokud oba sahají do stejných workerů
- `04` a `08`, pokud oba mění consent enforcement body
- `09` a `04`, pokud oba upravují request path do inference

## Minimální kombinace pro konkrétní cíle

### Jen rozřezání monorepa
- `01`
- `02`

### Jen backend orchestrace
- `01`
- `03`
- `05`

### Jen search/RAG kostra
- `01`
- `06`
- `09`

### Jen compliance a audit
- `01`
- `08`
- `11`

### Jen infra bootstrap
- `01`
- `10`
- `12`

## Povinný handoff formát mezi agenty

Každý agent musí na konci vrátit:

### Změněné cesty
- seznam složek a souborů

### Otevřené závislosti
- co blokuje dalšího agenta
- co potřebuje potvrdit

### Nezměněné hranice
- co zůstalo záměrně beze změny

### Doporučený další agent
- číslo a důvod
