# Investment Research Agent - An Agentic AI Financial Analysis System

### AAI-520-02: Natural Language Processing and GenAI  
**Final Team Project | Group 2 | University of San Diego**

## Project Overview

This project develops an **agentic AI financial analysis system** that researches a
user specified stock from end to end. Given a ticker and research question, a team
of cooperating LLM agents plans the research, dynamically selects and runs Yahoo
Finance tools, analyzes market and financial data, processes recent news through a
prompt chain, evaluates and revises draft analyses, learns from prior feedback, and
produces an evidence based report with supporting visualizations.

The system uses free OpenRouter-hosted language models and Yahoo Finance data and
demonstrates planning, dynamic tool use, delegated self-reflection, persistent
memory, prompt chaining, routing, and an evaluator-optimizer workflow.

**GitHub Repository:** https://github.com/umea14/Investment-Research-Agent

## Team Members

This project was completed collaboratively by:

- Robert Shifrin
- Victoria Ritten
- Erika Gallegos

The team contributed to project planning, implementation, testing, review,
analysis, documentation, and refinement of the final submission.

## Agent Roles and Capabilities

| Agent | Job |
| --- | --- |
| **Coordinator** | Plans the research, routes the question, merges results, runs review and learning, and writes the report |
| **Researcher** | Chooses Yahoo Finance tools, computes price and quarterly metrics in code, and writes a narrative from the resulting metrics |
| **NewsResearcher** | Runs a prompt chain over recent Yahoo Finance titles and summaries: ingest, preprocess, classify, extract, summarize |
| **Evaluator** | Independently scores each draft for accuracy, completeness, and clarity and gives feedback |

### Project Requirements and Where They are Implemented

| Requirement | Implementation (in `notebook.ipynb`) |
| --- | --- |
| Plans research steps for a stock | `Coordinator.plan_research` (Section 4) |
| Uses tools dynamically | LLM selects Yahoo tools as JSON; code validates constraints and checks for required market tools before execution (Sections 3-5) |
| Self-reflects on output quality | Delegated self-reflection through `EvaluatorAgent` (Section 7) |
| Learns across runs | `LessonMemory` and `LearningCoordinator.reflect` (Section 8) |
| Workflow 1: prompt chaining | `NewsResearcherAgent`, with a relevance gate (Section 5) |
| Workflow 2: routing | `RoutingCoordinator.route_task`, with a keyword fallback (Section 6) |
| Workflow 3: evaluator-optimizer | `ReviewingCoordinator.refine_packet`, at most 2 revisions (Section 7) |
| Report and visualizations | `ReportingCoordinator`, `plot_research` (Section 9) |
| Demonstration and tests | Section 10: offline unit tests and three end-to-end runs |

## Architecture

```text
question + ticker
      |
  Coordinator --- plan (LLM) --- route (LLM; keyword rules if all models fail)
      |--> Researcher:     Yahoo prices + financials -> metrics (code) -> narrative (LLM)
      |--> NewsResearcher: ingest -> preprocess -> classify -> gate -> extract -> summarize
      |
  Evaluator <-- each draft: score -> feedback -> revise (at most 2 rounds)
      |
  LessonMemory <-- criticism becomes a general rule; recalled in later runs
      |
  Coordinator --> executive summary + markdown report + charts
```

Design Principle: The LLMs decide, interpret, and explain, while ordinary code
computes the core financial metrics, validates structured LLM outputs where
schemas are defined, screens generated percentage magnitudes against supplied
numeric evidence, bounds loops, and determines evaluator verdicts from the
scores. A failed or invalid LLM response is retried on at most three free models
and is never replaced with invented content.

For the news pipeline, prompts restrict classification, extraction, and
summarization to the Yahoo Finance titles, summaries, and publication dates
supplied to the model. Structured outputs are schema-validated, but extracted
facts and final summaries are not independently verified against the original
source text.

## Setup

1. Create an OpenRouter API key at <https://openrouter.ai/keys>.
2. Open `notebook.ipynb` in Google Colab or Jupyter (Python 3.10+).
3. Run the cells from top to bottom. Paste your key at the hidden prompt. The
   key is never written to the notebook or to disk.

