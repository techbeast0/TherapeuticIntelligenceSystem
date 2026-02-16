# Therapeutic Intelligence System (TIS)

A **config-driven therapeutic + market intelligence engine** for pharma/biotech competitive analysis.

## 1) Final Blueprint

### 1.1 Problem statement
Competitive intelligence teams need fast, evidence-backed answers to questions such as:
- Compare one company vs top competitors in a disease/target landscape.
- Track changes since a time cutoff.
- Restrict to specific geographies, trial phases, or evidence types.

The system must replace manual, slow, inconsistent workflows with a reproducible pipeline.

### 1.2 System goal
Build a hybrid deterministic + agentic system that:
1. Accepts natural language intelligence queries.
2. Converts them into strict structured config.
3. Retrieves evidence deterministically from trusted sources.
4. Normalizes all evidence into canonical objects.
5. Runs domain analyst agents on structured evidence.
6. Validates outputs to prevent hallucinations.
7. Renders stable, traceable reports.
8. Supports multi-turn refinements via config patches.

### 1.3 Core constraints
- High factuality and evidence traceability.
- Auditability and reproducibility per run.
- Stable template formatting.
- Safe, deterministic tool layer with controlled sources.
- Graceful partial-output behavior on failures.

## 2) Product definition

### 2.1 User surfaces
- Chat-like interface and/or API endpoint.
- Full-report and partial-answer requests.
- Follow-up refinements (time, geography, scope, filters).

### 2.2 Output types
- Full markdown report (presentation-grade).
- Evidence appendix (NCT / PMID / URL references).
- Competitor ranking tables.
- Partial outputs:
  - market-only digest
  - clinical-only landscape
  - scientific-only digest
  - ranking-only output
  - delta report (what changed since X)

## 3) Requirements

### 3.1 Functional requirements
1. **Natural language query classification** into finite workflows:
   - `competitive_intelligence`
   - `market_digest`
   - `clinical_trials_landscape`
   - `scientific_digest`
   - `target_landscape`
   - `indication_landscape`
   - `delta_report`
2. **Query → Config conversion** to validated `QueryConfig`.
3. **Multi-turn refinement** via `ConversationState` + `DeltaConfig`.
4. **Deterministic evidence retrieval** from:
   - ClinicalTrials.gov
   - PubMed
   - curated market news
   - company pipeline pages
   - press releases
5. **Canonical normalization** to:
   - `ClinicalTrial`
   - `Publication`
   - `MarketEvent`
   - `PipelineAsset`
6. **Evidence traceability** for every major claim.
7. **Analyst reasoning** for clinical/scientific/market/competitor insights.
8. **Validation controls** to block unsupported claims.
9. **Output rendering** to markdown (PDF optional later).
10. **Run storage** for replay and audit.

### 3.2 Non-functional requirements
- Reliability with retries and partial-result fallback.
- Compliance-ready audit trail (tool logs, artifacts, replayability).
- Deterministic/stable structure for same config.
- Security: auth, allowlist outbound domains, no uncontrolled browsing.
- Maintainability: strict schemas + modular interfaces.
- Performance: POC 30–120s; production 20–60s with caching.
- Scalability: easy source/template/workflow extension.

## 4) Architecture

### 4.1 End-to-end flow
`User Query -> Intent Router -> Query Router -> QueryConfig -> Workflow -> Evidence Fetch -> Canonical Normalize -> Analyst Agents -> Validator -> Synthesis -> Renderer -> Run Store`

### 4.2 Design principle
This is **not** “LLM with ad-hoc tools.”
It is a **deterministic data pipeline with an LLM interpretation layer**.
- Deterministic layers own truth, structure, and retrieval.
- Agents own interpretation and synthesis.
- Validator enforces evidence integrity.

