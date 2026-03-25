# Finalizovaný návrh architektury pro MetaVO / MetaCentrum

## 1. Cíl a závazné zásady

Cílem je vybudovat právně obhajitelný, robustní a škálovatelný multimodální systém pro sběr webových dat, screenshot evidence a AI analýzu (LLM + RAG), který:

- odděluje **online vrstvu** od **batch vrstvy**, [Z5][H1]
- je nasaditelný v prostředí **MetaVO / MetaCentrum**, [R1][R2]
- zachovává **audit, evidenci, citace a dohledatelnost**, jak požaduje zdrojové zadání, [Z1][Z3][Z6]
- od začátku používá **stejné architektonické hranice a stejné cílové komponenty**, které mají zůstat i ve finální verzi.

### 1.1 Závazná zásada bez mezikroků

> **Neodkládají se rozhodnutí o hranicích systému. Odkládat lze až implementační hloubku uvnitř už uzamčených hranic.**

Z toho plyne následující pravidlo:

- pokud je nějaká komponenta vybrána jako finální, **nebude se v první fázi nahrazovat jinou komponentou jen proto, aby byl rychlejší start**, 
- jedinou výjimkou je **lokální Docker setup**, který musí existovat a fungovat jako vývojový a integrační mirror finální architektury,
- lokální setup ale musí používat **stejné kontrakty, stejné hranice a pokud možno stejné komponenty**, jen v menším měřítku.

Praktický důsledek:

- **RabbitMQ** se bere od začátku, ne až „později místo PostgreSQL queue“, [Z3][R3][R4][R5]
- **OpenSearch** se bere od začátku, ne až „později místo PostgreSQL fulltextu“, [Z3][R10][R11][R13]
- **OIDC/OAuth hranice auth modelu** se bere od začátku; pro aktuální testing je přípustné použít free OAuth/OIDC provider (např. Auth0 free tier) a plnou produkční OIDC konfiguraci dokončit až ve finální verzi, pokud se nemění kontrakt tokenů, claims a autorizace, [Z4]
- **Prefect** se bere od začátku jako workflow vrstva, i když první flow budou menší a jednodušší. [Z3][R6][R8][R9]

### 1.2 Monorepo je povinné

Celý systém se vede jako **monorepo**, i když některé runtime komponenty budou fyzicky nasazeny mimo sebe nebo mimo hlavní cluster. Monorepo je **jediný zdroj pravdy** pro:

- kontrakty,
- doménový model,
- infrastrukturu,
- workflow,
- politiky,
- testy,
- dokumentaci.

Samostatný dokument pro rozdělení monorepa je zde:

- **`monorepo_rozdeleni_metavo_v1.md`** — rozpad repozitáře, ownership a vazba na hlavní architekturu.

---

## 2. Co zůstává ze zdrojového zadání

Původní zadání už správně definuje základní stavebnici, která se zachovává:

- multimodální ingest s prioritním pořadím **API → HTML → Rendered DOM → Screenshot → Upstream AI provider**, [Z2]
- **vector DB**, volitelný fulltext, inference, API + UI, monitoring/logy, [Z3]
- **OIDC jako preferovanou autentizaci**, s lokální autentizací pouze jako alternativní MVP možností, [Z4]
- **Docker Compose pro MVP** a **Kubernetes pro produkci**, [Z4][Z5]
- **MetaCentrum pro dávkové výpočty**, zatímco online dotazy běží mimo batch vrstvu, [Z5]
- **CAPTCHA evidence, incident lifecycle, audit trail a doplnění dat po incidentu**, [Z3][Z6]
- **A/B experimenty**, **RAG vs no-RAG**, **verzování dokumentů/indexů** a **reprodukovatelnost**. [Z1][Z4]

Finální návrh tedy nepřepisuje zadání od nuly. Uzamyká konkrétní technická rozhodnutí uvnitř rámce, který už zadání definuje. [Z1][Z3][Z4][Z5]

---

## 3. Finální cílová architektura

## 3.1 Provozní model: online odděleně, batch v MetaVO / MetaCentru

**Rozhodnutí**

