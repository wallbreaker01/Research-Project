# LLM-Based Pull Request Reviewer Recommendation

This repository contains the data-processing scripts, LLM prompts, Jupyter
notebooks, and generated artifacts for an empirical study of pull-request (PR)
reviewer recommendation in JavaScript open-source projects.

The central idea is to recommend reviewers by combining:

1. The technical requirements of a new PR.
2. Historical developer and reviewer activity.
3. LLM-generated skill profiles.
4. Knowledge-unit representations of skills and PRs.
5. Frequency information about how often skills occur.
6. Similarity search over historical reviews using embeddings and FAISS.

The repository is organized as a research workspace rather than as a packaged
Python application. Most experiments are implemented as notebooks, while the
small command-line scripts collect and prepare data.

## What this project does

The complete research pipeline is:

```text
GitHub PR history
        |
        v
Normalized developer/reviewer PR data
        |
        v
Chronological train/test split
        |
        +--> LLM skill extraction
        |        |
        |        v
        |    Structured developer/reviewer profiles
        |
        +--> PR requirement extraction
                 |
                 v
        Candidate retrieval with embeddings/FAISS
                 |
                 v
        LLM reviewer ranking
                 |
                 v
        Top-k predictions
                 |
                 v
        Precision, recall, F1, hit rate, MRR, NDCG,
        stability, and failure analysis
```

The prompts in [`prompts/`](./prompts/)
are the canonical description of the LLM stages. They define the intended
sequence:

- **P1**: extract JavaScript expertise from PR history.
- **P2**: generate a structured developer or reviewer profile.
- **P3**: extract structured requirements from a target PR (fields + JSON in one file).
- **P4**: match the PR to candidate reviewers and rank the top three (matching + JSON in one file).
- **Failure analysis** (`failure_analysis_prompt.md`): explain missed recommendations during offline evaluation (not numbered P1–P4).

## Repository layout

```text
.
├── prompts/
│   ├── README.md
│   ├── P1_skill_extraction_prompt.md
│   ├── P2_structured_profile_prompt.md
│   ├── P3_pr_requirement_extraction_prompt.md
│   ├── P4_reviewer_matching_prompt.md
│   └── failure_analysis_prompt.md
├── newAnalysis/
│   ├── LLM/
│   ├── LLM+KU/
│   └── LLM+KU+freq/
└── oldAnalysis/
    ├── LLM/
    ├── LLM+KU/
    └── LLM+KU+freq/
```

Each analysis variant contains repository-specific folders such as `axios`,
`chartjs`, `Express Repo`, `fastify`, `JQuery Repo`, `Lodash Repo`, `Moment
Repo`, `npm_cli`, and `preact`. The exact set of repositories varies between
variants and analysis snapshots.

Typical contents of a repository-specific folder are:

```text
<repository>/
├── PR data for Developers/
├── PR data for Reviewers/
├── Developer's Profiles/
├── Reviewers Profiles/
├── developer_profile.ipynb
├── Reviewer_profile.ipynb
├── SinglePRAssignment(Developer).ipynb
├── SinglePRAssignment(Reviewer).ipynb
└── PrecisionMetricsCalculation/
    ├── Actual-PR-Reviewer-Extractor.ipynb
    ├── TopThreeReviewerList.ipynb
    └── Reviewer_Metrics_Evaluation.ipynb
```

Some folders contain additional intermediate JSON files, checkpoints, and
evaluation outputs. These are experiment artifacts, not Python package
modules.

## Analysis variants

The three directory names describe progressively richer representations:

| Variant | Meaning |
|---|---|
| `LLM` | LLM-generated skill/profile information without the additional knowledge-unit and frequency-weighting stages. |
| `LLM+KU` | LLM information combined with knowledge units representing technical concepts and skills. |
| `LLM+KU+freq` | Knowledge units augmented with frequency information from historical activity. This is the most complete variant in the repository. |

`oldAnalysis/` and `newAnalysis/` are separate analysis snapshots. They
contain duplicated experiment structures so results from different iterations
can be compared without overwriting one another. Treat them as independent
experiments; do not mix their generated inputs and outputs unless the
experiment design explicitly requires it.

## Main components

### Data collection scripts

The scripts under `LLM+KU+freq/` collect and prepare GitHub data:

- `DeveloperFeachData.py` fetches PR metadata and changed-file information and
  writes normalized developer-oriented PR JSON.
- `ReviewerFetchData.py` fetches PR review information and writes
  reviewer-oriented PR JSON.
- `split_pr_data.py` sorts PRs by `created_at` and creates chronological
  `train` and `test` files. This avoids using future PRs to predict earlier
  assignments.

The same scripts are present in multiple analysis variants. Use the copy
inside the variant being evaluated so that its defaults and output naming
remain consistent with that experiment.

