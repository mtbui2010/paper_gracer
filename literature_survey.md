# GRACE — Related Work Survey & Positioning

> **Sourcing note.** The literature survey was conducted by an LLM agent with knowledge cutoff January 2026 but without live web access. URLs point to canonical arXiv / project pages but should be re-verified on arXiv before camera-ready. A fresh search for the strings "symbolic verifier LLM planner household 2025/2026" and "hierarchical plan repair LLM 2025/2026" is recommended before submission.

This document collects the 25+ most relevant prior works and ends with an explicit **Positioning** section spelling out how GRACE differs from its closest neighbors. Groups A–E mirror the structure of the Related Work section in the paper outline.

---

## Group A — LLM-as-planner foundations

**SayCan (Ahn et al., Google, 2022)** — Pairs an LLM that scores task-relevance of natural-language skills with a learned value function ("affordance") that scores feasibility; their product picks the next skill. Established the canonical "LLM proposes, world grounds" pattern but uses a *learned* affordance, not symbolic preconditions, and replans only globally.
URL: https://arxiv.org/abs/2204.01691

**Inner Monologue (Huang et al., Google, 2022)** — Feeds back textual scene/success descriptions to the LLM after each step, letting the model self-correct in closed loop. Introduces feedback-driven replanning but the verifier is the LLM itself, not a deterministic checker.
URL: https://arxiv.org/abs/2207.05608

**ProgPrompt (Singh et al., NVIDIA, 2022)** — Prompts the LLM with Pythonic function signatures and assertions describing available actions/objects so it emits executable programs. Uses `assert` statements as inline preconditions but the assertions are LLM-generated, not externally enforced.
URL: https://arxiv.org/abs/2209.11302

**Code as Policies (Liang et al., Google, 2022)** — LLM writes Python code (loops, control flow, calls into perception APIs) that becomes the robot policy. Powerful expressiveness, but no separation between plan generation and a symbolic check.
URL: https://arxiv.org/abs/2209.07753

**LLM+P (Liu et al., UT Austin, 2023)** — Translates a natural-language problem into PDDL, hands it to a classical planner (Fast Downward), then translates the plan back. Provides correctness guarantees via the classical planner but is brittle to PDDL translation errors and does no incremental repair.
URL: https://arxiv.org/abs/2304.11477

**LLM-DP (Dagan, Keller & Lascarides, 2023)** — Combines an LLM with a symbolic planner over a maintained world model for the Alfworld environment, replanning when state diverges. Closest classical-planner hybrid; differs from GRACE because LLM-DP fully delegates planning to PDDL rather than using the symbolic layer as a *verifier* of LLM plans.
URL: https://arxiv.org/abs/2308.06039

---

## Group B — Reasoning / refinement techniques

**Chain-of-Thought (Wei et al., 2022)** — Prompting LLMs to emit intermediate reasoning steps before the answer improves multi-step problem solving.
URL: https://arxiv.org/abs/2201.11903

**ReAct (Yao et al., 2022)** — Interleaves "thought" and "action" tokens so the LLM reasons and queries the environment in a tight loop; the dominant prompting pattern for agentic LLMs and the backbone of pyplanner's `ReAct` strategy.
URL: https://arxiv.org/abs/2210.03629

**Reflexion (Shinn et al., 2023)** — After each failed trial the agent writes a verbal "reflection" into memory and retries; improves ALFWorld and HumanEval. Reflection is unstructured text — GRACE instead reasons over typed precondition violations.
URL: https://arxiv.org/abs/2303.11366

**Self-Refine (Madaan et al., 2023)** — Same LLM iteratively critiques and rewrites its own output without external signal. GRACE differs by using a *non-LLM* symbolic checker as the critic, removing the well-known LLM self-evaluation bias.
URL: https://arxiv.org/abs/2303.17651

**Tree of Thoughts (Yao et al., 2023)** — Explores a tree of reasoning branches with explicit backtracking and value estimation. Expensive at robot-runtime; GRACE keeps a single trajectory and only branches on verifier rejection.
URL: https://arxiv.org/abs/2305.10601

**ExpeL (Zhao et al., AAAI 2024)** — Distills natural-language "insights" from a corpus of successful and failed trajectories and retrieves them in future episodes. Directly inspires GRACE's Chroma-backed memory; GRACE differs by also seeding the store with curated ground-truth plans rather than only mining online traces.
URL: https://arxiv.org/abs/2308.10144

