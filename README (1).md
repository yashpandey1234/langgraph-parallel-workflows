# LangGraph Parallel Workflows — Fan-Out/Fan-In & Structured LLM Evaluation

A follow-up learning repo exploring **parallel execution** in LangGraph — multiple nodes running independently from the same start point and converging into a single aggregation node — applied to both a deterministic (cricket stats) workflow and an LLM-powered (essay evaluation) workflow.

## 📌 Why this repo

The earlier fundamentals repo covered *sequential* graphs (one step after another). Real workflows are rarely linear — often several independent computations or LLM calls need to happen at once and then get combined into one final result. This repo documents learning that **fan-out/fan-in** pattern in LangGraph.

## 🧠 Concepts Learned

- **Fan-out / Fan-in graphs** — connecting `START` to multiple independent nodes, then converging all of them into one shared downstream node (`add_edge(START, nodeA)`, `add_edge(START, nodeB)` ... all leading to a common `summary`/`final_evaluation` node).
- **Parallel state updates** — multiple nodes writing to the *same* state object independently and safely, since each node only returns the keys it's responsible for.
- **Reducers via `Annotated` + `operator.add`** — using `Annotated[list[int], operator.add]` so that when multiple parallel nodes each append a score to the same state key, LangGraph merges them correctly instead of overwriting.
- **Structured output with Pydantic** — using `model.with_structured_output(EvaluationSchema)` so the LLM returns a strictly-typed object (`feedback: str`, `score: int` with validation bounds `ge=0, le=10`) instead of free-form text that needs manual parsing.
- **Multi-criteria evaluation pattern** — running several independent LLM evaluations (language quality, depth of analysis, clarity of thought) in parallel, each scoring the same input differently, then aggregating into one final score + synthesized feedback.
- **Aggregation/synthesis node** — a final node that both computes a numeric result (average score) *and* makes a further LLM call to synthesize multiple text feedbacks into one coherent summary.
- **Reusing a compiled graph on multiple inputs** — invoking the same `workflow` on different data (two different essays) without recompiling.

## 🛠️ Tech Stack

| Category | Tools/Libraries |
|---|---|
| Orchestration | [LangGraph](https://pypi.org/project/langgraph/) |
| LLM Framework | [LangChain](https://python.langchain.com/) (`langchain-core`, `langchain-groq`) |
| LLM Provider | [Groq API](https://groq.com/) — `llama-3.3-70b-versatile` |
| Structured Output | [Pydantic](https://docs.pydantic.dev/) (`BaseModel`, `Field` with validation) |
| Language | Python 3.11+ |
| Env Management | `python-dotenv` |
| Notebook Environment | Jupyter Notebook |

## 📂 Notebooks in this repo

| Notebook | What it demonstrates |
|---|---|
| `4_batsman_workflow.ipynb` | A **deterministic parallel** graph: from a single batting scorecard (runs, balls, fours, sixes), 3 independent nodes each compute one metric (strike rate, balls-per-boundary, boundary %) in parallel, then a `summary` node combines all three into one report. |
| `5_UPSC_Essay_workflow.ipynb` | An **LLM-powered parallel evaluation** graph: 3 independent LLM calls each evaluate a UPSC-style essay on a different dimension (language, analysis depth, clarity of thought) using structured Pydantic output, then a `final_evaluation` node averages the scores and generates a synthesized overall feedback. |

## 🚀 Real-World Problems This Pattern Can Solve

The **parallel fan-out/fan-in + multi-criteria scoring** pattern generalizes to any situation where something needs to be judged on several independent dimensions at once, or where several independent computations feed one final report:

- **Automated essay/exam grading** — direct extension of the UPSC notebook: grading assignments, competitive exam answers, or writing samples across multiple rubrics (grammar, structure, argument quality) simultaneously.
- **Resume/candidate screening** — parallel evaluation of a resume on skills-match, experience relevance, and communication quality, then an aggregated hiring-recommendation score.
- **Product/content review scoring** — analyzing a product description or review across sentiment, factual accuracy, and policy-compliance in parallel before publishing.
- **Sports/performance analytics dashboards** — direct extension of the batsman notebook: computing multiple independent KPIs (strike rate, economy rate, conversion rate, etc.) from one dataset and merging into a single performance report — applies equally to sales dashboards, fitness trackers, or financial ratios.
- **Multi-model or multi-agent consensus systems** — running the same query through multiple LLM prompts/personas in parallel (e.g., "optimist", "skeptic", "fact-checker") and combining their outputs into one balanced answer.
- **Compliance/risk scoring** — evaluating a document or transaction against multiple independent risk criteria (fraud risk, regulatory compliance, sentiment risk) in parallel, then combining into a single risk score.
- **A/B content quality checks** — evaluating multiple pieces of generated content (ad copy, emails) on tone, clarity, and persuasiveness simultaneously before choosing the best version.
- **Structured data extraction pipelines** — using Pydantic-validated LLM output to reliably extract multiple fields from unstructured text (invoices, medical notes, legal documents) without brittle string parsing.

The common thread: **whenever a decision depends on several independent signals that can be computed simultaneously and then combined**, this fan-out/fan-in + structured-output pattern is the right fit.

## ⚙️ Setup

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create a `.env` file in the root (use `.env.example` as a template) and add your own key:

```
GROQ_API_KEY="your-groq-api-key-here"
```

Then run any notebook via Jupyter:

```bash
jupyter notebook
```

## 📦 requirements.txt

```
langgraph
langchain-core
langchain-groq
python-dotenv
pydantic
```

## 🗺️ Next Steps / Roadmap

- [ ] Add conditional/dynamic fan-out (e.g., number of evaluation criteria decided at runtime)
- [ ] Add a weighted scoring system instead of a simple average
- [ ] Persist evaluation history (checkpointing) to track score trends over time
- [ ] Add a human-in-the-loop review step before finalizing scores
- [ ] Extend the essay evaluator into a full UPSC mentor tool with topic-wise feedback trends

---

*This repo is a continuation of my LangGraph learning log, focused on parallel workflow patterns and structured LLM outputs.*
