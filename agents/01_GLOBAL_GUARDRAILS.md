# Globální guardrails pro všechny agenty

Tento soubor přikládej vždy. Každý agent musí respektovat tyto zásady bez výjimek.

## Hlavní princip

Projekt se nesmí rozsekat podle pohodlí implementace. Musí se držet **už uzamčených systémových hranic**.

## Tvrdá pravidla

- **Nezaváděj mezikroky**, které mění cílovou architekturu.
- **Neměň finální komponenty za „dočasné MVP“ varianty.**
- **Lokální Docker prostředí je jen zmenšený mirror finální architektury.**
- **Monorepo je jediný zdroj pravdy** pro kontrakty, doménový model, workflow, infrastrukturu, testy a dokumentaci.
- **Online vrstva a batch vrstva musí zůstat oddělené.**
- **Kontrakty a hranice mají přednost před rychlostí implementace.**

## Výslovně zakázané zkratky

- PostgreSQL jako dočasná queue
- PostgreSQL fulltext jako dočasná náhrada OpenSearch
- lokální JWT auth jako náhrada OIDC
- Compose-only produkční model
- Slurm hardlock
- „zatím bez orchestrace“

## Povinné finální komponenty

- PostgreSQL
- S3-kompatibilní object store
- RabbitMQ
- Prefect
- Qdrant
- OpenSearch
- OIDC
- Redis
- vLLM pro online inference
- llama.cpp + GGUF pro batch inference
- Prometheus + Grafana
- Loki jen pokud projde compliance gate

## Povinný způsob práce agenta

1. Nejprve si potvrď, že úkol spadá do tvé hranice.
2. Nezasahuj do sousední hranice, pokud to není nutné pro kontrakt nebo handoff.
3. Když narazíš na závislost, napiš přesně co chybí a komu to patří.
4. Každý výstup zakonči:
   - seznamem změněných cest,
   - seznamem otevřených rizik,
   - handoffem pro dalšího agenta,
   - kontrolou souladu s guardrails.

## Povinné zdrojové soubory

- `source/finalizovany_navrh_metavo_v6_bez_guwp.md`
  - `## 1. Cíl a závazné zásady` (ř. 3–46)
  - `## 3.1 Provozní model` (ř. 65–79)
  - `## 3.2 Packaging` (ř. 80–108)
  - `## 3.3 Hranice systému a finální vítězové` (ř. 109–134)
  - `## 4.1 Musí být uzamčeno hned` (ř. 313–328)
  - `## 6. Monorepo governance` (ř. 402–437)
  - `## 7. Konečný technologický výběr` (ř. 438–479)

- `source/monorepo_rozdeleni_metavo_v2_zjednodusene.md`
  - `## 1. Závazná zásada` (ř. 11–25)
  - `## 2. Co má struktura monorepa sledovat` (ř. 26–40)
  - `## 3. Doporučená top-level struktura` (ř. 41–55)
  - `## 5. Doporučená ownership mapa` (ř. 271–288)
  - `## 7. Co se v monorepu nemá dělat` (ř. 311–322)

## Definice hotovo

Úkol je hotový jen pokud:
- neporušuje uzamčené hranice,
- je jasné kam patří kód i dokumentace v monorepu,
- nejsou zavedené dočasné technologie navíc,
- jsou vypsané závislosti a další kroky.