**AutoGuide (Fu et al., 2024)** — Generates *state-conditioned* guidelines from past episodes and retrieves only the guidelines whose triggering state matches the current one. More elaborate state-matching than GRACE's plain embedding retrieval.
URL: https://arxiv.org/abs/2403.08978

**Voyager (Wang et al., NVIDIA, 2023)** — Lifelong Minecraft agent that grows a *skill library* of verified Python skills, retrieved by embedding. Same memory-of-successes idea as GRACE, but stores reusable *code skills* rather than full step-level plans, and has no symbolic precondition layer.
URL: https://arxiv.org/abs/2305.16291

---

## Group C — Embodied / grounded planning with verification

**LLM-Planner (Song et al., ICCV 2023)** — Few-shot LLM planner for ALFRED that retrieves similar in-context examples by goal embedding and replans when the agent stalls. Most direct prior art for GRACE's retrieval-augmented planning; GRACE adds (a) symbolic precondition gating *before* execution and (b) suffix-only repair localized to a sub-goal.
URL: https://arxiv.org/abs/2212.04088

**TidyBot (Wu et al., 2023)** — Personalizes object placement by summarizing user preferences with an LLM and uses standard pick-and-place primitives. Memory is preference-centric, not plan-centric — orthogonal to GRACE.
URL: https://arxiv.org/abs/2305.05658

**DEPS — Describe, Explain, Plan, Select (Wang et al., NeurIPS 2023)** — Open-Minecraft agent that describes execution failures in natural language, explains them, and re-plans; introduces a "selector" that picks the closest sub-goal. Conceptually similar to GRACE's suffix-replan, but the failure signal is LLM-generated text rather than typed precondition violations.
URL: https://arxiv.org/abs/2302.01560

**AdaPlanner (Sun et al., NeurIPS 2023)** — Adaptive in-context closed-loop planner that distinguishes "in-plan" refinement from "out-of-plan" replanning, and writes successful plans to a skill memory. Strong baseline for GRACE; differences are (1) GRACE's verifier is symbolic, not the LLM, and (2) GRACE's hierarchical repair is bound to sub-goal boundaries from the task ontology.
URL: https://arxiv.org/abs/2305.16653

**ISR-LLM (Zhou et al., ICRA 2024)** — Iteratively self-refines an LLM-generated PDDL plan against a validator. Closest in spirit to GRACE's symbolic gating, but ISR-LLM validates against PDDL produced by the *same* LLM rather than a hand-written verifier over a fixed action vocabulary.
URL: https://arxiv.org/abs/2308.13724

**SwiftSage (Lin et al., NeurIPS 2023)** — Splits fast (LLM) and slow (deliberative) modules in ScienceWorld. Architectural analogue of GRACE's two-stage approach but uses an LLM, not symbolic rules, in the slow path.
URL: https://arxiv.org/abs/2305.17390

**Guan et al. — "LLMs to Construct and Utilize World Models" (NeurIPS 2023)** — Uses an LLM to author PDDL operators, then validates plans with a classical planner — opposite direction from GRACE (LLM authors *symbols*; classical planner produces plan).
URL: https://arxiv.org/abs/2305.14909

**Kambhampati et al. — "LLM-Modulo" (ICML 2024)** — Explicitly argues for an LLM-generate + external-verifier-critique loop. GRACE is essentially an instance of LLM-Modulo specialized to AI2-THOR with a lightweight rule-based verifier instead of a classical planner.
URL: https://arxiv.org/abs/2402.01817

**Embodied-CoT (Zawalski et al., 2024)** — Trains a VLA to emit chains of thought tied to visual state. Operates at the perception-to-action layer; not directly comparable to GRACE's plan-level gating.
URL: https://arxiv.org/abs/2407.08693

---

## Group D — 2024-2026 plan repair / replanning

**ReplanVLM (Mei et al., RA-L 2024)** — Detects execution errors with a VLM and triggers replanning. GRACE replaces the VLM critic with a deterministic precondition check and replans only the local suffix.
URL: https://arxiv.org/abs/2407.21762

**RAP — Reasoning via Planning (Hao et al., EMNLP 2023)** — Uses an LLM as a world model with MCTS-style search; gives principled lookahead but is expensive at robot runtime. GRACE uses far cheaper one-shot generation plus verifier rejection rather than search.
URL: https://arxiv.org/abs/2305.14992

