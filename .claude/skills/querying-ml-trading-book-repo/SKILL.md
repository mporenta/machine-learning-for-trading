---
name: querying-ml-trading-book-repo
description: Guides the agent through answering questions about the Machine Learning for Trading book companion repo (mporenta/machine-learning-for-trading) — a 24-chapter educational Jupyter notebook collection, NOT an application codebase. Use when the user asks where the book covers a topic, which notebook implements a technique, what's in a specific chapter, how an alpha factor is built, or how to find code/data/factors across the repo. The skill surfaces the chapter map up front so queries are targeted, and delegates the actual searches to a read-only `ml-trading-book-querier` subagent that uses the authenticated `gh` CLI.
---

# Querying the ML-for-Trading book repo

Use this skill whenever the user asks a "where / what / how does the book do X" question about the repo `mporenta/machine-learning-for-trading`. The repo is the companion code for the 2nd edition of *Machine Learning for Trading* by Stefan Jansen — a fixed educational reference, not a live application. Treat it like a textbook indexed by chapter.

## Core rule: always delegate the actual queries

You (the main agent) do **not** call `gh` directly. Spawn the `ml-trading-book-querier` subagent via the Agent tool and have it run the `gh` commands, then synthesize its findings for the user.

Why: the subagent is configured read-only with `gh` CLI only, returns structured findings, and keeps raw search output out of your context window.

When you call the subagent, **always include the relevant slice of the chapter map below in the prompt** so it starts targeted (e.g. "look in chapters 14–16 for NLP topics") instead of grepping the whole repo.

## Repo map (give this to the subagent)

```
mporenta/machine-learning-for-trading
├── 01_machine_learning_for_trading/        Machine Learning for Trading: From Idea to Execution
├── 02_market_and_fundamental_data/         Market & Fundamental Data: Sources and Techniques
├── 03_alternative_data/                    Alternative Data for Trading
├── 04_alpha_factor_research/               Financial Feature Engineering: How to research Alpha Factors
├── 05_strategy_evaluation/                 Portfolio Optimization and Performance Evaluation
├── 06_machine_learning_process/            The Machine Learning Workflow
├── 07_linear_models/                       Linear Models: From Risk Factors to Asset Return Forecasts
├── 08_ml4t_workflow/                       The ML4T Workflow: From ML Model to Strategy Backtest
├── 09_time_series_models/                  Linear Time Series Models / Statistical Arbitrage
├── 10_bayesian_machine_learning/           Bayesian ML: recession forecasts & dynamic pairs trading
├── 11_decision_trees_random_forests/       Random Forests — Long-Short Strategy for Japanese Stocks
├── 12_gradient_boosting_machines/          Boosting your Trading Strategy
├── 13_unsupervised_learning/               Data-Driven Risk Factors & Hierarchical Risk Parity
├── 14_working_with_text_data/              Text Data for Trading: Sentiment Analysis
├── 15_topic_modeling/                      Topic Modeling for Earnings Calls and Financial News
├── 16_word_embeddings/                     Word Embeddings for Earnings Calls and SEC Filings
├── 17_deep_learning/                       Deep Learning for Trading
├── 18_convolutional_neural_nets/           CNNs: Time Series as Images
├── 19_recurrent_neural_nets/               RNNs for Trading: Multivariate Time Series and Text
├── 20_autoencoders_for_conditional_risk_factors/  Autoencoders for Conditional Risk Factors
├── 21_gans_for_synthetic_time_series/      GANs for Synthetic Time Series Data
├── 22_deep_reinforcement_learning/         Deep RL: Building a Trading Agent
├── 23_next_steps/                          Next Steps
├── 24_alpha_factor_library/                Appendix — Alpha Factor Library (100+ factors)
├── data/                                   Data-sourcing notebooks (create_datasets.ipynb, etc.)
├── installation/                           Conda/pip env files (ml4t-base.yml, OS-specific pins)
├── figures/  assets/                       Book figures (not used by code)
├── utils.py                                Shared helpers (notably MultipleTimeSeriesCV)
└── README.md                               Top-level book overview & chapter index
```

### Conventions to rely on