- **Online vrstva** běží jako dlouhožijící služby mimo batch scheduler. [Z5][H1]
- **Batch vrstva** běží v MetaVO / MetaCentru přes scheduler adapter. [Z5][R1][R2]
- Adapter je **OpenPBS-first** a architektura není hardcoded na Slurm. [R1][R2]
- Na batch packaging v MetaCentru se počítá s **Singularity/Apptainer**, ne s Dockerem spouštěným na compute nodech. [H2]

**Důsledek**

- login/frontend nody nejsou runtime pro dlouhé výpočty ani online služby, [H2]
- online API, auth, search, queue, observability a online inference neběží jako batch job,
- MetaVO / MetaCentrum slouží pro embeddingy, screenshot/VLM extrakci, reindexy, experimenty a další dávkové workloady. [Z5][H1]

## 3.2 Packaging: lokální Docker mirror, produkčně K8s + OpenPBS

**Rozhodnutí**

- **Lokální a integrační prostředí:** Docker Compose
- **Produkční online vrstva:** Kubernetes / Cloud
- **Produkční batch vrstva:** OpenPBS + Apptainer/Singularity

Toto není změna architektury podle prostředí. Je to pouze rozdíl v balení a orchestrace runtime. Samotné systémové hranice zůstávají stejné. [Z4][Z5][H1][H2]

### 3.2.1 Co smí být v lokálním Docker setupu odlišné

Jen tyto dvě věci:

- menší sizing,
- lokální instance služeb místo produkčně spravovaných služeb.

### 3.2.2 Co se nesmí lišit ani lokálně

- kontrakty mezi vrstvami,
- výběr hlavních komponent,
- datový model,
- auth model,
- queue model,
- search model,
- evidence/audit/consent model.

Lokální prostředí tedy není „rychlé provizorium“. Je to **zmenšená věrná kopie finální architektury**.

## 3.3 Hranice systému a finální vítězové

| Vrstva | Finální rozhodnutí | Poznámka |
|---|---|---|
| Metadata / audit / consent / stav | **PostgreSQL** | primární transakční a konfigurační vrstva [Z3] |
| Evidence store | **S3-kompatibilní object store** | lokálně MinIO; produkčně MinIO jen po compliance schválení, jinak Ceph S3 / MetaCloud object storage [Z3][Z5][H7] |
| Task queue | **RabbitMQ** | ack/nack, DLX, priority, oddělené workflow fronty [R3][R4][R5] |
| Workflow orchestrace | **Prefect** | Python-first flows, retry, state tracking [Z3][R6][R8][R9] |
| Vector retrieval | **Qdrant** | specializovaná vector vrstva [Z3] |
| Fulltext / hybrid retrieval | **OpenSearch** | lexical + hybrid + relevance tuning [Z3][R10][R11][R13] |
| Online inference | **vLLM** | high-throughput serving [R14][R15][R16][R17] |
| Batch inference | **llama.cpp + GGUF** | vhodné pro kvantizované batch workloady v MetaCentru [R18][R19][R20][R21] |
| Auth | **OIDC** | lokálně vlastní IdP v Dockeru, produkčně institucionální IdP [Z4] |
| Monitoring / metriky | **Prometheus + Grafana** | stejné od začátku [Z3][Z5] |
| Log backend | **Loki** | jen po compliance schválení; jinak alternativní backend se stejným observability kontraktem [Z3][H7] |
| L2 cache / rate-limit store | **Redis** | L1 cache zůstává in-process v API [H4][H5] |

### 3.3.1 Co se tímto výslovně odmítá

- **žádná PostgreSQL queue jako mezikrok**, 
- **žádný PostgreSQL fulltext jako mezikrok**, 
- **žádný lokální JWT auth jako mezikrok**, 
- **žádný Compose-only produkční model**, 
- **žádný Slurm hardlock**, 
- **žádné „zatím bez orchestrace, pak se uvidí“**.

## 3.4 Search a retrieval hranice

Zde se architektura uzamyká hned, protože pozdější přepis search vrstvy bývá drahý a často se už nikdy neudělá.

### 3.4.1 PostgreSQL

Patří sem pouze:

- source registry,
- crawl rules,
- retention rules,
- audit log,
- consent records,
- incident records,
- workflow metadata,
- verze dokumentů a indexů,
- provozní konfigurace. [Z1][Z3][Z4][Z6]

### 3.4.2 Object store

Patří sem:

