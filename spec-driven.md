# Spec-Kit-BI Workflow Guide

Spec-Kit-BI is a BI-focused derivative of Spec-Kit.

> Heritage: Derived from Spec-Kit; retains artifact-first workflow.

## Current Status

This repository currently preserves the original Spec-Kit harness while preparing BI/Medallion-specific artifacts.

## Workflow (Unchanged Core Harness)

1. Create a feature folder under `specs/<feature-name>/`.
2. Produce core artifacts (`spec.md`, `plan.md`, `tasks.md`) with existing command flow.
3. Iterate via agent context and prerequisite checks as in upstream Spec-Kit.

## BI Roadmap (Scaffold Only)

Future PRs will add BI-specific slash-command content and templates for:

- discovery
- bronze
- silver
- gold
- semantic
- metric catalog
- report plan

## Legacy Reference

The original long-form Spec-Driven Development narrative was relocated to:

- `docs/legacy/spec-driven.md`
