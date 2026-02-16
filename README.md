# Therapeutic Intelligence System (TIS)

An **Agno-based, config-driven therapeutic + market intelligence engine** for pharma/biotech competitive analysis.

## 1) Final Blueprint (Agno-first)

### 1.1 Business problem
Competitive intelligence teams need fast, evidence-backed answers to questions such as:
- Compare one company vs top competitors in a disease/target landscape.
- Track what changed since a time cutoff.
- Restrict to geography, trial phase, status, or source-specific scope.

Manual workflows (ClinicalTrials.gov + PubMed + press + pipeline scans) are slow, inconsistent, and hard to audit.

### 1.2 System goal
Build a **deterministic + agentic** intelligence system that:
1. Accepts natural language queries.
2. Maps each request to a strict `QueryConfig`.
3. Retrieves trusted evidence deterministically.
4. Normalizes evidence into canonical objects.
5. Runs Agno analyst agents on structured evidence.
6. Validates outputs to prevent hallucinations.
7. Renders stable markdown reports with citations.
8. Supports multi-turn refinements using config deltas.

### 1.3 Core constraints
- High factuality and strict evidence traceability.
- Full run auditability and replay.
- Stable report templates.
- Controlled tool usage (no uncontrolled browsing).
- Graceful partial output under source/tool failure.

---

## 2) Product definition

### 2.1 User surfaces
- Chat-style interface and/or FastAPI endpoint.
- Full report and partial report requests.
- Follow-up refinements (time, geography, phase, scope).

### 2.2 Output types
- Full markdown report.
- Evidence appendix (NCT / PMID / URL).
- Competitor ranking tables.
- Partial outputs:
  - market-only digest
  - clinical-only landscape
  - scientific-only digest
  - ranking-only output
  - delta report ("what changed since X")

---

## 3) Workflow types (finite set)

Every query maps to one workflow:
- `competitive_intelligence`
- `market_digest`
- `clinical_trials_landscape`
- `scientific_digest`
- `target_landscape`
- `indication_landscape`
- `delta_report`

---

## 4) Agno-aligned architecture

### 4.1 End-to-end flow
`User Query -> Intent Router -> Query Router -> QueryConfig -> Retrieval Plan -> Deterministic Fetchers -> Canonicalization -> Agno Analyst Agents -> Validator -> Synthesis -> Markdown Renderer -> Run Store`

### 4.2 Design principle
This is **not** free-form browsing with an LLM.
It is a **deterministic evidence pipeline** with Agno agents only for interpretation/synthesis.

### 4.3 Agno role in the stack
Use Agno for:
- agent definitions and role prompts
- controlled orchestration of analyst steps
- structured IO contracts between stages

Keep deterministic modules outside agents for:
- retrieval
- normalization
- validation checks
- storage/replay

---

## 5) Requirements

### 5.1 Functional requirements
1. Query classification into supported workflows.
2. Query -> strict `QueryConfig` conversion.
3. Multi-turn refinement via `DeltaConfig` patches.
4. Deterministic retrieval from:
   - ClinicalTrials.gov
   - PubMed
   - curated market news
   - company pipeline pages
   - press releases
5. Canonical normalization into:
   - `ClinicalTrial`
   - `Publication`
   - `MarketEvent`
   - `PipelineAsset`
6. Evidence traceability for each major claim.
7. Analyst outputs for clinical/scientific/market/ranking dimensions.
8. Validation gate that removes/blocks unsupported claims.
9. Stable markdown rendering (versioned templates).
10. Run artifact storage for replay and audit.

### 5.2 Non-functional requirements
- Reliability: retries + partial-result fallback.
- Auditability: tool logs + artifact persistence + replay.
- Stability: deterministic structure for same config.
- Security: JWT auth (prod), allowlisted outbound domains, no arbitrary code execution.
- Maintainability: strict schemas (Pydantic), modular contracts.
- Performance: POC 30–120s, target 20–60s with caching.
- Scalability: easy source/template/workflow extension.

---

## 6) Agents (Agno implementation plan)

### 6.1 Agent list
1. **Intent Router Agent**
   - Output: `new_query | refinement | follow_up | out_of_domain`
2. **Query Router Agent**
   - Output: validated `QueryConfig`
3. **Clinical Analyst Agent**
   - Input: canonical trials
   - Output: trial landscape insights + key trials
