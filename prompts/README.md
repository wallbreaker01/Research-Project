# LLM prompts (replication package)

These Markdown files are the **canonical text** of the prompts used in the notebooks.  
Manuscript: `The_Name_of_the_Title_Is_Hope__1_ (4).pdf` (ReviewerMatch).

## Quick map: file → purpose

| File | What it does |
| ---- | ------------ |
| [P1](P1_skill_extraction_prompt.md) | Read a developer’s PR history and describe their JavaScript skills (unstructured text). |
| [P2](P2_structured_profile_prompt.md) | Turn that analysis into a **structured profile** (experience, summary, skill matrix). Includes category/frequency **consistency rules** in the prompt body. |
| [P3](P3_pr_requirement_extraction_prompt.md) | Analyze a **target PR** and output requirement fields + JSON (paper §3.4 labels this **P3**). |
| [P4](P4_reviewer_matching_prompt.md) | Rank **top 3 reviewers** — full system: **KU matrix + frequency** (`LLM+KU+freq`). Paper §3.4 **P4**. |
| [P5](P5_llm_only_reviewer_matching_prompt.md) | Same ranking task — **ablation**: free-form `identified_skills`, **no KU, no frequency** (`LLM/`). |
| [P6](P6_llm_ku_binary_reviewer_matching_prompt.md) | Same ranking task — **ablation**: **KU matrix, binary only** (present/absent), **no frequency** (`LLM+KU/`). |
| [P7](P7_failure_analysis_prompt.md) | Optional: explain missed hits during evaluation (not used in live assignment). |

**Paper vs folder numbering:** In §3.4, the manuscript calls PR analysis **(P3)** and main matching **(P4)**. In this folder, those are files **P3** and **P4**; **P5** and **P6** are extra matching prompts for §4.4 ablations only.

## Which notebook uses which matching prompt?

All variants share the same **P3** PR-analysis prompt in `TopThreeReviewerList.ipynb`.  
Only the **matching** prompt and **profile** format differ:

| Experiment folder | Profile notebooks | Matching prompt file |
| ----------------- | ----------------- | -------------------- |
| `newAnalysis/LLM/` | `developer_profile.ipynb`, `Reviewer_profile.ipynb` | **P5** |
| `newAnalysis/LLM+KU/` | same names | **P6** |
| `newAnalysis/LLM+KU+freq/` | same names (frequency in matrix) | **P4** |

Example path (axios):  
`newAnalysis/LLM+KU+freq/axios/PrecisionMetricsCalculation/TopThreeReviewerList.ipynb`

## Model settings

| Stage | Model | Temperature |
| ----- | ----- | ----------- |
| P1, P2 (profiles) | `mistralai/ministral-8b-2512` via OpenRouter | **0.3** in notebooks |
| P3–P6 (assignment) | same | **0.1** |

## End-to-end flow

```text
P1 → P2  (build reviewer/developer profiles offline)
        ↓
P3       (when a new PR arrives: extract requirements)
        ↓
FAISS    (similar historical PRs; all variants)
        ↓
P4 or P5 or P6  (rank top 3 reviewers, depending on experiment arm)
```
