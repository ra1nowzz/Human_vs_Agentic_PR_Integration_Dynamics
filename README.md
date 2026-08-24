# Agentic PR Integration Dynamics

A small empirical software engineering analysis comparing the merge dynamics of human-authored and agentic pull requests using a repository-aligned cohort and the Aalen–Johansen cumulative incidence estimator.

![Aalen–Johansen cumulative incidence curves](figures/Aalen_Johansen_CIF.png)

## Research Question

How does the cumulative incidence of pull-request merge differ between human-authored and agentic PRs within the same repository cohort, and are the observed patterns sensitive to extremely fast (< 1 minute) merges?

## Data

The analysis uses the public **AIDev** dataset on Hugging Face:

- Dataset: `hao-li/AIDev`
- Configurations: `pull_request` and `human_pull_request`
- Fixed dataset revision: `68ed5f4`
- Dataset cutoff: `2025-08-01 23:59:59 UTC`
- Actors included: Human, OpenAI Codex, Claude Code, Copilot, Cursor, and Devin

To reduce repository-composition differences, the analysis keeps only repositories that appear in both the agentic and human PR datasets.

## Method

PR outcomes are modeled as competing risks:

- `0`: administratively censored
- `1`: merged
- `2`: closed without merge

Each PR is followed for up to **60 days**, subject to the dataset cutoff. The cumulative incidence of merge is estimated separately for each actor using the **Aalen–Johansen estimator**.

Two cohorts are compared:

1. **Main analysis** — includes all observed merges.
2. **Sensitivity analysis** — excludes merges occurring less than one minute after PR creation.

The figure displays the first 30 days after PR creation.

## Main Observation

Within this repository-aligned cohort, human-authored PRs show the highest cumulative incidence of merge over the displayed 30-day period. OpenAI Codex is the closest agentic group, while the remaining agent groups show lower cumulative incidence curves.

Removing sub-1-minute merges lowers some agent curves, but the broad ordering of the groups remains similar. This suggests that the main visual pattern is not driven solely by extremely fast merges.

These results are descriptive and should not be interpreted as causal evidence about agent quality or capability.

## Reproduce

Python 3.10+ is recommended.

```bash
pip install -r requirements.txt
jupyter notebook analysis.ipynb
```

The notebook downloads the dataset directly from Hugging Face at the fixed revision, so no raw dataset is stored in this repository.

## Repository Structure

```text
.
├── README.md
├── analysis.ipynb
├── requirements.txt
├── .gitignore
└── figures/
    └── Aalen_Johansen_CIF.png
```

## Notes and Limitations

- The analysis is observational.
- Restricting to repositories shared by the human and agentic cohorts improves comparability but changes the target population.
- The administrative observation window is 60 days, while the figure displays days 0–30.
- `lifelines` may jitter tied event times internally for the Aalen–Johansen estimator. A fixed random seed (`seed=42`) is used for reproducibility.
- Sample sizes differ substantially across actor groups.
