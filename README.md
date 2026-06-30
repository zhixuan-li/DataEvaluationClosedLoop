# Data and Evaluation Closed-Loop for Model Capability Enhancement

[![arXiv](https://img.shields.io/badge/arXiv-2606.28471-b31b1b.svg)](https://arxiv.org/pdf/2606.28471)

This repository accompanies the paper **Data and Evaluation Closed-Loop for Model Capability Enhancement**.

> Model capability is shaped by training data, but it is only observed through noisy benchmark scores. This work proposes a capability-centric data-evaluation loop that turns benchmark-level failures into structured capability diagnoses and then into targeted, experimentally validated data interventions.

**Paper:** [https://arxiv.org/pdf/2606.28471](https://arxiv.org/pdf/2606.28471)

## Overview

Large language model pre-training is often optimized through benchmark feedback: a model fails on a benchmark, and engineers must decide what data to add, remove, reweight, or repair. However, benchmark names are usually too coarse to explain what actually failed, while individual examples are too noisy to guide data decisions.

This work introduces the **capability slice** as an intermediate unit of analysis. A capability slice groups evaluation samples that share structured conditions along four dimensions:

- **Background condition**: what information the model must condition on.
- **Task type**: what task the model is asked to perform.
- **Solving operation**: what operations are needed to derive the answer.
- **Output constraint**: how the answer must be formatted and scored.

Around this unit, the paper builds a closed-loop toolkit:

```text
Benchmark-level failure
    -> capability-slice diagnosis
    -> data affordance profile
    -> data action
    -> controlled validation
    -> slice-level result analysis
```

The goal is to make data-driven model improvement more systematic, auditable, and experimentally testable.

## Key Contributions

### 1. Capability-slice view of evaluation

The paper argues that benchmark scores are noisy aggregates over prompts, decoding settings, scoring rules, and heterogeneous samples. Instead of treating a benchmark as a single capability probe, it decomposes evaluation samples into structured capability demands and aggregates model behavior over capability slices.

### 2. Evaluation sample taxonomy

The evaluation taxonomy annotates benchmark samples along four dimensions:

| Dimension | Purpose |
|---|---|
| Background condition | Describes the information environment required to solve the sample. |
| Task type | Identifies the primary user-facing objective. |
| Solving operation | Captures the operations needed to transform input into answer. |
| Output constraint | Captures answer form, scoring rule, rigidity, and exactness requirements. |

The taxonomy is applied to sixteen commonly used pre-training benchmarks, including PIQA, HellaSwag, MMLU, TriviaQA, RACE, DROP, MMLU-Pro, BBH, C-Eval, CMMLU, AGIEval-CN, MBPP, HumanEval, GSM8K, MathQA, and MATH.

### 3. Non-instruction data taxonomy

The data-side taxonomy characterizes raw corpus segments by the learnable signals they provide under language modeling:

| Dimension | Purpose |
|---|---|
| Content world | What knowledge, domains, entities, concepts, and symbolic systems are covered. |
| Discourse structure | How information is organized and connected across the text. |
| Operation opportunity | What reasoning or transformation patterns are exposed. |
| Supervision density | How clean, recoverable, informative, and learnable the signal is. |

This data-side view moves beyond domain-only corpus analysis and asks what capability-forming affordances the data actually supplies.

### 4. Evaluation-to-data mapping rules

The paper defines mapping rules that translate weak evaluation slices into candidate data affordance profiles. These rules are treated as structured hypothesis generators, not as causal guarantees. A resulting data intervention must still be tested under controlled experimental conditions.

### 5. Two controlled case studies

The framework is validated through two case studies that stress the loop in opposite directions.

#### Case Study 1: Output-constraint diagnosis reveals an EOS supervision bug

Continued pre-training causes BBH to drop by **-46.82%** relative to the warm-start checkpoint. A slice-level and prediction-level audit shows that the model often produces the correct answer but continues generating invalid suffixes. The root cause is traced to masked loss on document-boundary `<EOS>` tokens.

Restoring `<EOS>` supervision raises BBH from **25.14** to **66.44**, surpassing the warm-start checkpoint, while leaving other benchmarks largely unaffected or modestly improved.

#### Case Study 2: Solving-operation diagnosis guides targeted synthetic-data sampling

A mathematical weakness is decomposed by solving operations. The failure is not isolated arithmetic; it concentrates in operation combinations involving constraint tracking, boundary-case reasoning, symbolic transformation, counting, and comparison.

The diagnosis is converted into a weakness-targeted sampling procedure over synthetic instruction data. Under a fixed training recipe and token budget, the targeted checkpoint improves:

| Benchmark | Before | After |
|---|---:|---:|
| AIME2025 Pass@128 | 6.67 | 26.67 |
| AIME2026 Pass@128 | 0.00 | 26.67 |

General-domain performance remains close to unchanged, and operation-level re-evaluation confirms that the gains concentrate on the diagnosed slices.


## Why This Matters

This framework is designed for practical LLM pre-training workflows where engineers need to answer questions such as:

- A benchmark regressed. Is it a real capability regression or an output-format artifact?
- A model is weak at math. Does it need more arithmetic data, symbolic derivations, constraint-tracking examples, or better answer supervision?
- A data intervention improved a benchmark. Did it improve the intended capability slice or an unrelated part of the benchmark?
- Can data iteration be made reproducible instead of relying on benchmark-name intuition?

## Limitations

The paper treats mapping rules as structured heuristics rather than causal laws. The two case studies are run on a fixed set of benchmarks and a single continued pre-training pipeline. The instruction-data branch is validated experimentally, while the non-instruction data taxonomy is primarily used as a descriptive and mapping vocabulary in the reported experiments.