**Reflective Planning (Liu et al., 2024)** — Trains/reuses a reflection model that critiques candidate plans before commit; lives in the same "verify-before-execute" family as GRACE but the verifier is again learned, not symbolic.
URL: https://arxiv.org/abs/2410.16736

**HiP — Hierarchical Planning (Ajay et al., NeurIPS 2023)** — Decomposes into sub-goals but typically replans the whole sub-tree on failure; GRACE retains decomposition into execution and replans only the failed sub-goal's suffix.
URL: https://arxiv.org/abs/2309.08587

**No closely matching combination found.** In the surveyed literature (cutoff Jan 2026), I did not find a single paper that combines all three of GRACE's pillars (*rule-based LLM-free verifier* + *sub-goal-localised suffix repair* + *hybrid seeded/accumulated retrieval memory*) in one embodied household system. Each pillar individually has clear precedents, so GRACE's contribution is best framed as a **principled integration tailored to AI2-THOR**, with the lightweight symbolic verifier as the load-bearing engineering claim. A late-2025 / early-2026 search before submission is recommended to confirm.

---

## Group E — Benchmarks

**ALFWorld (Shridhar et al., 2021)** — TextWorld-style abstraction of ALFRED household tasks. The de-facto benchmark for LLM agents.
URL: https://arxiv.org/abs/2010.03768

**AI2-THOR / RoboTHOR (Kolve et al., 2017; Deitke et al., CVPR 2020)** — Photo-realistic interactive 3D household and navigation simulator; GRACE's execution backend (`ThorClient`) runs here.
URLs: https://arxiv.org/abs/1712.05474, https://arxiv.org/abs/2004.06799

**BEHAVIOR-1K (Li et al., CoRL 2023)** — 1,000 long-horizon household activities in OmniGibson; richer physics than AI2-THOR but heavier.
URL: https://arxiv.org/abs/2403.09227

**VirtualHome (Puig et al., CVPR 2018)** — Household program-style activities; used by ProgPrompt and many LLM-planner papers.
URL: https://arxiv.org/abs/1806.07011

**ScienceWorld (Wang et al., EMNLP 2022)** — Text-based science experiments requiring multi-step planning; used by SwiftSage / ExpeL.
URL: https://arxiv.org/abs/2203.07540

---

## Positioning — closest 3-4 papers and GRACE's differentiation

1. **AdaPlanner (Sun et al., NeurIPS 2023)** — closest in overall *shape*: closed-loop LLM planner with in-plan vs. out-of-plan refinement and a skill memory.
   *GRACE differs* by making the in-plan/out-of-plan decision with a **rule-based symbolic verifier over action preconditions** rather than letting the LLM self-judge, and by restricting "out-of-plan" repair to the **suffix of the current sub-goal** instead of the whole remainder.

2. **LLM-Planner (Song et al., ICCV 2023)** — closest *memory* design: retrieval-augmented few-shot planning for embodied tasks.
   *GRACE differs* by **seeding** the store with curated ground-truth plans (cold-start coverage) and continuously **accumulating successful episodes** keyed on task label, and by separating the verifier from the LLM.

3. **LLM-Modulo / Guan et al. (ICML & NeurIPS 2023-2024)** — closest *philosophy*: LLM-generate + external-verifier-critique.
   *GRACE differs* by using a **lightweight, hand-written rule verifier** over a small action-primitive set rather than a full classical planner over LLM-authored PDDL — orders of magnitude cheaper at runtime, but only sound within the task ontology.

4. **DEPS / Reflexion** — closest *repair* idea: detect failure, describe it, replan.
   *GRACE differs* by emitting **typed precondition violations** rather than free-form natural-language explanations, and by **scoping the replan to the current sub-goal** (e.g., only re-plan "acquire the mug", not "make coffee").

### Honest novelty assessment

In my survey (knowledge cutoff Jan 2026, no live web access) I did not find a single paper that combines all three of GRACE's pillars. The contribution should therefore be framed as a principled integration with the rule-based, LLM-free verifier as the chief engineering claim — emphasizing that the verifier costs zero tokens and runs in $O(|\pi|)$ time, in contrast to LLM critics (Self-Refine, Reflexion) or learned validators (Reflective Planning). A search before submission for late-2025 and early-2026 work on "symbolic verifier LLM planner" is recommended to confirm no immediate prior art has appeared.