- screenshoty,
- HTML/DOM snapshoty,
- PDF důkazy,
- raw exporty,
- hashované důkazní artefakty navázané na citace a incidenty. [Z2][Z3][Z6]

### 3.4.3 Qdrant

Patří sem:

- embeddingy chunků,
- metadata filtry potřebné pro RAG retrieval,
- retrieval kandidáti pro generativní odpověď. [Z3]

### 3.4.4 OpenSearch

Patří sem:

- lexical search,
- hybrid retrieval,
- relevance tuning,
- dotazové scénáře, kde je BM25/hybrid search důležitý,
- případný reranking pipeline navázaný na search vrstvu. [Z3][R10][R11][R13]

## 3.5 Inference contract a `guwp`

### 3.5.1 Uzamčený inference contract

Architektura uzamyká **inference contract**, nikoli konkrétní interní wrapper. Contract musí sjednotit:

- vstupní payload,
- výstupní payload,
- metadata,
- timeouty,
- retry/fallback pravidla,
- tracing a auditovatelnou identifikaci volání,
- mapování na online a batch inference backendy.

Backendy se mohou lišit podle workloadu, ale kontrakt se nemění.

### 3.5.2 Přesná definice `guwp`

`guwp` se nepovažuje za veřejný standard ani za povinnou infrastrukturní komponentu. Pro účely architektury je definován takto:

> **`guwp` = interní inference gateway vrstva sjednocující přístup k online inference backendům a externím modelovým API přes jednotný OpenAI-like kontrakt, s fallback politikou, auditovatelným routováním a observability.** [H8]

### 3.5.3 Kde `guwp` sedí v request path

Volitelná cesta je:

`API / orchestrátor -> inference contract -> guwp -> konkrétní backend`

`guwp` nevlastní:

- batch scheduler,
- storage,
- retrieval,
- consent model,
- evidence model.

Je to pouze tenká integrační vrstva.

### 3.5.4 Co se stane, když `guwp` nebude

Architektura na `guwp` **nesmí být existenčně závislá**. Pokud nebude použit nebo nebude připraven do produkce, systém musí fungovat takto:

- online inference jde přímo z inference contractu na `vLLM` nebo externí API,
- batch inference jde přes batch adapter a batch runtime,
- aplikační vrstva se nepřepisuje, protože kontrakt zůstává stejný. [H8]

## 3.6 Multimodální ingest: řízený routing, ne slepý fallback

Původní zadání správně definuje multimodální ingest a fallback pořadí. [Z2][Z3] Finální návrh ponechává stejné strategie, ale uzamyká jejich provozní pravidla:

1. **API / feed**
2. **HTML fetch + parse**
3. **Rendered DOM**
4. **Screenshot screening**
5. **Upstream AI provider** [Z2][Z3]

### 3.6.1 Závazná pravidla

- **source-profile routing**: per-source preferovaná strategie,
- **bounded fallback**: maximálně omezený počet fallback kroků,
- **quality gate**: další krok jen při nízkém quality score,
- **incident stop**: CAPTCHA / blokace / licenční problém workflow zastaví a založí incident,
- **retry po incidentu**: workflow pokračuje až po vyřešení. [Z2][Z3][Z6]

Tím se zachovává jádro původního návrhu, ale eliminuje se slepé sekvenční timeoutování.

## 3.7 Consent, audit a evidence

Původní dokument už obsahuje právní titul, audit, role Curator/Admin a důkazní artefakty. [Z1][Z4][Z6] Finální návrh doplňuje **hard Consent Gate** jako povinnou technickou kontrolu. [H3]

### 3.7.1 Povinný enforcement model

1. **UI hard gate** — bez explicitního odsouhlasení nelze spustit scraping, screening ani citlivou analýzu. [H3]
2. **API enforcement** — request musí nést buď `consent_token`, nebo odkaz na platný `ConsentRecord`. [H3]
3. **Append-only audit** — ukládá se verze textu, scope souhlasu, čas, identita, IP a zdroj/dataset. [H3]

### 3.7.2 Povinné checkboxy

- „Potvrzuji, že mám oprávnění ke sběru a zpracování obsahu z tohoto zdroje.“
- „Beru na vědomí, že odpovídám za dodržení licenčních podmínek.“

### 3.7.3 Doporučená tabulka

