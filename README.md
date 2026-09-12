# Human vs Agentic PR Integration Dynamics

A preliminary empirical study of **when pull requests are merged**, comparing human developers with five AI coding agents using the public [AIDev dataset](https://huggingface.co/datasets/hao-li/AIDev).

The analysis estimates cumulative merge probability with the **Aalen–Johansen (AJ) estimator**, treating unmerged closure as a competing event and unresolved observations as right-censored. Results describe the selected PR cohorts; they are not a causal benchmark of agent capability or code quality.

## Research questions

1. How does cumulative merge probability over time differ between human and agent PRs in a common repository pool?
2. How do these patterns change when merges occurring in less than one minute are excluded?
3. How do AJ estimates compare with 1 − Kaplan–Meier when unmerged closure is instead treated as censoring?

## Merge probability over time

![AJ cumulative merge probability on a logarithmic time axis, with main and sensitivity cohorts](figures/Aalen_Johansen_CIF.png)

The logarithmic axis makes minute- and hour-scale patterns visible alongside longer-term outcomes. The main analysis includes merges before one minute; the axis start only limits the displayed range. Each curve gives the estimated probability of merging **by that time**, accounting for competing unmerged closure.

- **The comparison changes over time.** OpenAI Codex has higher estimated cumulative merge probability during much of the first day. Human PRs catch up around one day and have higher estimates thereafter: 75.6% versus 67.2% at 30 days.
- **Excluding merges under one minute retains the broad pattern**, while changing some early differences. This sensitivity analysis changes the cohort and is not an adjustment for confounding.
- **These are descriptive point estimates.** Curve ordering does not establish statistically significant differences or explain why the groups differ.

<details>
<summary>Linear time axis: first 30 days</summary>

![The same AJ analyses on a linear time axis, displaying the first 30 days](figures/Aalen_Johansen_CIF_Linear.png)

This is an alternative view of the same analyses, not a separate 30-day follow-up design.

</details>

## A 30-day snapshot

![30-day AJ estimates: Human 75.6%, OpenAI Codex 67.2%, Cursor 58.0%, Claude Code 55.7%, Devin 49.7%, Copilot 43.2%](figures/CIF_30d_Dot_Plot.png)

Thirty days is a summary timepoint within the 60-day analysis window. The dashed line marks the Human estimate; confidence intervals are omitted. Estimates at **1, 3, 7, 30 and 60 days** are available in [cif_table.csv](results/cif_table.csv). Observed event counts and crude merge proportions are in [summary_table.csv](results/summary_table.csv); these proportions are distinct from censoring-aware CIF estimates.

## Why compare KM and AJ?

![Within-group comparison of 1 minus Kaplan–Meier and AJ, with pointwise AJ confidence intervals](figures/Methodology_Comparison_ALL_Agents_KM_vs_AJ.png)

AJ estimates cumulative merge probability in the presence of competing unmerged closure. The comparison curve, **1 − KM**, treats those closures as censored and estimates a different quantity; it should not be read as the observed-world cumulative merge probability with competing closures present. In these cohorts, it yields higher estimates than AJ.

Shading shows **95% pointwise AJ confidence intervals**, without adjustment for repository clustering. The closure percentages in panel titles are observed proportions over each PR's available follow-up, not closure CIF estimates and not necessarily 30-day proportions. The gap between curves illustrates the choice of estimand, rather than a significance test between methods.

## Study design

| Component | Definition |
| --- | --- |
| Data | `hao-li/AIDev`, configurations `pull_request` and `human_pull_request`, revision `68ed5f4` |
| Cohort | Repositories present in both source configurations; PRs deduplicated by `html_url`, retaining the agent label when duplicated |
| Main sample | 16,263 PRs: Human 6,513; OpenAI Codex 3,449; Claude Code 166; Copilot 2,443; Cursor 458; Devin 3,234 |
| Time origin | PR creation |
| Event of interest | Merge within available follow-up |
| Competing event | Closure within follow-up for PRs without a recorded merge; modeled as terminal |
| Censoring | No qualifying event before the earlier of 60 days after creation and `2025-08-01 23:59:59 UTC` |
| Sensitivity cohort | Exclude PRs merged in less than 60 seconds; retain other eligible PRs |

The common repository pool does **not** imply matched PRs, equal repository weights, or that every agent appears in every repository. Follow-up is **up to** 60 days per PR; later-created PRs can have shorter observation periods.

## Run the notebook

```bash
git clone https://github.com/ra1nowzz/Human_vs_Agentic_PR_Integration_Dynamics.git
cd Human_vs_Agentic_PR_Integration_Dynamics
pip install -r requirements.txt
jupyter notebook analysis.ipynb
```

Run the cells in order from the repository root. The notebook downloads the fixed dataset revision, constructs the cohorts, fits the estimators, and writes four figures and two CSV tables. Internet access is needed for the initial dataset download. Package versions are not pinned, so the environment is not an exact dependency lock.

| Path | Contents |
| --- | --- |
| [analysis.ipynb](analysis.ipynb) | Complete analysis and plotting code |
| [requirements.txt](requirements.txt) | Python dependencies |
| [figures/](figures/) | Logarithmic and linear CIF views, 30-day summary, KM–AJ comparison |
| [results/](results/) | Event counts and fixed-time CIF estimates |

## Limitations and next steps

- **Comparability:** Task type, PR size, calendar period, contributor experience, and repository policies are not controlled. Agent selection and usage patterns may also differ.
- **Uncertainty:** Cohort sizes differ substantially, especially for Claude Code. Current confidence intervals do not account for correlation among PRs in the same repository. Censoring-aware estimation still relies on appropriate censoring assumptions.
- **Event history:** Closure is modeled as terminal using snapshot timestamps; reopening and repeated review–revision cycles are not reconstructed.
- **Scope:** Merge probability measures one aspect of integration. It does not establish code quality, review burden, developer productivity, or causal effects of agent use.

Planned extensions are to examine repository and PR-level comparability, add uncertainty estimates that account for repository clustering, and—subject to comparable Human and agent timeline data—study where differences emerge around first human response and subsequent revision. These extensions are not implemented in this notebook.

## References and license

- [The Rise of AI Teammates in Software Engineering (SE) 3.0](https://arxiv.org/abs/2507.15003) — source study introducing AIDev.
- [On the Use of Agentic Coding: An Empirical Study of Pull Requests on GitHub](https://arxiv.org/abs/2509.14745) — related empirical study motivating interest in PR integration and incomplete observations.
- [AalenJohansenFitter documentation](https://lifelines.readthedocs.io/en/latest/fitters/univariate/AalenJohansenFitter.html) — estimator implementation used here.

Code is available under the [MIT License](LICENSE). The source data remain subject to the terms and attribution requirements on the [AIDev dataset page](https://huggingface.co/datasets/hao-li/AIDev).
