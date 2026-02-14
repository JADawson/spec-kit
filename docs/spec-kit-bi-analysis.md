# Spec Kit Prompting Framework Analysis (BI Adaptation)

## A) Site Map

### 1) Slash commands (`templates/commands/*.md`)
- `/speckit.constitution` (`templates/commands/constitution.md`)
  - Purpose: instantiate/update governing principles in `memory/constitution.md`, then propagate consistency to templates and command docs.
  - Inputs: `$ARGUMENTS`, existing constitution template, repo docs/templates.
  - Outputs: overwrite `/memory/constitution.md`, prepend sync-impact HTML comment.
  - Gates: semantic version bump logic, no unresolved placeholders, ISO dates, testable principles.
- `/speckit.specify` (`templates/commands/specify.md`)
  - Purpose: create a feature branch + initial `spec.md`, then quality-validate via checklist loop.
  - Inputs: natural-language feature description, `templates/spec-template.md`, JSON from `create-new-feature` script.
  - Outputs: `specs/<feature>/spec.md`, `specs/<feature>/checklists/requirements.md`.
  - Gates: script must run once; max 3 clarification markers; max 3 validation iterations.
- `/speckit.clarify` (`templates/commands/clarify.md`)
  - Purpose: ask targeted questions and write accepted clarifications back into spec.
  - Inputs: current spec, JSON paths from `check-prerequisites --paths-only`.
  - Outputs: updated `spec.md` (including `## Clarifications` session block).
  - Gates: max 5 asked questions per run; one-question-at-a-time loop; save after each accepted answer.
- `/speckit.plan` (`templates/commands/plan.md`)
  - Purpose: produce technical planning artifacts from spec + constitution.
  - Inputs: `spec.md`, `memory/constitution.md`, copied `plan.md` template, optional user stack constraints.
  - Outputs: `plan.md`, `research.md`, `data-model.md`, `contracts/*`, `quickstart.md`, updated agent context file.
  - Gates: constitution checks and unresolved-clarification failures are errors.
- `/speckit.tasks` (`templates/commands/tasks.md`)
  - Purpose: create executable `tasks.md` organized by user stories.
  - Inputs: `plan.md`, `spec.md`, optional `research.md`, `data-model.md`, `contracts/`, `quickstart.md`.
  - Outputs: `tasks.md`.
  - Gates: strict checklist task line format and required phase structure.
- `/speckit.checklist` (`templates/commands/checklist.md`)
  - Purpose: generate quality checklists that test requirements writing quality (not implementation behavior).
  - Inputs: user focus + feature docs; optional dynamic clarifying questions.
  - Outputs: `specs/<feature>/checklists/*.md`.
  - Gates: prohibited implementation-test wording; preferred traceability (>=80% referenced items).
- `/speckit.analyze` (`templates/commands/analyze.md`)
  - Purpose: read-only cross-artifact consistency report across `spec.md`/`plan.md`/`tasks.md`.
  - Inputs: required docs from prerequisites script + constitution.
  - Outputs: markdown analysis response only (no file writes).
  - Gates: read-only, max 50 findings, constitution conflicts = CRITICAL.
- `/speckit.implement` (`templates/commands/implement.md`)
  - Purpose: execute task plan with dependency order and progress tracking.
  - Inputs: `tasks.md` required; optional design docs and checklist states.
  - Outputs: implementation changes, task checkbox updates.
  - Gates: if checklist items incomplete, must pause and ask for explicit user override.

### 2) Templates (`templates/*.md`)
- `spec-template.md`: requires user stories with priorities and independent tests, FR/NFR-style requirements, success criteria, edge cases.
- `plan-template.md`: enforces technical context + constitution check gate + expected artifact tree.
- `tasks-template.md`: enforces phased execution, strict task syntax, story labels, dependency model.
- `checklist-template.md`: canonical checklist document structure and ID pattern.
- `agent-file-template.md`: template for synthesized agent context summaries extracted from `plan.md`.

