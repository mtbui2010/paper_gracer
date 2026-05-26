# GRACE — Failure case analysis

Hand-labelled categorization of GRACE failures on the 38-task AI2-THOR
benchmark. Fill this in during Phase 4.1 of the submission plan. Aim
for 8–10 distinct failure cases that together justify the
"Limitations" subsection of the paper and the "Future work" claims.

## Method

1. Identify candidates with the helper command in the README — tasks
   where GRACE has `precondition < 0.7` or `completeness < 0.5` (run on
   `qwen2.5:7b` so the LLM is strong enough that residual failures are
   informative, not noise).
2. For each task, open the GT plan in
   [`../../pyplanner/eval_dataset_gt.json`](../../pyplanner/eval_dataset_gt.json)
   and compare to GRACE's actual output.
3. Classify into one of the three categories below. If a fourth
   category emerges, add it to the taxonomy and note it in
   `experiments.tex` §5.

## Taxonomy

### C1. Object naming ambiguity
LLM names the right thing but uses a synonym the verifier / executor
cannot match (e.g.\ `Cup` vs.\ `Mug`, `LightBulb` vs.\ `LightSwitch`).
This is **orthogonal** to the planning method — would benefit from a
lexical mapping layer. Count toward Limitations, not Method.

### C2. Mis-decomposition
The high-level decompose step produces sub-goals that omit a required
phase (e.g.\ "brew coffee" without "acquire mug"). The verifier can't
catch this — it has no semantic goal model — and local repair inherits
the omission. Suggests future work on a goal-aware decomposer.

### C3. Sub-goal-boundary mismatch
The failed step is actually serving a future sub-goal, so local repair
patches the wrong block. Detect via the offset between
`failed_step.index` and the cumulative sub-goal length; future work
would escalate to multi-block repair.

### C4. Other (add new categories here)

## Case template (one per failure)

Copy this block 8–10 times below.

```
### Task <task_id> (<difficulty>, <room>)
Goal: "<task_desc verbatim>"
Category: C1 | C2 | C3 | C4
Reference plan: <N steps, summarized>
GRACE plan:     <N steps, summarized — pre-flatten if multi-subgoal>
What went wrong:
  - <2-3 sentences pinpointing the precise step that failed and why>
What would fix it:
  - <2-3 sentences; reference a future-work item if applicable>
```

---

## Cases

<!-- Populate during Phase 4.1. Examples below; replace with real cases. -->

### Task K07 (medium, kitchen) — EXAMPLE
Goal: "Heat food in the microwave"
Category: C2
Reference plan: MoveTo Microwave → Open Microwave → Find Bread → Pick → MoveTo Microwave → Open → Place Microwave → Close → TurnOn (9 steps)
GRACE plan: MoveTo Microwave → Open → TurnOn (3 steps; "load food" sub-goal missing)
What went wrong:
  - High-level decompose collapsed "load food + start microwave" into a single sub-goal "use the microwave", which the low-level expander interpreted as just opening and turning on, skipping the food entirely.
What would fix it:
  - Goal-aware decomposer that includes `expected_objects` ("Bread") in the high-level prompt, forcing each interactable to appear in at least one sub-goal.

### Task A03 (easy, bathroom) — EXAMPLE
Goal: "Brush teeth at the sink"
Category: C1
Reference plan: MoveTo Sink → Find ToothBrush → Pick → TurnOn Faucet → Wash ToothBrush (5 steps)
GRACE plan: MoveTo Sink → Find Toothbrush → Pick → TurnOn Tap → Wash Toothbrush (5 steps; objects miscased)
What went wrong:
  - GRACE emitted `Toothbrush` (lower-case middle "b"); executor expects `ToothBrush`. Similar for `Tap` vs.\ `Faucet`. Plan is otherwise correct.
What would fix it:
  - Normalise object names against the scene's visible-object list with a small canonicalisation table; LLM proposal then projected onto the closest canonical name.
