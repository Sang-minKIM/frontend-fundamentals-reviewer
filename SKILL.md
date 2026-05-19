---
name: frontend-fundamentals-reviewer
description: "Explicit-invocation-only frontend code review skill for PR, MR, diff, branch, commit, or local-change review. Use only when the user explicitly invokes $frontend-fundamentals-reviewer, names this skill, or asks to run the frontend fundamentals reviewer. Runs four read-only review subagents for Toss Frontend Fundamentals criteria: readability, predictability, cohesion, and coupling. Each detailed rule is scored out of 100, each criterion is the equal-weight average of its rules, and the main thread synthesizes conflicts, recommendations, and user choice points. Frontend-only; do not use implicitly for general coding or non-review tasks."
---

# Frontend Review Agents

Run a read-only frontend quality review using Toss Frontend Fundamentals. The goal is to score a concrete change set, explain the evidence behind the scores, and give the user clear tradeoff choices when the criteria disagree.

## Invocation Contract

- Proceed only when the user explicitly invokes `$frontend-fundamentals-reviewer`, names this skill, or asks to run the frontend fundamentals reviewer.
- Treat explicit invocation as permission to dispatch four read-only subagents when the environment supports subagents.
- Do not edit files, stage changes, commit, push, resolve review threads, or apply fixes unless the user separately asks for implementation.
- Review frontend surfaces only: TypeScript/JavaScript, React/Vue/Svelte components, hooks, state, routing, styles, tests, stories, forms, frontend API clients, and frontend package/config files.
- If the target has no frontend-relevant surface, report that and stop unless the user expands scope.

## Review Target

Accept any concrete review target: PR/MR URL, branch, commit range, pasted diff, changed file list, or local worktree changes.

When the target is local, discover the change set with git. Prefer a clear base if provided by the user; otherwise infer from the current branch and default branch. Gather:

- changed frontend files and their diffs
- directly related files, not the whole repository
- existing tests/stories/types/styles next to the changed files
- importing and imported modules for changed components/hooks/utils
- parent routes/components/providers/stores that exercise changed behavior
- form schemas, validation, constants, and API clients when touched by the change

Use `rg`/`rg --files` for relationship discovery. Exclude generated files, lockfiles, snapshots, build output, and vendored code unless the change directly targets them.

## Common Review Packet

Before dispatching subagents, prepare a compact packet they can share:

- review target and base/head or source of diff
- changed frontend files
- related files inspected and why they matter
- relevant commands or GitHub data used
- known constraints from the user
- the path to `references/toss-frontend-rubric.md`

Score the changed code and the related context needed to understand it. Do not punish unrelated historical debt unless the change depends on it or makes it worse; mention unrelated debt separately.

## Scoring

Read `references/toss-frontend-rubric.md` before dispatching or scoring.

- Each detailed rule receives a 0-100 score.
- A criterion score is the arithmetic mean of all detailed rule scores in that criterion.
- The overall score is the arithmetic mean of the four criterion scores.
- If a rule has no applicable surface in the reviewed change, score it as `100` and mark it `not_applicable: true` with a short reason.
- Require evidence for every score below 90, ideally with `file:line` or a diff hunk reference.
- Use the same score bands across agents:
  - `90-100`: strong, only tiny cleanup possible
  - `80-89`: good, minor local issues
  - `70-79`: workable, meaningful cleanup recommended
  - `60-69`: risky, likely to slow future changes
  - `<60`: high risk, fix before merge when possible

## Parallel Subagents

Dispatch four subagents in parallel, one per criterion. They are review-only and must not modify files.

Give each subagent the common review packet, the rubric path, and one of these scopes:

- `readability`: score the eight readability rules.
- `predictability`: score the three predictability rules.
- `cohesion`: score the three cohesion rules.
- `coupling`: score the three coupling rules.

Use this output contract for every subagent:

```markdown
## <Criterion> Review

Criterion score: <number>/100
Confidence: high|medium|low

| Rule | Score | Evidence | Main note |
| --- | ---: | --- | --- |

### Findings
- [P0-P3] `<rule-id>` `<file:line>`: issue, impact, suggested direction.

### Strengths
- Evidence-backed positives that explain high scores.

### Tradeoffs
- Criteria this finding may conflict with, if any.

### Context Inspected
- Files or diffs inspected beyond the changed files.
```

If a subagent cannot inspect enough context, it must lower confidence instead of inventing certainty.

## Synthesis

After all four subagents return:

1. Recalculate criterion and overall scores from the rule scores.
2. Merge duplicate findings and preserve the clearest evidence.
3. Separate changed-code issues from pre-existing unrelated debt.
4. Identify conflicts between criteria and recommend a direction.
5. Give the user explicit choices when a tradeoff is real.

Use this priority only as a default, not as a blind rule:

```text
readability > predictability > cohesion > coupling
```

Common conflicts to check:

- Readability vs cohesion: splitting large files may scatter code that changes together.
- Readability vs coupling: inlining may be clearer locally, while abstraction may reduce cross-file update risk.
- Predictability vs cohesion: project-wide conventions may compete with feature-local organization.
- Cohesion vs coupling: grouping everything together may create a module or hook with too many responsibilities.
- Coupling vs readability: allowing duplication can keep contexts independent, but too much duplication can hide shared business rules.

For each conflict, present:

- the conflicting findings and affected scores
- a recommended direction with rationale
- `Option A` and `Option B`, including what each option optimizes and what it sacrifices

## Final Review Format

Respond in the user's language. For Korean requests, write the review in Korean.

Always use this visible structure for the final synthesized review. Do not replace it with a shorter `Findings`/`Notes`-only format.

```markdown
**Overall**
총점: <overall-score>/100. <one-sentence verdict>

| 기준 | 점수 | 개선 필요 항목 |
| --- | ---: | --- |
| Readability | <score> | <items or `없음`> |
| Predictability | <score> | <items or `없음`> |
| Cohesion | <score> | <items or `없음`> |
| Coupling | <score> | <items or `없음`> |

**Top Findings**
1. [P0-P3] <finding title>
   - <evidence and impact>
   - <suggested direction>

**Criterion Summaries**
Readability는 ...
Predictability는 ...
Cohesion은 ...
Coupling은 ...

**Tradeoffs**
<Only include when there is a real tradeoff.>

**Verification**
<Commands/checks run and outcomes. Include skipped checks with the reason.>

**Suggested Next Steps**
<Only include actionable next steps when useful.>
```

For Korean scorecards, label the improvement-needed column exactly as `개선 필요 항목`. Populate it with the lowest-scoring applicable rules or concrete review points that most affected each criterion score, and avoid ambiguous labels that imply low rule importance.

Keep the review direct and evidence-heavy. Do not provide large rewritten code blocks unless the user asks for fixes.