Dependencies are installed by the first cell and are listed in
`requirements.txt`.

**Free-tier limits.** Free OpenRouter models are rate limited, and availability
can vary. One end-to-end run makes roughly 15-25 LLM requests. If a cell reports
that all free-model attempts failed, wait and retry the cell or use another
available free model. Check the OpenRouter account dashboard for current limits.

## Usage

```python
system = InvestmentResearchTeam(client)
run = system.analyze_stock(
    "AAPL",
    "How has the stock performed over six months, and what news explains it?",
    price_period="6mo",     # optional: 1mo, 3mo, 6mo, 1y, 2y, 5y
    excluded_tools=(),      # optional, e.g. ["get_company_financials"]
    max_revisions=2,        # evaluator-optimizer revisions per draft
)
show_run(run)               # report, charts, and delegation log
```

`run` is a dictionary holding the plan, routing decision, raw evidence,
analysis, news-chain trace, evaluation rounds, lessons used, and final report.
Lessons and score history are stored in `agent_memory.json`. They persist across
runs while that file remains available; storage in a temporary Colab runtime is
lost when the runtime is reset unless the file is saved elsewhere.

## Results of the Final Demonstration

Three end-to-end runs (AAPL, MSFT, and NVDA) completed successfully. All three
were routed by the LLM to both the market-data and news specialists and produced
final research reports with supporting charts.

The Evaluator changed two of the six specialist drafts. The AAPL news summary
improved from a mean score of 4.00 to 5.00 after one revision. The MSFT market
analysis also improved from 4.00 to 5.00 after one revision. The other four
drafts were approved on their first review.

By the end of the demonstration, four general lessons had been stored in
memory. Run 2 recalled one analysis lesson and one news lesson from Run 1, while
Run 3 recalled two analysis lessons and one news lesson accumulated from earlier
runs. Three runs are too few to establish that memory consistently improves
quality, but they demonstrate that evaluator feedback can be stored, recalled,
and applied in later analyses.

Temporary rate limits also occurred during evaluation, and the system
successfully used a fallback model. The rule-based routing fallback was not
needed in the three final end-to-end runs.

Full outputs are shown in Sections 10 and 11 of the notebook.

## Repository Contents

| File | Purpose |
| --- | --- |
| `Team2_FinalProject.ipynb` | Complete project notebook with executed outputs |
| `README.md` | Project overview, architecture, setup, usage, results, and limitations |

## Limitations

* Free LLMs are rate limited and can fail or vary between runs.
* Yahoo Finance data is not independently verified. The analysis uses a limited
  number of quarterly statements, which restricts longer-term comparisons.
* News coverage is limited to the titles and summaries returned by Yahoo
  Finance rather than full article text.
* Relevance, category, sentiment, extracted facts, and the final news summary
  are generated by LLMs. Prompts restrict these stages to retrieved source
  material and structured outputs are schema-validated, but extracted facts and
  summaries are not independently verified against the source text.
* The Evaluator is also an LLM. The automatic numeric screening can flag
  unsupported percentage magnitudes, but it cannot verify every factual claim
  or determine whether all financial reasoning is sound.
* The memory demonstration includes only three companies and stores lessons
  primarily by recency rather than relevance.
* The system supports research and evidence synthesis; it does not predict stock
  prices or provide investment advice.

## Code Style and Testing

Python code is written to follow PEP 8 conventions and uses comments and
docstrings to document the agent workflows and safeguards. Section 10.1 contains
offline unit tests for deterministic components that require no API key.

## Generative AI Use

OpenAI ChatGPT (GPT 5.6 Sol) was used as a development aid to review code and notebook
organization, identify potential inconsistencies between the documentation and
implementation, and suggest revisions for clarity and rubric alignment. All
suggestions were reviewed and modified by the team, and the team is responsible
for the final code, analyses, interpretations, and written content.

The OpenRouter-hosted LLMs used during execution are part of the implemented
multi-agent system and are documented throughout the notebook.