```sql
CREATE TABLE consent_record (
    consent_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id          UUID NOT NULL REFERENCES app_user(user_id),
    source_id        UUID NOT NULL REFERENCES source(source_id),
    consent_text     TEXT NOT NULL,
    consent_version  VARCHAR(20) NOT NULL,
    accepted_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    ip_address       INET,
    user_agent       TEXT
);
```

## 3.8 Škálovací model: 1M requestů != 1M synchronních LLM generací

Finální návrh explicitně rozlišuje dvě škálovací roviny:

- **front-door throughput** = auth, rate limiting, cache lookup, enqueue, retrieval bez generace, [H4]
- **inference throughput** = reálná generativní kapacita modelů omezená tok/s, KV cache a pamětí. [H4]

### 3.8.1 Povinné vrstvy pro škálování

- **L1 cache** v API procesu, [H5]
- **L2 distribuovaná cache a rate-limit store** přes Redis, [H5]
- **service classes**: `FAST_PATH`, `STANDARD`, `SLOW_PATH`, [H4]
- **asynchronní režim** pro delší odpovědi a batch-like úlohy, [H4]
- **kvóty a admission control** na gateway/API vrstvě. [H4]

Z toho plyne, že architektura se neškáluje stylem „1M synchronních generací“, ale tak, aby většina requestů skončila v cache, retrieval-only režimu nebo lehčí inference třídě. [H4][H5]

## 3.9 Observability, compliance a supply chain

### 3.9.1 Observability je povinná od začátku

Od začátku musí existovat:

- metriky,
- strukturované logy,
- tracing,
- korelace mezi requestem, jobem, inference a auditem. [Z3][Z5]

Nejde o to, zda observability bude. Jde jen o hloubku dashboardů a alertingu.

### 3.9.2 Supply-chain policy pro modely a kontejnery

Do finální architektury se doplňuje **Model Intake Policy**:

- modely se v produkci nestahují ad hoc z veřejných repozitářů, [H6]
- používá se allowlist / mirror, [H6]
- kontrolují se checksumy, [H6]
- build/release pipeline generuje SBOM a vulnerability scan, [H6]
- preferují se bezpečnější artefakty typu `safetensors`, kde je to relevantní, [H6]
- modely a runtime image se verzují a podepisují. [H6]

### 3.9.3 Compliance gate pro MinIO a Loki

- **MinIO**: lokálně povoleno; do produkce jen po právním/compliance schválení, jinak Ceph S3 / MetaCloud object storage. [H7]
- **Loki**: do produkce jen po právním/compliance schválení, jinak alternativní log backend. [H7]

Toto není změna systémových hranic. Je to governance rozhodnutí uvnitř už uzamčené object-store a logging boundary.

---

## 4. Co musí být uzamčeno hned, co navrženo hned a co lze prohlubovat později

> **Neodkládají se rozhodnutí o hranicích systému. Odkládat lze až implementační hloubku uvnitř už uzamčených hranic.**

## 4.1 Musí být uzamčeno hned

To jsou rozhodnutí, která by později způsobila přepis architektury:

1. **provozní model online vs batch**, [Z5][H1]
2. **scheduler abstraction a OpenPBS-first batch model**, [R1][R2]
3. **hranice mezi PostgreSQL / object store / OpenSearch / Qdrant**, [Z3]
4. **RabbitMQ jako task queue**, [R3][R4][R5]
5. **Prefect jako workflow vrstva**, [R6][R8][R9]
6. **OIDC jako auth model**, [Z4]
7. **inference contract**, [H8]
8. **source routing a bounded fallback pravidla**, [Z2][Z3]
9. **evidence / audit / consent model**, [Z1][Z4][H3]
10. **L1/L2 cache + service classes + async slow-path**, [H4][H5]
11. **monorepo boundary a ownership model**.

## 4.2 Musí být navrženo hned, ale lze prohlubovat později

Toto se nepřepíná na jiné technologie; jen se prohlubuje implementace:

1. **observability model** — základ od začátku, později hlubší dashboardy a alerting, [Z3][Z5]
2. **supply-chain policy** — základ od začátku, později automatizace a signing, [H6]
3. **compliance gate pro object store a log backend** — rozhodovací workflow od začátku, [H7]
4. **experiment / evaluace** — základní schopnost od začátku, později širší framework, [Z1][Z4]
5. **HA tuning** — architektonicky připravené, ale ne nutně plně rozvinuté v první produkční iteraci.

