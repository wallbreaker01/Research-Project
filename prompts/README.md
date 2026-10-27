# LLM Prompts

One file per paper prompt (**P1–P4**). Full manuscript: `The_Name_of_the_Title_Is_Hope__1_ (4).pdf` (ReviewerMatch). Implementation: `newAnalysis/LLM+KU+freq/axios/`.

## Paper ↔ replication files

| Paper | Section | File |
|-------|---------|------|
| **P1** | §3.3 skill-analysis prompt | [P1_skill_extraction_prompt.md](P1_skill_extraction_prompt.md) |
| **P2** | §3.3 structured expertise profile | [P2_structured_profile_prompt.md](P2_structured_profile_prompt.md) |
| **P3** | §3.4 requirement-extraction (+ JSON) | [P3_pr_requirement_extraction_prompt.md](P3_pr_requirement_extraction_prompt.md) |
| **P4** | §3.4 reviewer recommendation (+ JSON) | [P4_reviewer_matching_prompt.md](P4_reviewer_matching_prompt.md) |

**Also in the paper (not P1–P4 files):**

- **P6** — §4.4 ablation *LLM + KU (binary)* matching prompt (not in this folder yet).
- **RQ2 unmatched analysis** — rule-based justification in §4.2, not an LLM prompt. Optional: [failure_analysis_prompt.md](failure_analysis_prompt.md).

**Figure 2** labels recommendation as “P3”; §3.4 text uses **(P4)** — follow §3.4 and `P4_reviewer_matching_prompt.md`.

## Model configuration

Paper §3.3: **`mistralai/ministral-8b-2512`**, **temperature 0.1**, OpenRouter.

| Prompts | Paper | Axios notebook (if different) |
|---------|-------|-------------------------------|
| P1, P2 | temp **0.1** | `developer_profile.ipynb` uses **0.3** |
| P3, P4 | temp **0.1** | `TopThreeReviewerList.ipynb` |

## Pipeline order

1. **P1** → 2. **P2** → 3. **P3** → 4. **P4** (each matches one LangChain template in code).
