# Personal Skills Repository

This repository houses Gabriel's personal Claude/OpenCode skills.

Each installable skill lives under `skills/<skill-name>/` so it can be added individually from the repository.

## Contents

- `skills/` - source directory for personal skills
- `skills/legend-state/` - current skill package
- `skills/legend-state/SKILL.md` - main skill instructions and trigger description
- `skills/legend-state/references/` - supporting reference material for core API and sync/persistence patterns
- `skills/legend-state/evals/` - eval prompts used to test the skill
- `skills/konva-js/` - Konva.js canvas utilities skill (stage, shapes, filters, react-konva)
- `skills/konva-js/SKILL.md` - Konva.js skill instructions and reference
- `skills/konva-js/references/` - supporting reference material for shapes, animation, filters, and performance
- `skills/konva-js/evals/` - eval prompts for Konva use-cases (react transformer, image filters, performance)

## Generated artifacts

These are intentionally ignored by `.gitignore`:

- `*-workspace/` - eval runs, grading, benchmark output, and review HTML
- `dist/` - packaged `.skill` files

# Workspaces

- `<skill-name>-workspace/iteration-<n>/` — contains per-eval run outputs for with_skill and without_skill runs, grading.json, and artifacts produced while iterating on a skill. Example: `konva-js-workspace/iteration-1/` was created for initial Konva evals.

## Packaging

Packaged archives for installation are written to `dist/`.

Today that means `dist/legend-state.skill`, with more packaged skills expected over time.

## Installing a skill

Install an individual skill from this collection with:

```bash
bunx skills add https://github.com/p7gg/skills --skill legend-state
bunx skills add https://github.com/p7gg/skills --skill konva-js
```

## Adding a New Skill

To add another personal skill to this repo:

1. Create a new directory at `skills/<skill-name>/`.
2. Add a `skills/<skill-name>/SKILL.md` file with the skill metadata, trigger description, and operating instructions.
3. Add any supporting docs under `skills/<skill-name>/references/`.
4. Add evals under `skills/<skill-name>/evals/` if you want the skill to be testable and benchmarkable.
5. Update this `README.md` if the new skill changes repository structure, packaging conventions, or maintenance workflow.

Naming conventions:

- Use kebab-case directory names like `skills/my-skill/`.
- Keep packaged artifacts named `dist/<skill-name>.skill`.
- Keep temporary eval output in a matching workspace directory like `<skill-name>-workspace/`.

## Purpose

This repo is meant to collect and maintain reusable personal skills.

The current Legend-State skill is designed to help with:

- building new Legend-State code
- refactoring existing React state to Legend-State v3
- debugging observable/reactivity issues
- using fine-grained rendering patterns correctly
- setting up persistence and local-first sync with Legend-State
