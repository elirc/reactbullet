# Architectural Cartographer

This suite is a top-down onboarding map for this repository with a bias toward reading code like an engineer, not just consuming explanations.

The repository is a mono-repo with three app implementations that share the same domain and nearly the same feature modules: `react-vite`, `nextjs-app`, and `nextjs-pages` (Ref: `package.json:7`, `apps/react-vite/src/features/discussions/components/discussion-view.tsx:8-50`, `apps/nextjs-app/src/features/discussions/components/discussion-view.tsx:8-53`, `apps/nextjs-pages/src/features/discussions/components/discussion-view.tsx:8-53`).

## How To Use This Suite

1. Read `00-reading-map.md` first to get the map, not details.
2. Work through `01-junior-engineer.md` with terminal and editor open.
3. Move to `02-mid-level-engineer.md` and trace one feature end-to-end yourself.
4. Use `03-senior-engineer.md` to practice critique, risk analysis, and ownership planning.
5. Keep `04-reference-suite.md` open while doing real implementation or review work.

## Recommended Pace

1. Day 1: `00` + setup section in `01`.
2. Day 2-3: finish `01` checkpoints.
3. Day 4-5: finish `02` and complete diff-reading exercise.
4. Day 6+: iterate through `03` and compare your own technical judgments.

## Checkpoint Method

Each checkpoint is designed to force recall and synthesis. Do not skip self-grading. The value is in proving reasoning, not recognition.