## 4.3 Může se doplnit až po funkčním jádru

Sem patří jen věci, které nepřepisují hranice systému:

- pokročilé dashboardy,
- rozšířený experiment framework,
- rozšířené alerting scénáře,
- pokročilé failover a multi-zone nastavení,
- no-RAG jako samostatný produktový režim mimo hlavní request path. [Z1][Z4]

---

## 5. Implementační pořadí bez technologických přepínačů

## 5.1 Fáze A — lokální Docker mirror

Lokálně musí běžet **stejná architektura v malém**, ne alternativní MVP stack.

Lokální povinné komponenty:

- FastAPI,
- Web UI,
- PostgreSQL,
- RabbitMQ,
- Prefect,
- Redis,
- Qdrant,
- OpenSearch,
- S3-kompatibilní object store (lokálně MinIO),
- Prometheus,
- Grafana,
- Loki,
- lokální OIDC provider,
- online inference backend,
- batch adapter stub / lokální adapter pro integrační testy.

Tato fáze slouží k tomu, aby šlo průběžně ověřovat, že se systém pohybuje správným směrem bez pozdějších architektonických výměn.

## 5.2 Fáze B — integrační / předprodukční prostředí

Cíl:

- stejný stack jako lokálně,
- reálné integrační testy,
- první připojení na MetaVO batch cestu,
- reálné end-to-end workflow přes RabbitMQ + Prefect + OpenPBS adapter,
- reálné consent enforcement a audit trail.

## 5.3 Fáze C — produkční online vrstva + produkční batch vrstva

Cíl:

- online služby v Kubernetes / Cloud,
- batch workloady v OpenPBS,
- institucionální OIDC,
- compliance schválený object store a log backend,
- kapacitní tuning front-door vs inference throughput,
- hardening tajemství a release pipeline.

Ve všech třech fázích zůstávají stejné hranice i stejné finální komponenty; mění se jen balení, sizing, governance a provozní tvrdost.

---

## 6. Monorepo governance

Monorepo je závazná součást architektury, nikoli jen organizační preference.

### 6.1 Co musí být uvnitř monorepa

- API a UI,
- workflow definice,
- scheduler adapter,
- inference contract a případný `guwp`,
- workers,
- shared domain model,
- policy knihovny,
- integrační klienti,
- infrastruktura pro local/K8s/Apptainer,
- testy,
- dokumentace.

### 6.2 Co může runtime žít mimo monorepo

Mimo repozitář může fyzicky běžet:

- institucionální IdP,
- spravovaný object store,
- spravovaný OpenSearch,
- MetaVO batch runtime,
- externí modelová API.

Ale **kontrakty, adaptéry, provisioning, konfigurace a testovací artefakty pro tyto vazby musí zůstat v monorepu**.

Podrobný rozpad repozitáře je v samostatném dokumentu:

- **`monorepo_rozdeleni_metavo_v1.md`**

---

## 7. Konečný technologický výběr

### 7.1 Online vrstva

- **API / backend:** FastAPI [Z3][Z5]
- **Web UI:** admin + curator + analyst + user [Z3][Z4]
- **Auth model:** OIDC [Z4]
- **Metadata DB:** PostgreSQL [Z3]
- **Task queue:** RabbitMQ [R3][R4][R5]
- **Workflow orchestrace:** Prefect [R6][R8][R9]
- **L2 cache / rate-limit store:** Redis [H4][H5]
- **Vector DB:** Qdrant [Z3]
- **Fulltext / hybrid retrieval:** OpenSearch [R10][R11][R13]
- **Evidence store boundary:** S3-kompatibilní object store [Z3][Z5][H7]
- **Online inference:** vLLM [R14][R15][R16][R17]
- **Monitoring:** Prometheus + Grafana [Z3][Z5]
- **Log backend:** Loki, pouze po compliance schválení [Z3][H7]

### 7.2 Batch vrstva v MetaVO / MetaCentru