### 3) State/context artifacts
- `memory/constitution.md`: constitutional source of truth consumed by planning/analyze and created via `/constitution`.
- `specs/<feature>/`: working state for spec/plan/tasks/research/contracts/checklists/quickstart.
- Agent context files (e.g., `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, `.cursor/rules/specify-rules.mdc`): updated from plan metadata by `update-agent-context` scripts.

### 4) Script substrate assumed by prompts
- `scripts/bash/create-new-feature.sh`
  - Side effects: creates `specs/<NNN-name>/`, creates/checks out feature branch when git is available, copies `spec-template.md` to `spec.md`.
  - JSON: `BRANCH_NAME`, `SPEC_FILE`, `FEATURE_NUM`.
- `scripts/bash/setup-plan.sh`
  - Side effects: ensures feature dir exists; copies `plan-template.md` to `plan.md`.
  - JSON: `FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH`, `HAS_GIT`.
- `scripts/bash/check-prerequisites.sh`
  - Modes used by prompts:
    - `--json`: `{ FEATURE_DIR, AVAILABLE_DOCS }`.
    - `--json --paths-only`: `{ REPO_ROOT, BRANCH, FEATURE_DIR, FEATURE_SPEC, IMPL_PLAN, TASKS }`.
  - Validation gates: feature dir exists; `plan.md` required; optional `tasks.md` required depending on flags.
- `scripts/bash/update-agent-context.sh`
  - Side effects: create/update agent context files by parsing `plan.md` fields; preserves manual additions markers.

## B) Prompt & Guardrail Inventory

| Path | Trigger | Inputs | Outputs | Guardrails | Hidden assumptions | What makes it work |
|---|---|---|---|---|---|---|
| `templates/commands/constitution.md` | `/speckit.constitution` | `$ARGUMENTS`, `/memory/constitution.md`, templates/docs | overwrite constitution | semantic versioning, no unresolved placeholders, ISO dates, MUST/SHOULD declarative language | repo has template placeholders and dependent templates to sync | constitution as top-level policy compiler + sync impact report |
| `templates/commands/specify.md` | `/speckit.specify` | `$ARGUMENTS`, `create-new-feature` JSON, `templates/spec-template.md` | `spec.md`, `checklists/requirements.md` | script once only; max 3 `[NEEDS CLARIFICATION]`; explicit ERROR on empty description / missing flow; bounded validation loop | agent can execute script and parse absolute paths; feature branch naming convention | combines template fill + self-QA checklist + constrained clarification loop |
| `templates/commands/clarify.md` | `/speckit.clarify` | `$ARGUMENTS`, `check-prerequisites --json --paths-only`, `spec.md` | incremental writes to `spec.md` | max 5 questions, one-at-a-time, option tables or <=5 word answers, terminate on user stop | spec exists and is mutable; preserving heading hierarchy | interactive ambiguity reduction with immediate integration and validation |
| `templates/commands/plan.md` | `/speckit.plan` | `$ARGUMENTS`, `setup-plan` JSON, `spec.md`, constitution, plan template | `plan.md` + `research.md` + `data-model.md` + `contracts/` + `quickstart.md` + agent context | ERROR on constitution gate violations/unresolved clarifications; absolute paths | agent can perform “research” and generate missing docs | phased design synthesis with constitution gates and context refresh |
| `templates/commands/tasks.md` | `/speckit.tasks` | `$ARGUMENTS`, `check-prerequisites --json`, plan/spec + optional docs | `tasks.md` | mandatory strict task syntax; user-story organization; tests optional unless requested | plan/spec already present and coherent | dependency-ordered task compiler from artifacts |
| `templates/commands/checklist.md` | `/speckit.checklist` | `$ARGUMENTS`, `check-prerequisites --json`, feature docs | new checklist file under `checklists/` | requirement-quality-only checks; prohibited implementation-verification wording; max 5 clarifying questions; traceability target | user can answer scoping prompts; checklist naming heuristics | reframes QA as “unit tests for English requirements” |
| `templates/commands/analyze.md` | `/speckit.analyze` | `$ARGUMENTS`, `check-prerequisites --json --require-tasks --include-tasks`, spec/plan/tasks/constitution | analysis response only | strictly read-only; max 50 findings; constitution violations CRITICAL | all three core artifacts exist before run | semantic cross-artifact model + coverage mapping |
| `templates/commands/implement.md` | `/speckit.implement` | `$ARGUMENTS`, `check-prerequisites --json --require-tasks --include-tasks`, tasks/plan and optional docs/checklists | implementation changes + task checkboxes marked complete | must pause on incomplete checklists unless user overrides; phase/dependency order; tests-before-code guidance | tasks are sufficiently actionable; environment can execute project tooling | execution orchestration with checklist gate and phase discipline |

## C) End-to-end Workflow Trace

1. Constitution
- Must exist: `memory/constitution.md` template.
- Done when: placeholders resolved, version/dates valid, sync impact report included, dependent template alignment checked.
- Gates: constitutional clarity and semantic-version bump discipline.

2. Specify
- Must exist: initialized `.specify/templates/spec-template.md`; command script runtime.
- Done when: feature branch+folder created, `spec.md` populated, requirements checklist produced and iteratively validated.
- Gates: input non-empty, max clarification markers, validation checklist pass or warned after bounded retries.

3. Clarify
- Must exist: `spec.md`.
- Done when: high-impact ambiguities reduced, clarifications integrated into spec with session log.
- Gates: max question budget, one-question loop, structure/consistency validation after each write.

4. Plan
- Must exist: `spec.md`, constitution; plan scaffold from setup script.
- Done when: `plan.md` and design artifacts (`research.md`, `data-model.md`, `contracts/`, `quickstart.md`) are generated and constitutional checks re-evaluated.
- Gates: unresolved clarification/gate failures are errors.
- Agent context update: happens in Phase 1 via `update-agent-context` so execution agents inherit latest stack/structure guidance.

5. Tasks
- Must exist: `plan.md` and `spec.md` (optionally other design docs).
- Done when: strict-format `tasks.md` with phases, dependencies, and per-story independent test framing.
- Gates: checklist syntax, story grouping, dependency/parallel correctness.

6. Implement
- Must exist: `tasks.md` and `plan.md`; optionally checklists.
- Done when: tasks executed and marked complete, implementation validated against plan/spec.
- Gates: checklist completion gate (pause/override), dependency ordering, phase checkpoints.

## D) Adaptation Blueprint: Spec-Kit-BI

### 1) New BI slash commands
- `/speckit.discovery`
  - Purpose: profile dataset(s), detect quality/shape/risk signals before architecture.
  - Input: source location + business context + SLAs.
  - Output: `discovery.md` (schema profile, nulls/cardinality/drift, PII, key candidacy, relationship hypotheses, incremental refresh suitability).
- `/speckit.bronze`
  - Purpose: ingestion/landing design.
  - Output: `bronze.md` (connectors, ingestion mode, raw zone schema, file format/partitioning, lineage tags).
- `/speckit.silver`
  - Purpose: conformance and data quality modeling.
  - Output: `silver.md` (standardization rules, deduping, SCD policy, conformed dimensions, cleansing transformations).
- `/speckit.gold`
  - Purpose: business-ready star model.
  - Output: `gold.md` (facts/dims, grain, KPI dependencies, aggregate strategy).
- `/speckit.semantic`
  - Purpose: Power BI semantic model blueprint via MCP metadata.
  - Output: `semantic.md` (tables, relationships, measures, calc groups, role-level security, naming standards).
- `/speckit.refresh`
  - Purpose: refresh/partitioning strategy.
  - Output: `refresh.md` (incremental policy, partitions, historical retention, SLA windows, gateway constraints).
- `/speckit.dataquality`
  - Purpose: DQ and reconciliation gating.
  - Output: `dq.md` (tests, thresholds, quarantine rules, reconciliation to source-of-truth, alerting).

### 2) New/modified templates
- Add: `templates/discovery-template.md` (profiling + risk register + assumptions + unknowns).
- Add: `templates/bronze-template.md` (ingestion mode, schema-on-read/write, lineage fields).
- Add: `templates/silver-template.md` (conformance matrix, SCD handling, canonical dimensions).
- Add: `templates/gold-template.md` (star schema + KPI contract + aggregation policy).
- Add: `templates/semantic-template.md` (measure catalog, calc groups, role/security mapping, naming).
- Add: `templates/dq-template.md` (rule inventory, thresholds, monitors, escalation actions).
- Modify: `templates/plan-template.md` to include medallion checkpoints and BI performance gates.
- Keep largely: `templates/tasks-template.md` structure, with BI phase examples.

### 3) BI-specific guardrails to add
- PII handling
  - Mandatory classification per column; explicit masking/tokenization policy before Silver/Gold.
  - RLS requirements for sensitive dimensions and workspace dataset access boundaries.
- Reproducibility/lineage
  - Every Gold metric must trace to Silver/ Bronze source columns with transformation lineage.
  - Require run metadata (load id, source snapshot id, processing timestamp).
- Performance constraints
  - Cardinality and relationship direction constraints in semantic model.
  - Measure complexity and model-size budgets; aggregation table policy.
  - Incremental refresh eligibility gate before production.
- Semantic consistency
  - KPI glossary required; one canonical metric definition per business term.
  - Naming conventions for tables/columns/measures with anti-duplication checks.
- Deployment boundaries
  - Enforce dev/test/prod workspace promotion policy and data source parameterization.

### 4) Constitution changes for BI
- Add principles (example):
  1. **Metric Contract First**: every KPI has owner, formula, grain, and reconciliation target.
  2. **Lineage by Default**: every transformation is traceable Bronze→Silver→Gold.
  3. **Privacy by Design**: sensitive fields classified and protected before semantic exposure.
  4. **Performance is a Feature**: semantic models must satisfy explicit refresh/query SLAs.
  5. **Environment Promotion Discipline**: only tested artifacts promoted between workspaces.

### 5) Migration plan
- Copy with minimal edits:
  - `templates/tasks-template.md`, `templates/checklist-template.md`, `templates/commands/implement.md`, `templates/commands/analyze.md`.
- Rewrite substantially:
  - `templates/spec-template.md` (replace product feature stories with data-product outcomes + metric consumers).
  - `templates/plan-template.md` and `templates/commands/plan.md` (medallion/semantic phases).
  - `memory/constitution.md` defaults (BI principles).
- Keep identical/parity:
  - prerequisite/context scripts shape (`check-prerequisites`, feature-folder conventions) unless adding BI artifacts to `AVAILABLE_DOCS`.
- New BI source-of-truth artifacts:
  - `metric-catalog.md` (canonical KPI contract)
  - `semantic.md` (published model contract)
  - `dq.md` (quality SLO contract)

## E) Concrete Next Steps (Spec-Kit-BI v0)

1. Add folders/files:
   - `templates/commands/discovery.md`, `bronze.md`, `silver.md`, `gold.md`, `semantic.md`, `refresh.md`, `dataquality.md`
   - `templates/discovery-template.md`, `bronze-template.md`, `silver-template.md`, `gold-template.md`, `semantic-template.md`, `dq-template.md`
   - `memory/constitution-bi.md` (or replace default constitution template)
   - `templates/metric-catalog-template.md`
2. Add commands first (highest leverage):
   - `/speckit.discovery` → `/speckit.gold` → `/speckit.semantic`.
3. Draft templates first:
   - `discovery-template.md`, `gold-template.md`, `semantic-template.md`.
4. Script update (minimal):
   - extend prerequisite script doc discovery list to include new BI artifacts (`discovery.md`, `bronze.md`, `silver.md`, `gold.md`, `semantic.md`, `dq.md`, `refresh.md`).
5. Smallest test scenario:
   - Dataset: Orders + Customers CSVs (with nulls, late-arriving rows, one PII column, one drifting column).
   - Expected Gold: `FactSales`, `DimCustomer`, `DimDate` with explicit grain.
   - Measures: `Total Sales`, `Gross Margin %`, `Orders Count`, `AOV`.
   - Gate checks: one PII mask rule, one reconciliation check to source totals, one incremental refresh policy.
