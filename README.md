# Frontend Prompt Guide — VS Code & GitHub Copilot

A compact, import-ready prompt guide for frontend work (paste into a Loop doc). Designed for VS Code + GitHub Copilot workflows. Short, direct, practical — no fluff, a little cheeky.

**How to use**
- **Commit before you prompt.** Create a feature branch and commit current work. Reverting is cheaper than debugging/reverting AI mistakes.  
- **Give minimal, precise context.** Files, exact paths, and the exact goal. Big dumps → token waste and worse results. Break tasks into small steps.  
- **Specify environment every time.** (pnpm v8+, Node 20.x, test runner). If you don’t, the AI defaults to npm and old habits.  
- **Agent mode rule:** Ask the model to **describe the plan and tests** it will run before asking it to implement. If it skips this, stop and request the plan.  
- **Tests first.** If something is testable, ask the AI to write the unit test first (Vitest + @testing-library by default), then implement.  
- **Keep an eye on TS.** Never accept a solution that disables `tsconfig` strict checks. Ask the agent to run `pnpm typecheck` and propose fixes — not shortcuts.

---

It's recomended that the team make some default configuration for Github Copilot, this is a simplified version, have in mind that this should be short since it will eat up context.

**Repo-level Copilot instructions (recommended)**  
Create file: `.github/copilot-instructions.md` and paste:

```
---
applyTo: "**/*.ts,**/*.tsx,**/*.js,**/*.jsx"
---
# DFDS Frontend AI rules (short)
- Use pnpm (pnpm v8+) and Node 20.x for all commands.
- Do not change files outside the explicit file list I provide.
- Do **not** disable TypeScript strict checks. Run `pnpm typecheck` and propose fixes if type errors exist.
- Create unit tests first (Vitest/@testing-library) and include test commands.
- Use approved CVI tokens only (see `/docs/cvi-tokens.md`).
- Commit on a feature branch; provide a suggested commit message and PR title/description.
- Never run or suggest running production scripts or DB queries in the terminal. If needed, request explicit permission.
```

---

**Per-folder / language rules**  
Create `.github/instructions/frontend.instructions.md`:

```
---
applyTo: "**/apps/control-tower/**,**/*.tsx"
---
# Frontend-specific guidance
- Component names: PascalCase. Props strongly typed; no `any`.
- Styling: Tailwind + DFDS CVI tokens only; do not add new global CSS.
- Accessibility: All interactive elements must have aria-label or visible label.
- Prefer small, focused components (single responsibility).
```

---

**VS Code workspace settings (examples)**  
Add to `.vscode/settings.json` (or paste into Workspace settings):

```json
{
  "github.copilot.chat.codeGeneration.useInstructionFiles": true,
  "github.copilot.chat.reviewSelection.instructions": [
    { "file": ".github/guidance/frontend-review-guidelines.md" }
  ],
  "github.copilot.chat.pullRequestDescriptionGeneration.instructions": [
    { "text": "Generate PR body with 3 bullets: purpose, files changed, testing steps." }
  ]
}
```

---

### Be mindful before adding MCP / huge context
**MCP = massive-context prompts** (big file dumps, whole repos, or long multi-file pastes). Don’t drop MCP in casually — think about what the model actually needs to read.

**Why you should think twice (short & technical)**  
- **Token & cost waste.** Large dumps burn tokens fast and cost more.  
- **Silent truncation.** The model can hit the context limit and drop the exact lines you needed.  
- **Attention dilution.** The model wastes capacity on irrelevant code instead of reasoning about the task.  
- **Slower + flakier responses.** Bigger inputs increase latency and the chance of odd, irrelevant output.  
- **Bad shortcuts happen.** Overwhelmed models often suggest quick hacks (disable TS checks, add `// @ts-ignore`) rather than correct fixes.  
- **Harder to review & revert.** Large, sweeping changes are risky; they’re painful to validate and revert.

**Practical rules (do this instead)**  
- Only give the AI access to the things it need, to solve a specific task.
- Paste **only** the files/snippets that matter + a one-line repo rule (pnpm/Node/test-runner).  
- Summarise large files in 2–3 lines instead of dumping them.  
- Break the task into chunks: **plan → tests → implement → review**.  
- Use agent mode to make the model list exactly which files it will change **before** coding.  
- Commit before you prompt. Small focused commits beat one giant, scary PR every time.

**Short rule of thumb:** *less context, right context* — precise files + a one-line environment rule beats dumping the whole repo.

---

### Agent-mode — best-of-both-worlds checklist

Keep prompts small, specific, and testable. Ask the agent to plan and correct itself before coding. The following is thing you can try to use, it depend on you task and complexity

