Design Write-Up — Agentic AI Engineer Intern Take-Home
What the system does

Given a company name, the agent produces a competitive-landscape brief by (1) planning a fixed sequence of research steps, (2) executing them with two tools — web search and a calculator — (3) recovering from an induced failure, and (4) emitting a structured JSON and Markdown report.

Design decisions

Rule-based planner over an LLM-based one. PlanningModule builds the step list with plain Python rather than prompting an LLM to generate it. This keeps the system runnable with zero API keys, makes the plan 100% deterministic and reproducible for grading, and keeps the planning logic easy to read and verify by inspection. The tradeoff is that the plan is templated per-goal-type rather than truly generative — it doesn't yet reason about which steps a given goal needs.

ddgs for web search instead of hand-scraping DuckDuckGo HTML. An earlier version parsed lite.duckduckgo.com's raw HTML with BeautifulSoup; this broke the moment DuckDuckGo changed its markup (see transcript history — every search returned "no results found" until the fix). Switching to the ddgs package decouples the tool from DuckDuckGo's page structure and made every subsequent run reliable.

A safe AST-based calculator instead of eval(). calculator_tool parses expressions with Python's ast module and only evaluates a whitelisted set of arithmetic operators (+ - * / **), so it can't execute arbitrary code — important once tool inputs may eventually come from an LLM rather than hardcoded strings.

Deliberate failure + retry with reformulation. Step 3 (the "recent news" search) is flagged to raise a simulated TimeoutError on its first attempt only. ExecutionModule catches any exception, logs it, waits with a short backoff, reformulates the query (e.g. appends the year) and retries up to max_retries times before marking the step failed and moving on rather than crashing the whole run. Steps are labeled success, recovered, or failed in the trace and in the final report, so the failure — and the recovery — are visible, not hidden.

Structured, dual-format output. ReportingModule builds one Python dict that's returned both as JSON (sources_collected, steps_failed, steps_recovered_via_retry, confidence_score) and as a rendered Markdown brief, so the same run produces both a machine-readable artifact and a human-readable one.

Limitations
The plan is a fixed 4-step template per goal; it doesn't adapt its steps based on what earlier steps returned (e.g. it doesn't search deeper if step 1 finds very little).
confidence_score is currently a static formula and doesn't yet reflect how many steps actually succeeded vs. failed in that specific run.
Only one failure mode is simulated (a timeout); the retry logic doesn't yet differentiate its recovery strategy by error type (e.g. a malformed-response error vs. a true timeout arguably call for different fixes).
ddgs is a free, unauthenticated search backend and can rate-limit under heavy or rapid use — fine for this assignment's scale, not production-grade.
What I'd do differently with more time
Replace the rule-based planner with an LLM call (Claude/GPT) that reads the goal and proposes its own step sequence and tool arguments, with the current rule-based planner kept as a deterministic fallback/validator.
Make confidence_score a genuine function of run outcomes (e.g. successes vs. failures, weighted by step importance) instead of a fixed formula.
Add a third tool (e.g. a structured API call or a file reader) to broaden orchestration coverage per the rubric's "tool use & orchestration" criterion.
Add lightweight unit tests for ExecutionModule's retry logic and calculator_tool's AST evaluator, and CI to run them on push.