- **Scheduler adapter:** OpenPBS-first [R1][R2]
- **Batch packaging:** Apptainer/Singularity [H2]
- **Workers:** ingest / extract / embed / VLM / reindex [Z3][Z5]
- **Batch inference:** llama.cpp + GGUF [R18][R19][R20][R21]
- **Browser automation:** Playwright [Z3]

### 7.3 Startovní modely

Tato část ve zdrojovém PDF chybí a doplňuje se jako praktický baseline:

- **Embedding:** `BAAI/bge-m3`
- **VLM / screenshot extraction:** `Qwen2.5-VL-7B`
- **Online RAG LLM:** `Llama 3.1 8B Instruct` nebo `Qwen2.5 14B`
- **OCR fallback:** `Tesseract` pouze jako nouzová CPU cesta

Finální modelová volba musí projít benchmarkem na češtině, reálných datech, retrieval kvalitě, end-to-end latenci a nákladech batch vs online inference. [Z1][Z4][Z5]

---

## 8. Finální závěr

Tento návrh zachovává architektonické jádro původního zadání:

- multimodální ingest,
- evidence a audit,
- RAG s citacemi,
- oddělení online a batch vrstvy,
- Compose → Kubernetes trajektorii,
- OIDC preferenci,
- experimenty a reprodukovatelnost. [Z1][Z2][Z3][Z4][Z5][Z6]

Současně ale odstraňuje dva časté zdroje budoucího přepisování:

1. **neurčité mezivrstvy bez přesné role** — proto je `guwp` definován jako volitelná tenká gateway nad uzamčeným inference contractem, [H8]
2. **MVP mezikroky s jinými technologiemi než ve finále** — proto se od začátku používají finální systémové hranice a finální komponenty, s jedinou výjimkou lokálního Docker mirroru, který je pouze zmenšenou kopií stejné architektury.

Tento dokument tedy neříká „co by šlo někdy udělat“. Říká, **co se má uzamknout hned, aby se to nemuselo později překopávat**.

---

## 9. Reference

### Zdroje ze zdrojového zadání

- **[Z1]** _Návrh architektury a nasazení: Multimodální systém pro sběr webových dat, screenshot screening a AI analýzu (LLM + RAG)_, verze 1.0, 13.02.2026, str. 2–4. Soubor: `Navrh_architektury_AI_RAG_screenshot_screening_FINAL - zadání od zákazníka.pdf`
- **[Z2]** Tamtéž, str. 3: správa zdrojů, multimodální ingest, screenshot screening, indexace a vyhledávání.
- **[Z3]** Tamtéž, str. 5–7: přehled komponent, queue kandidáti, orchestrátor, vector DB, fulltext, inference, monitoring/logy, CAPTCHA evidence.
- **[Z4]** Tamtéž, str. 4–5: nefunkční požadavky, OIDC preference, Compose/Kubernetes přenositelnost, audit, Vault/Secrets.
- **[Z5]** Tamtéž, str. 7–8: deployment, Docker Compose MVP, Kubernetes produkce, MetaCentrum batch, monitoring a zálohy.
- **[Z6]** Tamtéž, str. 6–9: CAPTCHA lifecycle, incident queue, retry, akceptační kritéria, rizika a mitigace.

### Doplňující reference z hlubokého výzkumu

- **[H1]** _zprava-hluboky-vyzkum.md_, části **Exekutivní shrnutí** a **Orchestrace a výpočetní substrate**: dvouvrstvá platforma, MetaCentrum/MetaVO jako Kubernetes + Cloud + Grid, online služby jako dlouhožijící runtime mimo čistý batch.
- **[H2]** _zprava-hluboky-vyzkum.md_, část o **praktických provozních omezeních MetaCentra** a tabulka **MetaCentrum Grid + Singularity**: login/frontend nody nejsou na dlouhé výpočty, Docker na compute nodech není správná cesta, doporučené je Singularity/Apptainer nebo Kubernetes.
- **[H3]** _zprava-hluboky-vyzkum.md_, část **Povinný krok: checkbox + auditovatelný souhlas (Consent Gate)** a navazující sekvenční diagram: UI hard gate, `consent_token` / `ConsentRecord`, append-only audit, scope a verze textu souhlasu.
- **[H4]** _zprava-hluboky-vyzkum.md_, pasáže **Škálování na až miliony requestů**, **Proč 1M RPS != 1M LLM generací** a navazující sizing poznámky: rozlišení front-door throughput vs inference throughput, potřeba async režimu, quotas a service classes.
- **[H5]** _zprava-hluboky-vyzkum.md_, část **Caching**: L1 in-process cache, Redis jako L2 cache a store pro rate limiting / embedding cache.
- **[H6]** _zprava-hluboky-vyzkum.md_, část **Rizika a mitigace**: preferovat `safetensors`, řešit allowlist/mirror modelů, checksumy, podepisování artefaktů a SBOM.
- **[H7]** _zprava-hluboky-vyzkum.md_, části **Storage**, **Monitoring a observability** a **Migrační / decision table**: licenční/compliance upozornění pro MinIO community a Loki, alternativa Ceph S3 / jiný log backend.
- **[H8]** _zprava-hluboky-vyzkum.md_, poznámka **k guwp**: veřejně nejednoznačně identifikovaná komponenta, proto ji brát jako interní wrapper s povinným due diligence.