- Isolate & chunk the request (1–2 lines). State the exact goal, environment (pnpm, Node), and the files you expect touched. Smaller = better.
- Ask the agent to describe the plan first. It should list the files it will change (full paths) and give a short implementation plan (3–6 steps). If the plan is missing or fuzzy, stop and request clarification.
- Tests first. Request failing unit tests (Vitest + @testing-library) that reproduce the desired behavior and show the expected results.
- Approve the tests & plan. Only after you sign off on the tests and the step-by-step plan do you let it implement.
- Typecheck & safety checks. Require pnpm typecheck output and a concrete fix list for any TS errors — never disable strict checks.
- Commit & PR metadata up front. Ask for a suggested commit message, branch name, PR title and a 2–3 line PR description.
- Testing steps. Ask for exact commands to run locally (install, test, typecheck, and a quick manual smoke test).
- If it drifts, rewind. If the agent starts changing scope or suggesting shortcuts (e.g., // @ts-ignore), ask it to revert to the approved plan and retry. Commit early and often.

# Quick prompt you can paste:

Scope: Add IconButton component (Node 20, pnpm). Edit only: /apps/control-tower/components/IconButton.tsx, index.ts.
Step 1: Describe the plan and list files changed.
Step 2: Write failing Vitest tests and expected results.
Step 3: After I approve tests, implement and run pnpm typecheck, provide fixes if any.
Also: give suggested commit message and PR title.
---

### Prompt examples

**Prompts you can paste (templates)** But be playfull every model has it own way of working.

**1) Implement feature (small)**
```
Project: Control Tower (Next.js + pnpm)
Goal: Add `IconButton` component used in /apps/control-tower/components/header.tsx
Files to touch: /apps/control-tower/components/IconButton.tsx, /apps/control-tower/components/index.ts
Constraints:
- TypeScript, no `any`
- Tailwind + DFDS CVI tokens only
- Export default as `IconButton`
- Add unit tests (Vitest + @testing-library)
Before coding: describe the plan and list the exact changes (file paths). Then write the tests first.
```

**2) Fix bug**
```
Repo: order-intake
Symptom: Dropdown in InvoiceForm doesn't update when invoice type changes.
Files to inspect: /apps/order-intake/components/InvoiceForm.tsx
Environment: pnpm, Node 20, Vitest
Task: Explain likely root causes and list 3 targeted fixes, with file paths. Then write a failing unit test that reproduces the bug. Do not modify files outside InvoiceForm without permission.
```

**3) Refactor safely**
```
Task: Extract `useFormLogic` hook from /apps/control-tower/components/CheckoutForm.tsx into /apps/control-tower/hooks/useFormLogic.ts
Constraints: Keep public API identical. Update tests. Provide suggested commit message and PR description.
```

**4) PR body generation (short)**
```
Generate PR body with 3 bullets: purpose, files changed, testing steps. Keep it factual and short.
```

---

### Things move fast — what worked yesterday may not work today
- **Tool behavior changes.** New Copilot or VS Code releases can alter how instruction files are interpreted. A workflow that worked last week might break after an update.  
- **Model drift.** A new model version may prefer different defaults (package manager, test runner, or patterns). Always re-run a quick smoke test after upgrading a model or extension.  
- **Dependencies update.** New library versions can change APIs or tests. Pin versions when you need stability.  
- **Practical steps:** after any tool/model update, run a small checklist:
  1. Run `pnpm install && pnpm --filter <app> test`  
  2. Run `pnpm --filter <app> typecheck`  
  3. Try one common Copilot flow (generate a tiny component) and validate output.  
  4. If something breaks vs yesterday, trim context and re-run — or start a fresh chat.

**Short rule:** treat the AI + IDE combo like another dependency — verify after updates and keep expectations flexible.

---

**Useful tips & reminders**
- Always **say which package manager** (pnpm) and Node version. AI defaults to npm unless told otherwise.  
- If asking for refactor/rename, **require tests** to be updated.  
- Keep prompts focused: `"Change only these files: …"` is powerful.  
- When a new model/version ships, **test workflows** — tools behavior can change.  
- Prefer small commits per logical change (one concern per PR).  
- When stuck: open a fresh chat and paste a 1–2 line summary + file list. Large chat histories reduce reliability.

---

**Short examples of "what to paste" into Copilot chat**

**Minimal context for a feature:**
```
pnpm, Node 20. Task: Add IconButton component used in /apps/control-tower/components/header.tsx. Edit only: IconButton.tsx, index.ts. Write tests first. Describe plan before coding.
```

**Debugging:**
```
pnpm. Repo: order-intake. Bug: dropdown not updating in InvoiceForm. Check /apps/order-intake/components/InvoiceForm.tsx. Explain top 3 root causes and propose one minimal fix. Provide a unit test reproducing bug.
```

---
