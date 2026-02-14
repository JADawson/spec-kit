# Quick Start

This quick start keeps the original harness flow while preparing for BI-specific artifacts.

## 1) Initialize a project

```bash
uvx --from git+https://github.com/your-org/spec-kit-bi.git specify init <PROJECT_NAME>
```

## 2) Validate tools

```bash
specify check
```

## 3) Run existing artifact-first flow

Use current core commands to scaffold a feature under `specs/<feature>/`:

```bash
/speckit.constitution Define BI data governance principles
/speckit.specify Define requirements for medallion-based KPI delivery
/speckit.plan Propose medallion layers, semantic modeling approach, and validation strategy
/speckit.tasks
```

## 4) Track migration status

See [docs/bi/migration-todo.md](bi/migration-todo.md) for deferred BI command/template implementation work.
