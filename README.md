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
pip install -r Requirements.txt

Requires Python 3.9+. Designed to run in Google Colab (each notebook installs its own dependencies in its first cell) or any local Jupyter environment.

Run

Each notebook is a full, independently runnable copy of the agent, pointed at a different company — open any one of them and run all cells top to bottom:

Anthropic.ipynb — runs the agent against "Anthropic"
openai.ipynb — runs the agent against "OpenAI"
stripe.ipynb — runs the agent against "Stripe"

To try a different company, open any notebook, find the line:

python
company = "Stripe"   # change to any company name

change the string, and re-run all cells.

Each run will:

Print a visible plan before acting
Print each tool call, its result, and any retries as it happens
Print the final Markdown report
All output is saved inline in the notebook itself, so you can open any .ipynb on GitHub and see the full transcript without re-running it
Deliberately induced failure

Step 3 ("search for recent news") is seeded to raise a simulated TimeoutError on its first attempt only, to demonstrate the agent's error handling. Watch for this in any of the notebook outputs — the agent logs the error, reformulates the query, retries, and recovers without crashing the run. This is marked recovered in the step status and called out in the final report.

Sample runs

Anthropic.ipynb, openai.ipynb, and stripe.ipynb each contain a full sample run with saved output, showing:

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
├── Anthropic.ipynb        # sample run: Anthropic
├── openai.ipynb           # sample run: OpenAI
├── stripe.ipynb           # sample run: Stripe
├── Requirements.txt
├── architecture_diagram.svg
├── writeup.md
└── README.md