### Profile-generation notebooks

- `developer_profile.ipynb` analyzes developer PR activity and generates
  structured developer profiles.
- `Reviewer_profile.ipynb` analyzes review activity and generates structured
  reviewer profiles.

The notebooks process records in batches and include checkpoint/progress logic
so a long profile-generation run can be resumed.

### Assignment notebooks

- `SinglePRAssignment(Developer).ipynb` recommends a developer for a target PR.
- `SinglePRAssignment(Reviewer).ipynb` recommends a reviewer for a target PR.
- `PrecisionMetricsCalculation/TopThreeReviewerList.ipynb` performs bulk
  reviewer assignment and produces ranked predictions, generally using the
  top three candidates for evaluation.

The assignment notebooks use sentence-transformer embeddings and FAISS to
retrieve similar profiles or historical PR/review examples before asking the
LLM to rank candidates.

### Evaluation notebooks

- `Actual-PR-Reviewer-Extractor.ipynb` extracts the actual reviewer assignments
  from the review data.
- `Reviewer_Metrics_Evaluation.ipynb` compares predictions with actual
  assignments.

The evaluation notebook calculates top-k accuracy, precision, recall, F1,
hit-rate, average precision, reciprocal rank/MRR, DCG/NDCG, stability
measures, and detailed correct/incorrect prediction records. It also contains
quantitative and LLM-assisted failure analysis.

## Data formats

### Developer PR data

Developer-oriented files are JSON lists. A record contains fields such as:

```json
{
  "pr_id": "...",
  "repo_name": "owner/repository",
  "pr_number": 123,
  "title": "Short PR title",
  "state": "closed",
  "author": {
    "username": "author",
    "id": 12345,
    "profile_url": "https://github.com/author"
  },
  "created_at": "2024-01-01T00:00:00Z",
  "description": "...",
  "labels": [],
  "changed_files": [],
  "language_stats": {}
}
```

### Reviewer PR data

Reviewer-oriented files are JSON objects containing metadata and a
`pr_reviewer_data` list. PR entries include review records with fields such as
`reviewer_username`. The split script preserves the metadata and updates the
split statistics.

### Profiles and predictions

Profile and prediction filenames vary slightly between snapshots, but the
directories normally identify their purpose:

- `Developer's Profiles/`: generated developer skill profiles.
- `Reviewers Profiles/`: generated reviewer skill profiles.
- `actual_pr_reviewer_assignments.json`: observed assignments.
- `all_pr_reviewer_assignments.json`: model predictions for a collection of
  PRs.
- `pr_reviewer_evaluation_metrics.json`: aggregate evaluation results.

Inspect the JSON schema and metadata in the specific experiment folder before
writing a new consumer. Do not assume that a file produced by `oldAnalysis/`
has exactly the same fields as its `newAnalysis/` counterpart.

## Requirements

Use Python 3.9 or newer. A practical environment for the notebooks includes:

- Jupyter Notebook or JupyterLab.
- `requests` for GitHub API access.
- `pandas` and `numpy` for data preparation and metrics.
- LangChain and its OpenAI-compatible integration.
- `pydantic` for structured LLM output.
- `sentence-transformers` for embeddings.
- `faiss-cpu` for vector search.

The notebooks are not fully standardized: some install packages in a notebook
cell and some use imports from different LangChain generations. If an import
fails, use the imports shown in the notebook being executed and install the
missing package in the active environment rather than changing a different
experiment snapshot.

## Setup

From the repository root in PowerShell:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install jupyter requests pandas numpy pydantic `
  langchain langchain-openai sentence-transformers faiss-cpu
python -m jupyter notebook
```

If PowerShell prevents activation, either allow local scripts for the current
user or invoke the environment's Python directly:

```powershell
.\.venv\Scripts\python.exe -m jupyter notebook
```

The notebooks may also reference an OpenRouter-compatible model. Install the
provider integration required by the specific notebook and use the model
configured in that experiment, rather than silently changing the model when
reproducing published results.

## Credentials and safety

Two types of credentials may be needed:

1. A GitHub token for the data-collection scripts.
2. An OpenRouter/OpenAI-compatible API key for LLM notebooks.

Never commit either credential. Before running a notebook, replace any
credential cell with environment-variable configuration, for example:

```python
import os

os.environ["OPENAI_API_KEY"] = os.environ["OPENROUTER_API_KEY"]
os.environ["OPENAI_BASE_URL"] = "https://openrouter.ai/api/v1"
```

For GitHub requests, configure the `Authorization` header from an environment
variable in your local copy of the collection script. Use a token with only
the permissions required to read the repositories and reviews being analyzed.
If a token has ever been committed or shared, revoke it and issue a new one
before continuing.