### Externí veřejné reference

- **[R1]** MetaVO / MetaCentrum dokumentace: _Documentation – MetaCentrum (MetaVO)_. Uvádí Grid/PBS, Cloud a Kubernetes služby.  
  https://metavo.metacentrum.cz/cs/docs/index.html
- **[R2]** MetaCentrum Documentation. Uvádí, že MetaCentrum je vstupní bod do tří infrastruktur: **Grid založený na OpenPBS**, Cloud a Kubernetes.  
  https://docs.metacentrum.cz/
- **[R3]** RabbitMQ Docs: _Consumer Acknowledgements and Publisher Confirms_.  
  https://www.rabbitmq.com/docs/confirms
- **[R4]** RabbitMQ Docs: _Dead Letter Exchanges_.  
  https://www.rabbitmq.com/docs/dlx
- **[R5]** RabbitMQ Docs: _Queues / Priorities_ a _Priority Support in Queues_.  
  https://www.rabbitmq.com/docs/queues  
  https://www.rabbitmq.com/docs/priority
- **[R6]** Prefect Docs: _How to automatically rerun your workflow when it fails_.  
  https://docs.prefect.io/v3/how-to-guides/workflows/retries
- **[R7]** Prefect Docs: _How to manually retry a flow run_.  
  https://docs.prefect.io/v3/how-to-guides/workflows/retry-flow-runs
- **[R8]** Prefect Docs: _States_.  
  https://docs.prefect.io/v3/concepts/states
- **[R9]** Prefect Docs: _Flows_.  
  https://docs.prefect.io/v3/concepts/flows
- **[R10]** OpenSearch Docs: _Why use OpenSearch?_ / lexical search and vector database positioning.  
  https://docs.opensearch.org/latest/about/
- **[R11]** OpenSearch Docs: _Hybrid search_.  
  https://docs.opensearch.org/latest/vector-search/ai-search/hybrid-search/index/
- **[R12]** OpenSearch Docs: _Vector search_.  
  https://docs.opensearch.org/latest/vector-search/
- **[R13]** OpenSearch Docs: _Optimizing hybrid search_.  
  https://docs.opensearch.org/latest/search-plugins/search-relevance/optimize-hybrid-search/
- **[R14]** vLLM project site: high-throughput serving, continuous batching, advanced scheduling.  
  https://vllm.ai/
- **[R15]** vLLM Docs: documentation portal and online serving examples.  
  https://docs.vllm.ai/en/latest/
- **[R16]** vLLM Docs: stable docs with continuous batching / high-throughput serving references.  
  https://docs.vllm.ai/en/v0.8.2/
- **[R17]** vLLM Docs: optimization and chunked prefill for throughput/latency tuning.  
  https://docs.vllm.ai/en/stable/configuration/optimization/
- **[R18]** Hugging Face Docs: _GGUF_.  
  https://huggingface.co/docs/hub/gguf
- **[R19]** Hugging Face Docs: _GGUF and interaction with Transformers_.  
  https://huggingface.co/docs/transformers/gguf
- **[R20]** Hugging Face Docs: _GGUF usage with llama.cpp_.  
  https://huggingface.co/docs/hub/gguf-llamacpp
- **[R21]** llama.cpp repository: project description and runtime positioning.  
  https://github.com/ggml-org/llama.cpp
