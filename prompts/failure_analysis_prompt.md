# Failure analysis prompt (evaluation only)

**Not numbered P1–P4.** Used offline when a top-3 recommendation misses the actual assignee (Section 3 evaluation / validation).

**Source:** `newAnalysis/LLM+KU+freq/axios/PrecisionMetricsCalculation/Reviewer_Metrics_Evaluation.ipynb` (`analyze_failure_with_llm`).

**Model configuration:** `mistralai/ministral-8b-2512`, temperature 0.1, max tokens 4,000, OpenRouter.

## Prompt

```
Analyze why this PR reviewer assignment failed to match:

**PR #{pr_number}**
Title: {title}
Description: {description}
Labels: {labels}
Files Changed: {files_changed}
Complexity: {complexity}

**Actual Reviewers Assigned:** {actual_reviewers}
**Predicted Top 3:** {predicted_top3}

**Top 3 Predictions with Scores:**
{top_3_scores_lines}

Based on the PR details above, Provide a concise reason why the actual reviewer was better
suited than the predicted ones. Focus on the most important factor only. Check the PR properly and avoid generic answers. Also distinguish betwwen the actual and predicted reviewers.
```

**Note:** Placeholders describe runtime f-string fields in the notebook. Closing sentence is verbatim from source (including “Provide” and “betwwen”).
