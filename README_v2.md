# Investment Research Agent

An agentic AI system that researches a stock end to end. Given a ticker and a
question, a team of cooperating LLM agents plans the research, chooses and runs
data tools, summarises recent news, reviews and improves its own drafts,
remembers lessons between runs, and writes a sourced report with charts.

Final project for the University of San Diego Applied Artificial Intelligence
program (Natural Language Processing and GenAI), Team 2. It runs entirely on
free OpenRouter models and free Yahoo Finance data.

**Repository:** https://github.com/umea14/Investment-Research-Agent

## What it does

| Agent | Job |
| --- | --- |
| **Coordinator** | Plans the research, routes the question, merges results, runs review and learning, writes the report |
| **Researcher** | Chooses Yahoo Finance tools, computes price and quarterly metrics in code, writes a grounded analysis |
| **NewsResearcher** | Prompt chain over recent headlines: ingest, preprocess, classify, extract, summarize |
| **Evaluator** | Independently scores each draft for accuracy, completeness and clarity and gives feedback |

### Project requirements and where they are implemented

| Requirement | Implementation (in `notebook.ipynb`) |
| --- | --- |
| Plans research steps for a stock | `Coordinator.plan_research` (Section 4) |
| Uses tools dynamically | LLM selects Yahoo tools as JSON; code validates the selection before running it (Sections 3-5) |
| Self-reflects on output quality | `EvaluatorAgent` (Section 7) |
| Learns across runs | `LessonMemory` and `LearningCoordinator.reflect` (Section 8) |
| Workflow 1: prompt chaining | `NewsResearcherAgent`, with a relevance gate (Section 5) |
| Workflow 2: routing | `RoutingCoordinator.route_task`, with a keyword fallback (Section 6) |
| Workflow 3: evaluator-optimizer | `ReviewingCoordinator.refine_packet`, at most 2 revisions (Section 7) |
| Report and visualizations | `ReportingCoordinator`, `plot_research` (Section 9) |
| Demonstration and tests | Section 10: offline unit tests, three end-to-end runs |

## Architecture

```
question + ticker
      |
  Coordinator --- plan (LLM) --- route (LLM; keyword rules if all models fail)
      |--> Researcher:     Yahoo prices + financials -> metrics (code) -> narrative (LLM)
      |--> NewsResearcher: ingest -> preprocess -> classify -> gate -> extract -> summarize
      |
  Evaluator <-- each draft: score -> feedback -> revise (at most 2 rounds)
      |
  LessonMemory <-- criticism becomes a general rule; recalled in the next run
      |
  Coordinator --> executive summary + markdown report + charts
```

Design principle: the LLMs decide and explain, while ordinary code computes
every number, validates every LLM output against a schema, bounds every loop,
and decides the evaluator's verdict. A failed or invalid LLM answer is retried
on at most three free models and is never replaced with an invented one.

## Setup

1. Create an OpenRouter API key at <https://openrouter.ai/keys>.
2. Open `notebook.ipynb` in Google Colab or Jupyter (Python 3.10+).
3. Run the cells from top to bottom. Paste your key at the hidden prompt. The
   key is never written to the notebook or to disk.

Dependencies are installed by the first cell and are listed in
`requirements.txt`.

**Free-tier limits.** Free OpenRouter models are limited to about 50 requests
per day (1,000 per day once an account has bought $10 of credits). One full
run makes roughly 20-30 requests, so the complete notebook needs the higher
allowance or a fresh day. If a cell reports "all 3 free attempts failed", wait
a minute and re-run it; if the message mentions `free-models-per-day`, the
daily allowance is used up.

## Usage

```python
system = InvestmentResearchTeam(client)
run = system.analyze_stock(
    "AAPL",
    "How has the stock performed over six months, and what news explains it?",
    price_period="6mo",     # optional: 1mo, 3mo, 6mo, 1y, 2y, 5y
    excluded_tools=(),      # optional, e.g. ["get_company_financials"]
    max_revisions=2,        # evaluator-optimizer rounds per draft
)
show_run(run)               # report, charts and delegation log
```

`run` is a dictionary holding the plan, routing decision, raw evidence,
analysis, news-chain trace, evaluation rounds, lessons used and the final
report. Lessons and score history are stored in `agent_memory.json` next to
the notebook (in Colab this lasts for the runtime session).

## Results of the final run

Three runs (AAPL, MSFT, NVDA) were all routed to both specialists and produced
a complete report with four charts. Of six drafts, five were approved on the
first review and one (the NVDA news summary) was revised once, improving from
4.00 to 4.33. Three lessons were stored and recalled in later runs. Three runs
are too few to show that memory improves quality; they show that the mechanism
works end to end. Full outputs are in the notebook (Sections 10 and 11).

## Repository contents

| File | Purpose |
| --- | --- |
| `notebook.ipynb` | The complete project with outputs |
| `README.md` | This file |
| `requirements.txt` | Python dependencies |

## Limitations

* Free models are rate limited and occasionally fail; results vary between runs.
* Yahoo Finance data is unofficial and not independently verified. Only four
  quarters of financials are available, so there is no year-over-year view.
* News comes from the headlines Yahoo returns, many of them promotional;
  category and sentiment labels are model judgements.
* An LLM reviews LLM output. The automatic check catches wrong percentages but
  not wrong reasoning, and the approval threshold is a heuristic.
* This is research support, not investment advice.

## Code style

Python code follows PEP 8 (79 columns), formatted with `black` and checked with
`pycodestyle`. Section 10.1 of the notebook has offline unit tests for the
deterministic code that need no API key.

## Team

Team 2, University of San Diego, Applied Artificial Intelligence.
