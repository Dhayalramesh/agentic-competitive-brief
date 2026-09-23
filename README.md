agentic-competitive-brief

A small agentic AI system that takes a company name and produces a structured competitive-landscape brief. Built for the Agentic AI Engineer Intern take-home assignment.

Given a goal like "produce a competitive-landscape brief for Stripe", the agent:

Plans a sequence of concrete, tool-using steps (visible before execution)
Executes each step, calling real tools (web search, calculator)
Self-corrects when a step fails — retries with a reformulated query and backoff
Reports a structured result as both JSON and Markdown
Architecture

See architecture_diagram.svg.

Goal → PlanningModule → ExecutionModule (tools + retry/self-correction loop) → ReportingModule → Report
PlanningModule — decomposes the goal into an ordered list of Step objects, each naming a tool and its arguments. Rule-based (no LLM/API key required), so it's deterministic and reproducible.
ExecutionModule — runs each step against the real tools. On failure, it logs the error, backs off, reformulates the query, and retries (up to max_retries) before marking the step failed and moving on.
Tools:
web_search_tool — live web search via the ddgs package
calculator_tool — safe arithmetic evaluation via Python's ast module (no eval())
ReportingModule — synthesizes the final structured report (JSON + Markdown) from the execution trace.
Setup
bash
git clone https://github.com/Dhayalramesh/agentic-competitive-brief.git
cd agentic-competitive-brief
pip install -r requirements.txt

Requires Python 3.9+.

Run
bash
python agent.py --company "Stripe"

Or in a notebook/Colab, run the cells in agentic_competitive_brief.ipynb in order, and change the company variable in the last cell to any company name.

The agent will:

Print a visible plan before acting
Print each tool call, its result, and any retries as it runs
Print the final Markdown report
Optionally save run_transcript.log, final_report.json, and final_report.md
Deliberately induced failure

Step 3 ("search for recent news") is seeded to raise a simulated TimeoutError on its first attempt only, to demonstrate the agent's error handling. Watch for this in the transcript — the agent logs the error, reformulates the query, retries, and recovers without crashing the run. This is marked recovered in the step status and called out in the final report.

Sample runs

See transcripts/ for 3 full sample runs (Anthropic, OpenAI, Stripe), each showing:

The visible planning trace
Two clean tool calls
One simulated failure + successful retry/recovery
The final structured report
Design write-up

See writeup.md for design decisions, limitations, and what I'd do differently with more time.

Limitations
The plan is a fixed template per goal rather than dynamically generated per-goal by an LLM
confidence_score is currently a static formula rather than derived from actual step outcomes
Only one failure mode (timeout) is simulated; recovery strategy doesn't yet vary by error type
ddgs is a free, unauthenticated search backend — fine at this scale, not production-hardened
Repo structure
.
├── agent.py                       # or agentic_competitive_brief.ipynb
├── requirements.txt
├── architecture_diagram.svg
├── writeup.md
├── transcripts/
│   ├── anthropic_run.log
│   ├── openai_run.log
│   └── stripe_run.log
└── README.md
