# research-novelties-and-architecture-comparisons

##### [**Undermind**](https://undermind.ai)

---


## Table of Contents

- [Research Compilation: Novelty Framework and Laya–Qwen Architecture Comparisons](#research-compilation-novelty-framework-and-layaqwen-architecture-comparisons)
  - [Purpose and framing](#purpose-and-framing)
- [Part A — Research novelties](#part-a-research-novelties)
  - [N1 — Joint routing under three kinds of uncertainty (primary novelty)](#n1-joint-routing-under-three-kinds-of-uncertainty-primary-novelty)
    - [Core idea](#core-idea)
    - [What makes it potentially novel](#what-makes-it-potentially-novel)
    - [Main hypothesis](#main-hypothesis)
    - [Primary evaluation](#primary-evaluation)
  - [N2 — Test-oracle-aware failure diagnosis and action selection](#n2-test-oracle-aware-failure-diagnosis-and-action-selection)
    - [Core idea](#core-idea-1)
    - [Novelty boundary](#novelty-boundary)
    - [Main hypothesis](#main-hypothesis-1)
    - [Primary evaluation](#primary-evaluation-1)
  - [N3 — Intent-preserving verification portfolio under budget](#n3-intent-preserving-verification-portfolio-under-budget)
    - [Core idea](#core-idea-2)
    - [Novelty boundary](#novelty-boundary-1)
    - [Main hypothesis](#main-hypothesis-2)
    - [Primary evaluation](#primary-evaluation-2)
  - [How the three novelties fit together](#how-the-three-novelties-fit-together)
- [Part B — Laya–Qwen architecture approaches to compare](#part-b-layaqwen-architecture-approaches-to-compare)
  - [Terminology](#terminology)
  - [Architecture A0 — One-shot Qwen baseline](#architecture-a0-one-shot-qwen-baseline)
  - [Architecture A1 — Qwen with a fixed test-and-repair loop](#architecture-a1-qwen-with-a-fixed-test-and-repair-loop)
  - [Architecture A2 — Model/tool router](#architecture-a2-modeltool-router)
  - [Architecture A3 — Laya-assisted decision component with Qwen as executor](#architecture-a3-laya-assisted-decision-component-with-qwen-as-executor)
  - [Architecture A4 — Generic adaptive controller without three-way uncertainty separation](#architecture-a4-generic-adaptive-controller-without-three-way-uncertainty-separation)
  - [Architecture A5 — Full hybrid three-uncertainty architecture (proposed)](#architecture-a5-full-hybrid-three-uncertainty-architecture-proposed)
  - [Architecture comparison matrix](#architecture-comparison-matrix)
- [Experimental comparison plan](#experimental-comparison-plan)
  - [Minimum architecture comparisons](#minimum-architecture-comparisons)
  - [Ablation experiments](#ablation-experiments)
  - [Task and benchmark design](#task-and-benchmark-design)
  - [Metrics](#metrics)
  - [Suggested hypotheses](#suggested-hypotheses)
  - [Go/no-go rule](#gono-go-rule)
- [Prior-work and novelty caution](#prior-work-and-novelty-caution)
  - [Immediate next steps](#immediate-next-steps)
  - [Existing literature workspace](#existing-literature-workspace)

# Research Compilation: Novelty Framework and Laya–Qwen Architecture Comparisons

## Purpose and framing

This document separates (A) the research novelties from (B) the implementation architectures to compare experimentally. The research goal is to test whether an adaptive code-generation workflow can make better verification and corrective-action decisions by separately representing uncertainty about user intent, implementation correctness, and test/specification reliability.

**Working title:** Intent-Aware Adaptive Verification and Corrective-Action Selection for Code-Generating Models.

**Central caution:** The novelty is not simply using Qwen, Laya, a router, multiple tests, or a hybrid system. The contribution must be demonstrated through matched-budget comparisons and evidence that the explicit uncertainty separation improves decisions and outcomes.

------------------------------------------------------------------------

# Part A — Research novelties

## N1 — Joint routing under three kinds of uncertainty (primary novelty)

**Research question:** Can a lightweight policy that separately tracks (1) uncertainty about what the user intends, (2) uncertainty about whether the implementation is correct, and (3) uncertainty about whether tests/specifications are valid, select the next check and intervention target better than generic adaptive controllers at the same budget?

### Core idea

The controller maintains a distinct state for: - **Intent uncertainty:** requirements are ambiguous, incomplete, or admit multiple plausible behaviors. - **Implementation uncertainty:** the generated code may not implement the expected behavior. - **Test/oracle uncertainty:** tests, expected outputs, or specifications may be defective, incomplete, or over-constrained.

Based on evidence and remaining budget, the controller selects an action: run a specific check, generate discriminating tests, patch implementation, inspect or revise tests/specifications, ask the user a clarifying question, gather more evidence, or stop/abstain.

### What makes it potentially novel

The strongest potential contribution is the **explicit separation of the three uncertainty sources coupled to joint action selection**. Merely passing all three as text to an LLM or routing between tools is not enough. The policy should make its uncertainty state and action rationale inspectable.

### Main hypothesis

At matched model/tool/latency budgets, the three-uncertainty policy achieves higher intended-behavior success and lower false acceptance than a generic cost-sensitive adaptive controller.

### Primary evaluation

- Intended-behavior satisfaction on an independently adjudicated subset.
- Hidden-test pass rate and false-acceptance rate.
- Success-versus-cost curve and cost per solved task.
- Calibration and quality of action decisions.

## N2 — Test-oracle-aware failure diagnosis and action selection

**Research question:** When a test fails, can the policy distinguish a wrong implementation, a faulty or over-constrained test, and a mis-specified or ambiguous requirement, then choose a repair target that improves intended-behavior correctness and avoids overfitting tests?

### Core idea

Use evidence to estimate whether a failure is most consistent with: 1. an implementation defect; 2. a test/oracle defect; 3. an ambiguous or inadequate specification; or 4. insufficient evidence to diagnose confidently.

The system then chooses a targeted action rather than automatically rewriting the code.

### Novelty boundary

Code-versus-test diagnosis and joint code/test repair already have prior work. N2 must therefore go beyond simple routing by demonstrating calibrated diagnosis, explicit uncertainty, better action selection at matched budgets, and/or fewer intent regressions.

### Main hypothesis

Compared with generic repair routing, the approach reduces wrong-target interventions and regression on previously satisfied behavioral constraints.

### Primary evaluation

- Failure-source diagnosis accuracy and macro-F1.
- Repair success conditioned on the true failure source.
- False test/specification repairs and false implementation repairs.
- Regression rate and intended-behavior satisfaction.

## N3 — Intent-preserving verification portfolio under budget

**Research question:** Can the system choose which tests and checks to run based on their expected information about the requested behavior, rather than following a fixed verification sequence?

### Core idea

Choose among checks such as: - existing unit tests; - targeted edge-case tests; - static analysis and type checks; - property-based tests, when feasible; - mutation-based tests, when feasible; - specification consistency checks.

Selection should account for expected information, relevance to the intended behavior, estimated cost, and oracle quality. The portfolio is adaptive: the next check depends on the evidence gathered so far.

### Novelty boundary

Adaptive test selection and cost-sensitive execution already exist. N3 is strongest when its selection policy accounts for intent ambiguity and oracle reliability and is evaluated against strong cost-sensitive controllers.

### Main hypothesis

At matched verification budgets, the adaptive portfolio improves success-versus-cost trade-offs and detects more behaviorally important defects per unit cost than fixed check ordering.

### Primary evaluation

- Defects detected per tool call or unit cost.
- Hidden-test pass rate and false acceptance.
- Check selection quality and diversity.
- Wall-clock time, tokens, tool calls, and cost per solved task.

## How the three novelties fit together

Treat them as one integrated framework, not three disconnected systems: - **N1 is the policy and primary conceptual contribution.** - **N2 is the failure-diagnosis and intervention mechanism.** - **N3 is the evidence-gathering/verification-selection mechanism.**

N3 selects checks; their results update the evidence state; N2 helps diagnose failures and target the intervention; N1 chooses the next action and decides whether to continue or stop. All modules feed the same loop.

If the results do not support three distinct contribution claims, present N1 as the primary contribution and N2/N3 as supporting mechanisms.

------------------------------------------------------------------------

# Part B — Laya–Qwen architecture approaches to compare

## Terminology

- **Qwen:** candidate code-generation, explanation, and repair model. Select the exact model only after checking local hardware/API availability and the current model’s coding performance.
- **Laya:** candidate decision component, only after inspecting its actual repository capabilities and interface. Do not assume that it automatically estimates the three uncertainties or is a ready-made router.
- **Router:** a mechanism that chooses a model/tool/action based on a rule, classifier, score, or policy.
- **Adaptive controller:** a component that observes evidence and budget state, selects the next action, and decides whether to continue or stop.
- **Hybrid architecture:** a modular system with Qwen for code work, verification tools for evidence, explicit uncertainty estimation, and a replaceable action controller that may use Laya.

## Architecture A0 — One-shot Qwen baseline

**Flow:** User requirement → Qwen → code returned.

**Purpose:** Establish a minimal reference point for code correctness and cost.

**Strengths:** Simplest and cheapest baseline.

**Limitations:** No adaptive verification, failure diagnosis, or corrective-action selection.

**Role in paper:** Essential lower-complexity baseline, not a serious adaptive competitor.

## Architecture A1 — Qwen with a fixed test-and-repair loop

**Flow:** Qwen generates code → run a predefined test sequence → send failures to Qwen → repeat a fixed number of times or until tests pass.

**Decision logic:** Fixed, not uncertainty-aware.

**Strengths:** Simple, reproducible, and often a strong baseline.

**Limitations:** May regenerate code unnecessarily, trust defective tests, miss ambiguity, or spend budget on low-value checks.

**Role in paper:** Main non-adaptive workflow baseline.

## Architecture A2 — Model/tool router

**Flow:** Request/evidence → router → choose among Qwen generation, Qwen repair, test runner, or another available tool.

**Decision logic:** Can be a simple rule table or lightweight classifier. It need not maintain three distinct uncertainty states.

**Strengths:** Low overhead; tests whether action routing alone explains gains.

**Limitations:** A router based only on task type or last test result may not reason about the source of uncertainty, oracle reliability, or the value of another check.

**Role in paper:** Key baseline for the claim that explicit uncertainty representation matters beyond routing.

## Architecture A3 — Laya-assisted decision component with Qwen as executor

**Flow:** Evidence/state → Laya-based decision logic → action selection → Qwen or verification tool executes the action → evidence returned to controller.

**Decision logic:** Laya is used only for the subset of decision functions its verified API supports. State features and action choices must be explicitly defined.

**Strengths:** Tests whether Laya adds value as a decision component while Qwen remains the code engine.

**Limitations:** May not natively support uncertainty estimation, budget-aware sequential decisions, or all desired action types. Integration overhead and interface constraints may be material.

**Role in paper:** Component-level comparison. Do not call it an uncertainty-aware policy unless it actually implements and exposes that behavior.

## Architecture A4 — Generic adaptive controller without three-way uncertainty separation

**Flow:** Evidence + remaining budget → generic confidence/cost-aware policy → select check, repair, gather evidence, or stop → update evidence.

**Decision logic:** Adaptive, but uses a single correctness confidence or generic evidence state rather than separately modeling intent, implementation, and oracle uncertainty.

**Strengths:** Strong and scientifically important baseline; controls for the benefit of adaptivity itself.

**Limitations:** May confuse ambiguous intent with implementation defects or treat faulty tests as reliable evidence.

**Role in paper:** Most important conceptual comparator for N1. Where feasible, reproduce a strong cost-sensitive/Bayesian controller from prior work rather than relying only on a home-grown weak baseline.

## Architecture A5 — Full hybrid three-uncertainty architecture (proposed)

**Flow:** Requirement analysis → Qwen generates code → adaptive verification portfolio → evidence/uncertainty estimator tracks intent, implementation, and oracle/test uncertainty separately → diagnosis and corrective-action policy → Laya-assisted or replaceable controller chooses the next action → loop until stopping/budget condition.

**Decision logic:** Joint selection of verification action, intervention target, and stop/continue decision.

**Strengths:** Modular, inspectable, ablatable, and directly aligned with the research question.

**Limitations:** More engineering; uncertainty estimates require validation; improvements must be shown at matched budgets.

**Role in paper:** Proposed system.

## Architecture comparison matrix

| Architecture | Qwen code generation/repair | Adaptive routing | Three separate uncertainties | Chooses verification checks | Diagnosis/repair targeting | Main purpose |
|:---|:---|:---|:---|:---|:---|:---|
| A0 One-shot Qwen | Yes | No | No | No | No | Minimal baseline |
| A1 Fixed test/repair loop | Yes | No (fixed sequence) | No | Fixed sequence | Basic failure-driven repair | Non-adaptive baseline |
| A2 Simple router | Yes | Yes, simple | Not required | Possibly | Usually shallow | Test routing alone |
| A3 Laya-assisted controller | Yes | Depends on Laya integration | Only if implemented | Depends on policy | Depends on policy | Test Laya’s contribution |
| A4 Generic adaptive controller | Yes | Yes | No explicit three-way separation | Adaptive | Generic target choice | Control for adaptivity |
| A5 Proposed hybrid | Yes | Yes | Yes | Adaptive portfolio | Explicit source-aware targeting | Main proposed approach |

------------------------------------------------------------------------

# Experimental comparison plan

## Minimum architecture comparisons

For a manageable 3–6 month project, prioritize: 1. A0: one-shot Qwen. 2. A1: fixed test-and-repair loop. 3. A2: simple model/tool router. 4. A4: generic adaptive controller without three-way uncertainty separation. 5. A5: proposed hybrid.

Add A3 as a separate component experiment if Laya can be integrated cleanly. Do not spend the whole project making Laya work if its interface is unsuitable; a replaceable controller keeps the research testable.

## Ablation experiments

Compare the full hybrid against variants: - remove intent uncertainty; - remove implementation uncertainty as a separate signal; - remove test/oracle uncertainty; - replace adaptive verification selection with a fixed sequence; - disable failure-source diagnosis; - disable targeted test/specification repair; - remove budget awareness; - replace the Laya-assisted policy with simple rules or another controller.

These ablations test which mechanisms cause improvement rather than merely whether the full system works.

## Task and benchmark design

Start with function-level Python code generation to keep the infrastructure affordable. Use a standard benchmark such as HumanEval+ or MBPP+ for baseline correctness, then add a small, manually audited extension with controlled conditions: - ambiguous or incomplete requirements; - implementation bugs; - faulty, incomplete, or over-constrained tests/oracles; - combinations of the above.

A standard coding benchmark alone is not enough to validate intent and oracle uncertainty. The controlled extension is central to testing those claims.

## Metrics

**Outcome metrics** - hidden-test pass rate; - independently adjudicated intended-behavior satisfaction; - false acceptance rate; - diagnosis accuracy; - repair success; - regression rate on previously passing behaviors.

**Efficiency metrics** - model calls and tokens; - verification/tool calls; - wall-clock latency; - monetary or compute cost; - cost per successful task; - success-versus-cost curve.

**Policy metrics** - action selection quality; - calibration of uncertainty estimates; - unnecessary repair and unnecessary verification rates; - stopping/abstention quality.

Use equal or carefully matched budgets, held-out task families, repeated runs where stochasticity applies, confidence intervals, and hidden tests. Passing visible tests must not be treated as proof of intended-behavior correctness.

## Suggested hypotheses

**H1 — Primary:** At matched budgets, A5 improves intended-behavior satisfaction and reduces false acceptance relative to A4, the generic adaptive controller.

**H2 — Diagnosis:** N2 improves failure-source diagnosis and reduces wrong-target repairs and regressions.

**H3 — Verification:** N3 improves the success-versus-cost trade-off over fixed verification sequences.

**H4 — Interaction:** Benefits are largest when requirements are ambiguous or tests/oracles are defective or incomplete.

**H5 — Component attribution:** Ablating any uncertainty source reduces performance on the corresponding task condition.

## Go/no-go rule

Run a pilot on roughly 50–100 tasks before scaling. Continue with the full study only if the proposed system improves intended-behavior success, false acceptance, or the success-versus-cost curve over the strongest practical baseline. If gains appear only against one-shot or fixed-loop baselines, narrow the novelty claim.

------------------------------------------------------------------------

# Prior-work and novelty caution

The preliminary literature synthesis found direct overlap with: - cost-sensitive/Bayesian control for coding agents; - budget-calibrated recovery routing; - adaptive routing for ambiguous requirements; - policy-controlled evidence selection; - behavior/specification alignment; - bidirectional code-test diagnosis and code/test co-evolution; - cost-effectiveness of code execution in program repair.

Therefore, do not claim that adaptive routing, ambiguity-aware clarification, code-versus-test diagnosis, or budget-aware test selection is individually new. The candidate gap is the explicit separation of the three uncertainty sources coupled to joint verification, intervention-target, and stopping decisions, evaluated against strong matched-budget controllers.

This gap remains a hypothesis until the full texts of the closest studies are checked and the experimental distinction is demonstrated.

## Immediate next steps

1.  Inspect Laya’s repository and establish what it actually supports.
2.  Pick one Qwen coding model based on the user’s compute constraints.
3.  Implement A0 and A1 first.
4.  Implement A2 and a simple A4 controller.
5.  Add the three-way uncertainty state, N2 diagnosis, and N3 verification selection incrementally.
6.  Pilot, ablate, and revise the novelty claims based on results.

## Existing literature workspace

The preliminary novelty shortlist and close-work list are maintained in the Undermind research workspace. Treat it as a working literature map rather than a final proof of priority.