4. **Scientific Analyst Agent**
   - Input: canonical publications
   - Output: MoA / novelty / resistance insights
5. **Market Analyst Agent**
   - Input: market events + pipeline assets
   - Output: strategic signals
6. **Competitor Ranker Agent**
   - Input: consolidated evidence features
   - Output: top-k ranking + explicit scoring logic
7. **Validator/Auditor Agent**
   - Input: claims + canonical evidence
   - Output: issue list (`LOW|MEDIUM|HIGH`)
8. **Synthesizer Agent**
   - Input: validated sections
   - Output: report blocks ready for rendering

### 6.2 Memory policy
- No hidden long-term memory inside agents.
- State is deterministic in `ConversationState` + run artifacts.

---

## 7) Deterministic tools and contracts

### 7.1 Tool inventory
- `clinicaltrials_fetcher`
- `pubmed_fetcher`
- `market_news_fetcher`
- `pipeline_fetcher`
- `press_release_fetcher`
- `html_cleaner`
- canonical normalizers
- cache store
- run artifact store

### 7.2 Mandatory tool return schema
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

---

## 8) Ontology and coding layer

### 8.1 ICD-10 (recommended)
Use ICD-10 for:
- indication normalization
- synonym expansion
- deterministic filtering/refinement
- retrieval query quality

Store codes in canonical indication fields and in `QueryConfig` when available.

### 8.2 ClinVar (optional)
Use ClinVar only for variant-centric workflows (e.g., EGFR exon-level intelligence).
Do not rely on ClinVar for market events or broad competitor intelligence.

---

## 9) Core contracts

### 9.1 QueryConfig
Must include:
- query type
- primary entity
- indication (+ optional ICD-10)
- target/biomarker
- geography
- time window
- filters (phase/status)
- output format
- template version

### 9.2 ConversationState
Stores:
- active `QueryConfig`
- `last_run_id`
- cached evidence keys
- user preferences

### 9.3 DeltaConfig
Patch object for refinement turns (time/geo/filter/scope changes).

---

## 10) Validation and failure policy

### 10.1 Retry and fallback
- 3 retries with exponential backoff.
- Source-level timeouts and rate-limit handling.
- Partial evidence allowed with explicit warnings.

### 10.2 Agent fallback
- If schema validation fails: rerun once with stricter instructions.
- If still invalid: emit empty section + `insufficient evidence` note.

### 10.3 Validator enforcement
- HIGH-severity issues must be fixed or removed before final report.
- No unsupported claim passes to rendering.

---

## 11) Rendering policy
- Versioned templates (e.g., `report_v2_1`).
- Fixed section order and required blocks.
- Mandatory evidence appendix.
- Limitations/warnings section when evidence is partial.

---

## 12) Agno-first implementation plan

### 12.1 Notebook sequence
1. system overview + schemas
2. intent router
3. query router
4. ICD-10 mapper
5. ClinicalTrials fetcher
6. PubMed fetcher
7. market + pipeline fetchers
8. canonicalization
9. clinical analyst (Agno)
10. scientific analyst (Agno)
11. market analyst (Agno)
12. competitor ranker (Agno)
13. validator (Agno + deterministic checks)
14. fix/rerun loop
15. synthesis + renderer
16. end-to-end run notebook
17. multi-turn refinement notebook
18. modularization checklist

### 12.2 Module layout (production-ready)
- `tis_core/schemas`
- `tis_core/tools`
- `tis_core/normalizers`
- `tis_core/agents` *(Agno agent definitions)*
- `tis_core/workflows` *(Agno orchestration + stage runners)*
- `tis_core/validators`
- `tis_core/rendering`
- `tis_core/storage`
- `app/` *(FastAPI endpoints + auth + middleware)*

---

## 13) POC -> production phases
- **Phase 0:** schemas, logging, run store, query routing.
- **Phase 1:** retrieval + canonicalization + caching/retries.
- **Phase 2:** clinical/scientific Agno agents + markdown rendering.
- **Phase 3:** market + competitor ranking agents.
- **Phase 4:** validator hardening + multi-turn delta execution.
- **Phase 5:** FastAPI packaging, auth hardening, observability, jobs.

---

## 14) Acceptance criteria
System is complete when it can:
- answer 20+ diverse in-domain queries,
- support multi-turn refinements,
- produce stable template-driven reports,
- include evidence appendix and citations,
- block unsupported claims via validation,
- store complete run artifacts for replay.
