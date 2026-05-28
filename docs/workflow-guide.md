# Workflow Guide

How the pi-superpowers skills work together — and what it looks like when
they're working.

## The Mental Model

The workflow is a **funnel, not a checklist**. Big ideas enter wide
(brainstorming) and narrow toward shipping (finishing). You can enter at
any point depending on what you already know:

```
"I have an idea"      → brainstorming → writing-plans → execution
"I have a plan"       → execution (executing-plans or subagent-driven-development)
"I have a bug"        → systematic-debugging
"I'm done, I think"   → verification → code review → finishing
```

Skills auto-activate based on what you say. You don't need to invoke
anything explicitly. The key triggers:

- **Describe a feature or idea** → brainstorming kicks in (asks
  questions one at a time)
- **Say "let's implement"** → writing-plans creates a task list, then
  executing-plans runs through it
- **A test fails or something breaks** → systematic-debugging takes over
- **Say "I think that's done"** → verification-before-completion demands
  evidence before claims

**How to tell which skill is active:**

- Agent is asking clarifying questions one at a time → brainstorming
- Agent presents a design in 200-300 word sections → brainstorming
- Agent produces a numbered task list with TDD steps → writing-plans
- Agent works RED-GREEN-REFACTOR on each task → executing-plans + TDD
- Agent collects evidence and forms hypotheses → systematic-debugging
- Agent runs tests and shows output before claiming done → verification
- Agent presents severity-categorized findings → requesting-code-review
- Agent asks "merge, PR, or park?" → finishing-a-development-branch

---

## What It Looks Like When It's Working

Three annotated sessions showing the skills in action. Margin notes
(`←`) show which skill is driving at each point.

### Example 1: Full Lifecycle — "Add a changelog skill"

> **You:** I want a skill that generates changelogs from git history.

← **brainstorming** activates. The agent doesn't start coding — it asks
questions to understand what you actually want.

> **Agent:** Before we design this, a few questions. What format are you
> thinking?
> a) Conventional Commits → grouped by type (feat, fix, chore)
> b) Freeform — just a list of commit subjects
> c) Something else?

> **You:** a) Conventional Commits.

> **Agent:** Should it generate for a specific range, or always since
> the last tag?
>
> a) Since last tag (most common for releases)
> b) Between two arbitrary refs
> c) Both, with "since last tag" as default

> **You:** c)

The agent continues asking one question at a time — scope, output file,
edge cases — until it has enough to propose a design.

> **Agent:** Here's the first section of the design: ...
> Does this look right so far?

← Still **brainstorming**. The agent presents the design in sections,
checking after each one. Once validated, it writes the design doc.

> **Agent:** Design written to
> `docs/plans/2026-05-28-changelog-skill-design.md` and committed.
> Ready to set up for implementation?

> **You:** Yes.

← **using-git-worktrees** creates an isolated workspace. Then
**writing-plans** produces a task list.

> **Agent:** Here's the implementation plan:
> 1. Create skill directory and SKILL.md with frontmatter
> 2. Write git log parsing logic with tests
> 3. Add grouping by conventional commit type
> 4. ...
>
> Ready to execute?

> **You:** Go.

← **executing-plans** takes over. Each task follows
**test-driven-development** — write a failing test, make it pass,
refactor.

The agent works through the plan. After the last task:

> **Agent:** All tasks complete. Running full test suite...
> ✓ 14 tests passing. No failures.

← **verification-before-completion** — the agent shows evidence, not
just claims.

> **Agent:** Running code review before we finish...
> **Minor:** Consider extracting the date formatter.
> **Suggestion:** Add a `--format` flag for future extensibility.
> No critical or major issues found.

← **requesting-code-review** — the agent reviews its own work with
severity categories.

> **Agent:** Implementation complete. How would you like to proceed?
> a) Merge to main
> b) Create a PR
> c) Park the branch for later

← **finishing-a-development-branch** presents your options.

---

### Example 2: Mid-Stream — "Tests are failing after a refactor"

> **You:** I refactored the parser module and now 3 tests fail.

← **systematic-debugging** activates immediately. No brainstorming
needed — you have a concrete problem.

> **Agent:** Let me collect evidence. Running the test suite...
>
> 3 failures in `parser.test.ts`:
> - `test_parse_header`: expected "title" got undefined
> - `test_parse_body`: expected array got string
> - `test_parse_empty`: TypeError: Cannot read property 'split'
>
> All three fail on property access. Hypothesis: the refactor changed
> the return shape of `parse()`. Let me check the diff...
>
> Confirmed. `parse()` used to return `{title, body}` but now returns
> `{metadata: {title}, content: body}`. The tests expect the old shape.

The agent traces the root cause (changed return type), doesn't guess.

> **Agent:** Two options:
> a) Update the 3 tests to match the new shape (if the new shape is
>    intentional)
> b) Revert the return shape (if it was accidental)
>
> Which was intended?

> **You:** The new shape is intentional.

> **Agent:** Updated all three tests. Running suite...
> ✓ All 12 tests passing.

← **verification-before-completion** — shows the actual test output.

---

### Example 3: Quick Task — "Fix the typo in the README"

> **You:** There's a typo in the README — "impelmentation" on line 12.

> **Agent:** Fixed "impelmentation" → "implementation" in README.md,
> line 12. Committed.

← No skill activates. The agent just fixes it. Not every change needs
the full ceremony. Skills activate when the task warrants them — small
fixes stay small.

---

## Skill Quick Reference

| Skill | Activates when... | What it does | Leads to... |
|-------|-------------------|--------------|-------------|
| **brainstorming** | You describe a new feature or idea | Asks questions one at a time, proposes approaches, presents design in sections | writing-plans |
| **writing-plans** | You have a validated design | Creates a numbered task list with TDD steps | executing-plans |
| **using-git-worktrees** | Starting feature work needing isolation | Creates an isolated branch and worktree | executing-plans |
| **executing-plans** | You have a written plan | Works through tasks sequentially with review checkpoints | finishing |
| **subagent-driven-development** | Plan with independent tasks | Dispatches one subagent per task with two-stage review | finishing |
| **dispatching-parallel-agents** | 2+ independent tasks, no shared state | Runs multiple subagents concurrently | varies |
| **test-driven-development** | Implementing any feature or fix | RED-GREEN-REFACTOR cycle | used within execution |
| **systematic-debugging** | A bug, test failure, or unexpected behavior | 4-phase root cause investigation | verification |
| **verification-before-completion** | About to claim work is done | Runs tests, demands evidence before assertions | code review |
| **requesting-code-review** | Before merging | Reviews with severity categories | finishing |
| **receiving-code-review** | You get review feedback | Evaluates feedback technically, doesn't blindly agree | apply or push back |
| **finishing-a-development-branch** | All tests pass, work is complete | Presents options: merge, PR, or park | done |
| **writing-skills** | Creating or editing skills | TDD applied to process documentation | done |