## Reproducing an experiment

The following is the recommended order. Run commands from the relevant
repository-specific directory when a notebook uses relative paths.

### 1. Collect PR and review data

Open the selected variant's `DeveloperFeachData.py` and
`ReviewerFetchData.py`. Configure `REPO_OWNER`, `REPO_NAME`, the output
directory, and local credential handling before running them:

```powershell
Set-Location ".\newAnalysis\LLM+KU+freq"
python .\DeveloperFeachData.py
python .\ReviewerFetchData.py
```

The collection code requests PR pages from the GitHub REST API, retries
transient failures/rate limits, and records normalized metadata. Large
repositories can take a long time and are subject to GitHub API limits.

### 2. Create a chronological split

Run the splitter against the generated JSON file:

```powershell
python .\split_pr_data.py `
  ".\fastify\PR data for Developers\fastify_all_pr_data.json" `
  0.8
```

The script accepts either a developer PR list or a reviewer-data object. The
default ratio depends on the experiment snapshot: the `LLM` and `LLM+KU`
copies default to 80/20, while `LLM+KU+freq` copies default to 90/10. Pass an
explicit ratio when reproducing a particular study configuration.

### 3. Generate profiles

Open the `developer_profile.ipynb` and `Reviewer_profile.ipynb` notebooks in
the target repository folder. Set the input paths, API configuration, model,
batch size, and output directories. Run the cells in order. Keep the
checkpoint files so an interrupted run can resume.

### 4. Generate reviewer predictions

Use `SinglePRAssignment(Reviewer).ipynb` for a single PR or
`PrecisionMetricsCalculation/TopThreeReviewerList.ipynb` for bulk assignment.
Confirm that:

- The target PR data is from the test split.
- Candidate profiles come from the training data.
- The profile directory matches the selected variant.
- The embedding model and LLM model match the experiment configuration.

The developer assignment notebook follows the same process but ranks
developers instead of reviewers.

### 5. Evaluate predictions

Run `Actual-PR-Reviewer-Extractor.ipynb` first, then
`Reviewer_Metrics_Evaluation.ipynb`. Check that the PR identifiers overlap
between actual and predicted files before interpreting the metrics. The
evaluation notebook writes aggregate metrics and detailed error records to JSON
in the evaluation directory.

## How to understand the project

If you are new to the repository, read it in this order:

1. [`prompts/README.md`](./prompts/README.md)
   to understand the intended LLM pipeline.
2. One variant's `split_pr_data.py` to understand the data contract and
   chronological split.
3. `developer_profile.ipynb` and `Reviewer_profile.ipynb` to see how raw
   activity becomes profiles.
4. `SinglePRAssignment(Reviewer).ipynb` to understand candidate retrieval and
   ranking.
5. `TopThreeReviewerList.ipynb` to understand bulk prediction output.
6. `Reviewer_Metrics_Evaluation.ipynb` to understand the reported results.
7. The corresponding folders under `oldAnalysis/` and `newAnalysis/` to
   compare iterations.

When tracing a result backward, start with the metric JSON, identify the
prediction file and PR identifiers used, then follow those identifiers to the
test split, profile files, and original GitHub-derived PR records.

## Reproducibility guidance

- Record the analysis snapshot (`oldAnalysis` or `newAnalysis`), variant, and
  repository for every run.
- Record the train/test ratio, model name, temperature, embedding model,
  prompt revision, and date of data collection.
- Preserve the generated JSON files used by an evaluation; regenerating data
  from GitHub later may produce different results as repositories change.
- Do not train on the target PR or on reviews that occur after the prediction
  point.
- Keep API keys outside notebooks and outside generated outputs.
- Treat notebook outputs as evidence for a run, not as the source of truth for
  the implementation. Clear stale outputs before publishing a new result.

## Known limitations

- There is no single dependency lockfile or top-level executable entry point.
- Notebook code and dependency imports differ between experiment snapshots.
- Many paths are relative, so notebooks should be run from their experiment
  directory.
- GitHub API responses, repository history, model versions, and rate limits can
  change over time.
- Generated datasets and profiles are large and may not be suitable for
  memory-constrained environments.
- The repository contains duplicated snapshots; changes made in one variant do
  not automatically propagate to another.

## Contributing changes

For a new experiment:

1. Copy the closest existing variant rather than editing a different snapshot
   in place.
2. Keep prompts, data schema, notebooks, and generated outputs aligned.
3. Document model and split settings in the experiment directory.
4. Run the smallest relevant notebook path on a small PR subset first.
5. Validate that output JSON can be loaded and that prediction/actual PR IDs
   align before running a full evaluation.

This repository is intended for research reproducibility and analysis. It is
not currently a production reviewer-assignment service.
