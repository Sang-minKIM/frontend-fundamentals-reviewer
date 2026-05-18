# Toss Frontend Fundamentals Review Rubric

Use this rubric for `$frontend-fundamentals-reviewer`. Score only the reviewed change set and directly related frontend context.

## General Scoring Rules

- Start each detailed rule at 100 and subtract for concrete risk or friction.
- Prefer evidence from changed lines, direct call sites, nearby tests, and directly related modules.
- For old debt unrelated to the change, mention it separately and do not subtract unless the change depends on it or makes it worse.
- If there is no applicable surface for a rule, score 100 and mark it `not_applicable: true`.
- Scores below 90 need evidence. Scores below 75 should include a suggested direction.

## Readability

Criterion score: equal-weight average of the eight rule scores.

| Rule ID | Rule | What to Evaluate |
| --- | --- | --- |
| `readability-context-separate-code-paths` | Separate code paths that do not run together | Penalize scattered branches, role/state/mode checks repeated across effects and render paths, and components that force readers to simulate mutually exclusive cases at once. Reward extracting separate components/functions per case while keeping the top-level decision easy to see. |
| `readability-context-abstract-implementation` | Abstract implementation details | Penalize low-level filtering/sorting/parsing/formatting details embedded where the reader needs business intent. Reward names that reveal "what" the code does and helper functions with consistent abstraction level. Do not reward vague wrappers that hide important domain behavior. |
| `readability-context-split-by-logic-type` | Split functions by logic type | Penalize functions/hooks/components that mix fetching, derivation, validation, rendering, state orchestration, logging, and side effects in one hard-to-test unit. Reward separating logic by type while preserving a readable top-level flow. |
| `readability-naming-complex-conditions` | Name complex conditions | Penalize multi-clause boolean expressions with unclear intent, especially business rules and repeated conditions. Reward named booleans/functions using `is`, `has`, or `should`, preferably positive wording. Simple obvious conditions should not be penalized. |
| `readability-naming-magic-numbers` | Name magic numbers | Penalize unexplained numeric/string thresholds, delays, sizes, retry counts, timeouts, and limits in code. Reward named constants with units such as `_MS`, `_PX`, `_COUNT`, or domain-specific names. Do not penalize obvious `0`, `1`, simple indexes, or conventional values when context is clear. |
| `readability-flow-reduce-time-travel` | Reduce time travel | Penalize declarations far from use, initialization that requires jumping around the file, callbacks defined far from the UI they affect, and top-level flow that forces repeated scrolling. Reward placing derived values near dependencies and arranging code top-to-bottom by reading order. |
| `readability-flow-simplify-ternary` | Keep ternaries simple | Penalize nested ternaries, mixed boolean/operator precedence, and JSX branches that are easier as early returns or named functions. Reward simple binary ternaries and explicit `if`/early-return flows for multi-branch logic. |
| `readability-flow-left-to-right` | Read left to right | Penalize Yoda conditions, deeply nested expressions, excessive chains, negative-first control flow when the positive path is primary, and code where the main intent is visually hidden at the far right. Reward positive primary paths, early returns, and meaningful intermediate values. |

## Predictability

Criterion score: equal-weight average of the three rule scores.

| Rule ID | Rule | What to Evaluate |
| --- | --- | --- |
| `predictability-unique-names` | Avoid overlapping names | Penalize same names for different concepts, different names for the same concept, ambiguous imports, and role-specific functions without role-specific names. Reward project-consistent prefixes such as `fetch*`, `use*`, `format*`, `parse*`, `handle*`, `is*`, `has*`, and `should*`. |
| `predictability-consistent-return-types` | Keep similar return types consistent | Penalize related hooks/API helpers/utils that return incompatible shapes, mix throw/null/result-object patterns unexpectedly, or vary loading/error/data names without reason. Reward shared generic response types and consistent hook return conventions. |
| `predictability-expose-hidden-logic` | Expose hidden logic | Penalize functions/components/hooks whose names, parameters, or return values hide side effects such as analytics, navigation, storage writes, mutation, cache invalidation, global state updates, or network calls. Reward explicit call sites, names that reveal side effects, and APIs that make behavior visible. |

## Cohesion

Criterion score: equal-weight average of the three rule scores.

| Rule ID | Rule | What to Evaluate |
| --- | --- | --- |
| `cohesion-colocate-modified-files` | Colocate files changed together | Penalize feature changes that require hopping across type-based directories without a strong shared-layer reason. Reward feature/component directories that keep component, hook, type, style, test, and story files near the code they change with. Shared code belongs in shared locations only when it is genuinely reused across features. |
| `cohesion-eliminate-magic-numbers` | Place constants near their use | Penalize dumping local constants into global constants files, importing feature-local values from distant shared modules, or scattering the same domain threshold across unrelated places. Reward constants placed at file, component, feature, or shared scope according to actual use range. |
| `cohesion-form-structure` | Keep form structure cohesive | Penalize form state, schema, validation, error messages, default values, transformation, and submit behavior being split so field changes require many edits. Reward cohesive form hooks/modules, schema-driven validation, and colocated field-specific rules. Mark not applicable when no form or form-adjacent behavior is involved. |

## Coupling

Criterion score: equal-weight average of the three rule scores.

| Rule ID | Rule | What to Evaluate |
| --- | --- | --- |
| `coupling-single-responsibility` | Manage one responsibility per unit | Penalize broad hooks/components/utils that force consumers to subscribe to or import more than they need, especially shared state hooks, providers, and large components. Reward smaller responsibility-focused units and APIs that expose only the relevant state or behavior. |
| `coupling-allow-duplication` | Allow duplication when contexts differ | Penalize premature abstractions that introduce variants, flags, conditional behavior, or shared modules for code that is likely to evolve separately. Reward keeping similar code separate when change reasons differ. Also penalize duplicated true business rules that must stay synchronized. |
| `coupling-eliminate-props-drilling` | Remove props drilling | Penalize passing props through layers that do not use them, especially identity/session/theme/form state routed through layout-only components. Reward composition, colocated state, context, scoped stores, or provider boundaries when they reduce unnecessary dependency chains. Mark not applicable when component hierarchy or prop passing is not involved. |

## Tradeoff Priority

Default priority:

```text
readability > predictability > cohesion > coupling
```

Use this as a starting point. Override it when the evidence shows a different change risk:

- Prefer cohesion when the same files always change together and splitting would hide the workflow.
- Prefer coupling reduction when a shared abstraction is already accumulating variants or triggering unrelated re-renders.
- Prefer predictability when inconsistent naming or return shapes would cause incorrect API usage.
- Prefer readability when future maintainers cannot understand the change without simulating multiple branches or hidden effects.