- **Chapter dirs are numerically prefixed** `NN_<snake_case_topic>/`. Numbers mirror the book's chapter order.
- **Every chapter has a `README.md`** (authoritative for that chapter's purpose, data, and run order). Always read these first for "what's in chapter N" questions.
- **Notebooks are `NN_<topic>.ipynb`** inside chapter dirs. Some chapters add subdirectories for multi-part topics (e.g. `13_unsupervised_learning/03_clustering_algorithms/`).
- **The data lives elsewhere** — most notebooks expect data fetched by `data/create_datasets.ipynb`, `data/create_stooq_data.ipynb`, or by `zipline ingest`. If a user asks "where does data X come from", search `data/` first.
- **Chapter 24** is the alpha factor appendix — go straight there for "list factors", "formulaic alphas", "alpha factor library".
- **Backtesting** chapters are 8, 11, 12, 18 (and others) — they use Zipline; custom bundles live under `08_ml4t_workflow/04_ml4t_workflow_with_zipline/01_custom_bundles/`.

## Workflow

### Step 1 — classify the question

| Question shape | Target |
|----------------|--------|
| "Where does the book cover X?" / "Which chapter is X in?" | Chapter READMEs + top-level `README.md` |
| "Which notebook implements X?" | `gh search code` scoped to the repo |
| "What's in chapter N?" | `NN_*/README.md` + directory listing |
| "How is alpha factor X built?" | `24_alpha_factor_library/` notebooks |
| "Where does data Y come from?" | `data/` notebooks and `data/README.md` |
| "What env / Python version / dependencies?" | `installation/` and root `CLAUDE.md` |
| "What's `MultipleTimeSeriesCV`?" | `utils.py` |

### Step 2 — delegate to the subagent

Spawn `ml-trading-book-querier` with a prompt that:

1. States the user's question verbatim.
2. Includes the targeted slice of the chapter map (don't paste the whole map — pick the chapters that match).
3. Specifies which `gh` operations to try first (search vs. README fetch vs. directory listing).
4. Asks for a structured report: matched files (with paths + line numbers if from `gh search code`), short excerpt of each match, and a one-line answer suggestion.

### Step 3 — synthesize for the user

- Cite chapter number and notebook path (e.g. `12_gradient_boosting_machines/04_boosting_for_intraday_strategy_part1.ipynb`).
- Where helpful, link to a `gh`-style permalink form: `https://github.com/mporenta/machine-learning-for-trading/blob/main/<path>`.
- If the subagent's findings are thin, send a follow-up to it with a refined query — don't burn a fresh Agent call.

## Examples

### Example 1 — "Where does the book cover Kalman filters?"

Main agent's delegation prompt (abbreviated):

> User asks: "Where does the book cover Kalman filters?"
> Likely chapters: 4 (alpha factor research) and 9 (time series). Repo: `mporenta/machine-learning-for-trading`.
> Run: `gh search code --repo mporenta/machine-learning-for-trading "Kalman"` and `gh api repos/mporenta/machine-learning-for-trading/contents/04_alpha_factor_research/README.md`.
> Report matched paths + a one-line summary of each hit.

Expected subagent response shape:

```
Matches:
- 04_alpha_factor_research/03_kalman_filter_and_wavelets.ipynb  (gh search code: 5 hits for "Kalman")
- 04_alpha_factor_research/README.md  (mentions Kalman filter under "Denoising techniques")
- 09_time_series_models/README.md     (Kalman filter referenced in state-space models section)

Suggested answer: Chapter 4 has the dedicated notebook
`03_kalman_filter_and_wavelets.ipynb`; chapter 9 references it again in
the state-space context.
```

Main agent reply to user: a short paragraph naming the notebook + path, plus the chapter 9 cross-reference.

### Example 2 — "List the formulaic alphas implemented in the appendix"

Delegation: tell the subagent the answer is in `24_alpha_factor_library/03_101_formulaic_alphas.ipynb` and the chapter `README.md`; have it fetch `gh api repos/mporenta/machine-learning-for-trading/contents/24_alpha_factor_library/README.md` and list the notebooks in that directory.

### Example 3 — "Which notebooks use `zipline ingest`?"

Delegation: `gh search code --repo mporenta/machine-learning-for-trading "zipline ingest"`. Expect hits across chapters 8, 11, 12, 18. Subagent returns a deduplicated list of notebook paths; main agent groups them by chapter for the reply.

### Example 4 — "Is there a test suite?"

Don't even delegate — answer directly from this skill: no, the repo is a notebook collection with no tests/linters/build step (per `CLAUDE.md`). Each notebook is self-contained and most are checked in already executed.

## Anti-patterns

- **Don't** spawn the subagent for questions the chapter map already answers (e.g. "what chapter is on RNNs?" → chapter 19, no query needed).
- **Don't** ask the subagent to clone the repo or use local `grep`/`find`. It is `gh`-only by design.
- **Don't** modernize APIs or recommend rewriting notebooks to newer library versions — pinned dependencies are intentional (see `CLAUDE.md`).
- **Don't** invent data files. If a notebook references missing data, point at the relevant `data/` sourcing notebook or `zipline ingest` step.
- **Don't** paste large notebook JSON into the user reply. The subagent should excerpt code cells, not dump raw `.ipynb` contents.

## Subagent reference

The subagent definition lives at `.claude/agents/ml-trading-book-querier.md`. It is read-only, restricted to `Bash` (for `gh`), and lists the canonical `gh` invocations for searching code, reading files, and listing directory contents in this repo.
