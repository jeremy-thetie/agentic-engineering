# Agentic Engineering Guide

---

## 1. Core Mindset

- **LLMs/agents do not build mental models.** You own context, vision, and system sense. Agents *amplify* throughput, not strategic thinking or code judgment.
- **Success = Feedback loops.** Every plan, fix, and review should sharpen both code and your “system taste”—codified in brief context files/templates, not just memory.
- **Ship, Learn, Codify, Repeat.** Every PR should upgrade your living standard operating procedure (SOP) or taste/context file.

---

## Non‑Negotiables
- Scope control: Plans list exact files/functions. Out-of-scope must be explicit.
- Diff-only outputs: Agents must show code diffs for listed files only.
- Human review: No merge without a human reading the plan, diff, and tests.
- Minimalism: Prefer the smallest viable change; avoid refactors unless planned.
- Traceability: Reference the plan/ticket in commits; archive research/validation.

## Cost Discipline (Tokens/Time)
- Prefer targeted grep/search and line-cited reads over whole-file dumps.
- Ask before long or broad operations; avoid large, unnecessary rewrites.
- Keep prompts and outputs minimal; strip boilerplate and repetition.
- Store only high-signal artifacts in /notes/*; prune noisy logs.

---

## 2. Canonical Agentic Workflow

### **Phase 1: Targeted Research**

**Goal:** Identify which code, files, or flows need changing—don’t just guess.

- **How:** Use research agents (“/research FEATURE/BUG/GOAL”) or manual grep/tools. Require every research output to cite specific files, lines, or prior notes.
- **Action**: Archive all findings to `/notes/research/`.
- **Template:**
  ```
  ## Research Note [Date/Author]
  - Context/ticket: [link or summary]
  - Areas of code (files/lines): ...
  - Edge cases or prior changes noted: ...
  ```
- **Pitfall:** Accept nothing “hand-wavy”—always review for hallucination or critical omission.

---

### **Phase 2: Concrete Planning**

**Goal:** Create a sequenced, reviewable plan (“contract” with your agent). Don’t skip!

- **How:** Write out step-by-step tasks, file/function targets, expected test coverage, and explicit constraints.
- **Template:**
  ```
  ## Implementation Plan [Date/Feature]
  1. [ ] Update <file/function> to do <X>
  2. [ ] Refactor <Y> as needed (list files!)
  3. [ ] Add/modify test cases (cite test files)
  4. [ ] Validate against [test name or criteria]
  - DO NOT alter <list of out-of-scope files>
  ```
- Checklist:
  - [ ] Enumerate impacted files/functions and owners
  - [ ] Define explicit out-of-scope areas
  - [ ] Specify tests to add/update and pass criteria
  - [ ] Plan rollback and observability (logs/metrics)
  - [ ] Note user-facing impact and comms, if any

- **Pitfall:** Plans must not be vague. If a code review flagged “scope drift” before, update your plan template.

---

### **Phase 3: Implementation**

**Goal:** Let agent(s) or engineers execute the actual code, referencing the plan above only.

- **How:**
  - Assign each plan step to a human or agent (parallelize when possible, tracked via branches/tabs).
  - Require direct file/diff reference in every agent output.
  - Branches: feat/<slug>, fix/<slug>, chore/<slug>
  - Commits: conventional style; reference plan/ticket
  - Guardrails: modify only files listed in the plan; show diffs
  - Sync: small PRs; request early human reviews
- **Example prompt (for an agent):**
  ```
  "Implement Step 1 of the plan above. Output only the code diff for <target file>. DO NOT touch out-of-scope files."
  ```
- **Pitfall:** If the agent invents files, refactors unplanned code, or writes excess boilerplate, STOP and revisit the plan.

---

### **Phase 4: Validation/Test**

**Goal:** Prove output matches plan and works; check for unplanned changes.

- **How:**
  - Run planned tests; add extra only for *new* edge cases found.
  - Have a human or code reviewer agent cross-check against original plan. Use a validation agent if available (“/validate_plan”).
  - Archive test/validation summaries to `/notes/retros/`.
- **Template:**
  ```
  ### Validation Checklist
  - [ ] All plan steps visible in diff/PR
  - [ ] No unplanned files changed
  - [ ] All new/existing tests pass: [test files/results]
  - [ ] Lessons to update context/taste file? [brief]
  ```
- **Pitfall:** Do not let agents write forests of extra tests (“over-testing”). Prune for what matters.

---

### **Phase 5: Commit, PR, and Context Update**

**Goal:** Push code with traceable context, and *evolve your team's taste/context docs*.

- **How:**
  - Reference the plan or ticket in each commit (“Refs: PLAN.md/ISSUE-#…”).
  - Require new learning/taste/tips to land in `AGENT.md` or `/notes/retros/`.
- **Example PR checklist:**
  ```
  - [ ] Diff matches plan; no out-of-scope files
  - [ ] Tests updated/run; only minimal necessary tests
  - [ ] Human review completed; deviations justified in PR
  - [ ] Plan/ticket referenced in commits
  - [ ] Context/taste updated (AGENT.md or /notes/retros/)
  ```
- **Pitfall:** Never skip final human review, even if all agent tests “pass”.

---

## 3. “Taste”/Context Files—**How & What to Update**

- **What belongs:** Team code styles, CLI conventions, pitfalls, bug patterns, “always check X for Y,” common edge cases, business-rule clarifications.
- **Example snippet:**
  ```
  ## AGENT.md snapshot
  - Never refactor <X> without reviewing <Y>.
  - Test case <foo> known-issue: watch for <bar>.
  - Typical perf bottleneck at <api endpoint>: see <tests/perf_foo>.
  ```
- **Keep it short:** Trim old context regularly, keep “taste files” actionable.

---

## 4. Common Pitfalls (and Fixes)

- **Scope/Context Creep:** Agents or people wander outside plan.
    - *Fix*: Validate against plan at each checkpoint.
- **Over-testing:** Agent-generated test bloat.
    - *Fix*: Limit to handful of useful tests. Always review tests like code.
- **Unreviewed agent code:** Ship only with human-owned review.
    - *Fix*: Block merges until plan, diff, and taste/context updates are checked.
- **Loss of learning:** PRs don’t update context/taste files.
    - *Fix*: Make taste/context update a strict PR requirement.
- **Context window overflows:** Models “forget” key team or project context.
    - *Fix*: Routinely condense and re-anchor “living” documents.

---

## 5. Templates & Prompts

**Research Prompt**
```
/research [FEATURE/BUG]
- Summarize context
- List all candidate files/functions (with line nums)
- Note any prior agentic changes/PRs relevant
```
**Plan Prompt**
```
/plan [FEATURE/BUG]
- Stepwise contract, with file/function-exact targets
- Explicit tests to add/run
- “Out of scope” boundaries
```
**Implementation Prompt**
```
/implement [PLAN snippet]
Only show code diffs for listed files;
No extra commentary, no out-of-scope edits.
```
**Validation Prompt**
```
/validate_plan [PLAN, code diff, tests]
Did all plan steps get done, and nothing out-of-scope?
Any suggested context/taste file updates?
```

---

## 6. Example Working File Structure

```
repo-root/
├── src/
├── tests/
└── notes/
    ├── research/      # Agent/human research logs
    ├── plans/         # Implementation plans
    └── retros/        # Retro notes, lessons, validation summaries
AGENT.md              # Context file—team “taste,” don’ts/do’s, bug war stories
```

---

## 7. When to Use (Agentic) vs. Manual

Use the full workflow when:
- Multi-step, multi-file, or “unknown surface” work.
- Team wants compound learning or audit trail.
- Engineering time is too valuable for rote change + research.

Go manual for:
- One-liner fixes, doc updates, or tiny tweaks.
- Emergencies where review-phase is impossible (flag as hotfix, no context update).

---

## 8. Final Notes & Your Customizations

**This guide is intentionally minimal and modular.** Don't let best practices grow stale—prune and revise! Move “how-tos,” anti-patterns, or favorite prompts here as you go.

---

**Own your process, improve every loop, and let the agent do grunt work while you grow your “taste” and team muscle.**
