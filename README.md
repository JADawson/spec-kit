<div align="center">
    <img src="./media/logo_small.webp"/>
    <h1>🌱 Spec-Kit-BI</h1>
    <h3><em>Spec-Driven Development scaffold for BI and Medallion Architecture.</em></h3>
</div>

<p align="center">
    <strong>Derived from Spec-Kit; retains artifact-first workflow.</strong>
</p>

---

## Purpose

This repository is a preparation-stage fork for building a BI-focused derivative of Spec-Kit.

This PR intentionally limits scope to:

- repository cleanup/rebrand,
- folder and template scaffolding,
- preserving core harness behavior.

It intentionally does **not** implement BI slash commands or full BI artifact templates yet.

## What stays the same

- Core Spec-Kit harness and workflow patterns
- `specs/<feature>/` feature-folder convention
- Existing scripts under `scripts/bash/*` and `scripts/powershell/*`
- Agent-context update approach and prerequisite checking model

## New BI Scaffold

- BI documentation workspace: `bi/` and `docs/bi/`
- Examples placeholder: `examples/`
- BI template placeholders under `templates/`
- BI command placeholders under `templates/commands/`

See:

- [Workflow guide](./spec-driven.md)
- [BI roadmap](./docs/bi/roadmap.md)
- [Migration TODO](./docs/bi/migration-todo.md)

## Legacy / Heritage

To avoid mixing generic app-building examples into BI-first onboarding, the original long-form narrative has been relocated to:

- [`docs/legacy/spec-driven.md`](./docs/legacy/spec-driven.md)

## Quick start (current harness)

```bash
# Initialize from your fork
uv tool install specify-cli --from git+https://github.com/your-org/spec-kit-bi.git

# Create a working project
specify init my-bi-project
specify check
```

## License

MIT (same as upstream Spec-Kit).