## 5) Agent system (minimum complete set)
1. **Intent Router Agent**: classify message (`new_query`, `refinement`, `follow_up`, `out_of_domain`).
2. **Query Router Agent**: generate strict `QueryConfig` JSON.
3. **Evidence Planner Agent** (optional): choose retrieval plan.
4. **Clinical Analyst Agent**: analyze trials.
5. **Scientific Analyst Agent**: analyze publications.
6. **Market Analyst Agent**: analyze market and pipeline signals.
7. **Competitor Ranker Agent**: top-k competitor scoring.
8. **Validator/Auditor Agent**: evidence alignment and contradiction checks.
9. **Synthesizer Agent**: compose report-ready structured sections.

All agents run memory-light with explicit structured inputs.

## 6) Deterministic tools and contracts

### 6.1 Tooling
- `clinicaltrials_fetcher`
- `pubmed_fetcher`
- `market_news_fetcher`
- `pipeline_fetcher`
- `press_release_fetcher`
- `html_cleaner`
- canonical normalizers
- cache store
- run artifact store

### 6.2 Mandatory return contract
```json
{
  "status": "ok|error",
  "data": {},
  "retrieved_at": "ISO-8601",
  "source": "string",
  "warnings": [],
  "errors": []
}
```

## 7) ICD-10 and ClinVar integration policy

### 7.1 ICD-10 (recommended)
Use ICD-10 in the ontology/normalization layer for:
- indication normalization
- synonym expansion
- deterministic filtering and refinement
- better retrieval consistency

Store ICD-10 in canonical indication attributes and QueryConfig.

### 7.2 ClinVar (optional, variant-centric)
Use ClinVar only for variant-specific genomics workflows (e.g., resistance variants).
Do **not** treat it as a primary source for market intelligence or broad competitor analysis.

## 8) Failure handling and safety policy
- Retry: 3 attempts with exponential backoff.
- Per-source timeouts and rate-limit handling.
- Partial evidence fallback with warnings + confidence adjustment.
- Agent schema failure: rerun once with stricter prompt.
- Persisting failure: return empty section with `insufficient evidence`.
- HIGH-severity validator issues must be corrected or removed before final report.

## 9) Core data contracts

### 9.1 QueryConfig
Must include:
- query type
- primary entity
- indication (+ optional ICD-10)
- target/biomarker
- geography
- time window
- filters (phase/status)
- requested output format
- template version

### 9.2 ConversationState
Stores:
- active QueryConfig
- last run id
- cached evidence keys
- user preferences

### 9.3 DeltaConfig
Represents refinements (time/geo/filter/scope changes) to patch previous config.

## 10) Versioned rendering
- Template files versioned (e.g., `report_v2_1`).
- Renderer enforces section order, required blocks, and appendix.
- Includes limitation/warning sections when evidence is partial.

## 11) Notebook-first implementation plan
Implement in this order:
1. system overview + schemas
2. intent router
3. query router
4. ICD-10 mapper
5. ClinicalTrials fetcher
6. PubMed fetcher
7. market + pipeline fetchers
8. canonicalization
9. clinical analyst
10. scientific analyst
11. market analyst
12. competitor ranker
13. validator
14. fix/rerun loop
15. synthesis + renderer
16. end-to-end notebook
17. multi-turn refinement notebook
18. modularization checklist notebook

## 12) Modularization target structure
- `tis_core/schemas`
- `tis_core/tools`
- `tis_core/normalizers`
- `tis_core/agents`
- `tis_core/orchestrator`
- `tis_core/rendering`
- `tis_core/storage`
- `app/` (FastAPI)

## 13) POC to production phases
- **Phase 0**: schemas, run storage, query routing.
- **Phase 1**: deterministic retrieval + normalization.
- **Phase 2**: clinical/scientific agents + markdown rendering.
- **Phase 3**: market intelligence + competitor ranking.
- **Phase 4**: validator hardening + multi-turn support.
- **Phase 5**: API modularization, auth, observability, production infra.

## 14) Acceptance criteria
System is complete when it can:
- answer 20+ diverse in-domain queries,
- support multi-turn refinements,
- produce stable template-driven outputs,
- include evidence appendix and citations,
- block unsupported claims,
- store complete run artifacts for replay.
