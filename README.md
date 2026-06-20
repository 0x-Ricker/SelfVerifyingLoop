# The Self-Verifying Loop

### 300 agents. 4,000 steps. 5 live data feeds. Zero hallucinated numbers shipped.

> Most agent swarms don’t produce intelligence.
> They produce **confident garbage at scale**.
>
> This one doesn’t stop until every number survives verification against a live source.

---

<p align="center">
  <b>Planner / Verifier:</b> Opus 4.8 &nbsp;&nbsp;•&nbsp;&nbsp;
  <b>Execution Swarm:</b> Kimi K2.6 &nbsp;&nbsp;•&nbsp;&nbsp;
  <b>Scale:</b> 300 parallel agents / 4,000 steps / 100-company research jobs
</p>

---

## What this repo is

This repo is a pattern for building **research-grade agent swarms** that don’t just generate output — they **check their own work, reject failures, and rerun until the report is clean**.

Most multi-agent systems have one fatal flaw:

* they optimize for **throughput**
* they optimize for **parallelism**
* they optimize for **wow-factor demos**

…but they don’t optimize for **truth**

That’s why swarms are fast and still unusable for serious research.
They can produce 100 company writeups in minutes — and quietly hallucinate 12 of them.

This repo fixes that by treating the swarm as **one stage in a loop**, not the final answer.

---

# The idea in one sentence

> **A swarm gives you speed. A verifier gives you trust. A loop gives you both.**

---

# Why this exists

The dirty secret of agent swarms is that **more agents usually means more confident nonsense**.

If you point 300 agents at a research job, they will absolutely come back fast.

They will also come back with:

* stale numbers
* fake or broken citations
* missing fields
* inconsistent financial figures
* companies that don’t exist
* outputs that *look* polished enough to fool you

And that’s the real problem: **bad swarm output often looks identical to good swarm output** until someone catches it manually.

So instead of asking:

> “How do I make the swarm bigger?”

this repo asks:

> **“How do I make the swarm unable to ship bad work?”**

---

# The architecture

## The Self-Verifying Loop

```text id="lu3j0s"
┌────────────────────┐
│ 1. PLAN            │
│ Opus 4.8 breaks    │
│ the job into tasks │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ 2. EXECUTE         │
│ Kimi K2.6 swarm    │
│ runs tasks in      │
│ parallel           │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ 3. VERIFY          │
│ Opus checks every  │
│ output against the │
│ live source cited  │
└─────────┬──────────┘
          │
     fail │ pass
          │
          ▼
┌────────────────────┐
│ 4. REQUEUE         │
│ Failed tasks go    │
│ back into swarm    │
│ with rejection     │
│ reason attached    │
└─────────┬──────────┘
          │
          └──── repeat until verify = clean
```

The loop only stops when **nothing fails verification**.

That’s the whole system.

Not “generate once and hope.”
Not “manually spot-check 100 rows.”
Not “the model sounded confident.”

**Verify. Reject. Requeue. Repeat.**

---

# Why raw swarms break

A raw swarm has exactly one quality level:

> **whatever the worst agent produced**

If 97 agents get their company right and 3 hallucinate revenue numbers, the final report still contains 3 landmines.

And worse:

* the report still looks complete
* the table still looks polished
* the output still sounds authoritative
* nobody knows which 3 rows are wrong until it matters

That’s why “just add more agents” doesn’t solve the problem.

It scales:

* output volume
* speed
* task coverage

…and also scales:

* hallucinations
* stale data
* citation failures
* hidden errors

Without verification, a swarm is just a **mistake multiplier with better UX**.

---

# What makes this different

This system turns verification into a **first-class stage with real teeth**.

Every agent output is checked against the source it claims to use.

If a result fails, it does **not** get “flagged for later.”
It gets **rejected immediately** and sent back to rerun.

## Verification rules

Every company/task must pass a checklist like:

* revenue pulled from a live source
* margin pulled from a live source
* source URL attached
* source URL resolves
* cited figure matches source within tolerance
* no required field left empty
* output schema is complete

If any of those fail, the task goes back into the queue.

No exceptions. No “close enough.” No human babysitting.

---

# The test run

To stress-test the loop, I gave it a job that punishes hallucination harder than almost anything:

> **Analyze 100 companies in the EV market and generate a research-grade report + comparison matrix with every figure traced to a live source.**

## Stack used

* **Planner / verifier:** Opus 4.8
* **Execution swarm:** Kimi K2.6
* **Live data feeds:** 5
* **Parallel agents:** 300
* **Total workflow steps:** ~4,000

---

# What happened

## Pass 1

* **100 companies checked**
* **88 passed**
* **12 rejected**

Failures included:

* revenue figures that didn’t match the cited source
* citations that didn’t resolve
* missing margin fields

## Pass 2

* **3 still failed**

## Pass 3

* **0 failed**

Loop stops automatically.

A normal swarm would have shipped all 12 errors and called it a success.

This loop caught them without me reading a single row.

---

# Example verifier output

```json id="q4fb54"
{
  "pass": 1,
  "checked": 100,
  "passed": 88,
  "rejected": [
    { "company": "co_041", "reason": "revenue != source" },
    { "company": "co_067", "reason": "citation 404" },
    { "company": "co_092", "reason": "margin empty" }
  ],
  "action": "requeue rejected -> swarm"
}
```

---

# Why verification actually works here

Because it’s grounded in **live feeds**, not model self-confidence.

The verifier is not asking:

> “Does this output feel plausible?”

It is asking:

> **“Does the number in this row match the source URL the agent attached?”**

That difference is everything.

## Example live sources used

* Binance
* Yahoo Finance
* World Bank
* IMF
* live stock market data

Once every figure has a live source behind it, verification stops being fuzzy and becomes mechanical.

That’s the line between:

* **“AI-generated report”**
* and **“research pipeline”**

---

# Repo philosophy

## Swarm = execution layer

## Loop = trust layer

Most people are building better swarms.

The more important thing is building **a system that assumes the swarm will fail** and is designed to catch it.

This repo is opinionated about one thing:

> **Generation should never be the final step in a high-stakes workflow.**

The final step should be **verification**.

---

# Core design principles

## 1. The swarm is not the product

The swarm is just the first draft engine.

If you treat the first pass as the answer, you are shipping hallucinations with better formatting.

---

## 2. Verification must be rejectable

“Looks good” is not verification.

A good verify layer uses checks like:

* source resolves / doesn’t resolve
* field present / missing
* number matches / doesn’t match
* date fresh / stale
* schema complete / incomplete

If the rule can’t reject work cleanly, it’s not strong enough.

---

## 3. Failed work should rerun automatically

Humans should not have to manually inspect every row and patch bad outputs one by one.

If the system knows *why* a task failed, it should be able to requeue that task automatically.

That is where the leverage comes from.

---

## 4. Quality should equal the checklist, not the model mood

Raw LLM quality is unstable.

A checklist is stable.

The goal is to move the system from:

* “I hope the model got it right”

to:

* **“The task cannot exit unless it passes objective checks.”**

---

# Raw swarm vs self-verifying loop

## Raw swarm

* ❌ Runs once and returns whatever came back
* ❌ Hidden errors ship with the report
* ❌ Quality equals the worst agent
* ❌ Manual auditing becomes mandatory
* ❌ Confidence gets mistaken for correctness
* ❌ More agents = more ways to be wrong faster

## Self-verifying loop

* ✅ Runs until the verify stage is clean
* ✅ Failed rows are rejected automatically
* ✅ Rejected tasks rerun automatically
* ✅ Quality is enforced by the checklist
* ✅ Every figure traces back to a live source
* ✅ Human review becomes optional instead of required

---

# Minimal mental model

If you only remember one thing, remember this:

```text id="3dbfc7"
swarm = produce output fast
verifier = check output against reality
loop = keep going until reality wins
```

---

# How to implement it

You do **not** need a frontier lab to build this pattern.

You need three parts.

## 1) Planner / verifier model

A model that can:

* break a job into structured subtasks
* evaluate completed tasks against a strict checklist
* produce rejection reasons cleanly enough to rerun

## 2) Parallel execution swarm

A model / agent layer that can:

* process many tasks concurrently
* return consistent structured outputs
* be rerun cheaply for failed tasks

## 3) Source-aware verification contract

You need a schema that forces every task to return:

* structured fields
* source URLs
* timestamps if relevant
* enough information for the verifier to compare claim vs source

Without that, “verification” collapses into subjective review.

---

# Suggested workflow

## Step 1 — Define the output contract

For each task, require fields like:

```json id="j6lb9v"
{
  "company": "Tesla",
  "revenue": "97.7B",
  "gross_margin": "17.9%",
  "market_cap": "X",
  "source_urls": [
    "https://...",
    "https://..."
  ],
  "notes": "..."
}
```

## Step 2 — Define failure conditions

Example:

* missing source URL
* URL 404s
* numeric mismatch vs source
* stale date
* empty required field
* malformed output schema

## Step 3 — Run the swarm

Generate outputs in parallel.

## Step 4 — Verify every result

Do not sample.
Do not spot-check.
Check **everything**.

## Step 5 — Requeue only failures

Do not rerun the entire job.
Rerun the rejected tasks with the failure reason attached.

## Step 6 — Repeat until clean

Loop ends only when the rejection queue is empty.

---

# Where this matters most

This pattern is most useful anywhere hallucinations are expensive.

## Finance

* company research
* market maps
* earnings comparisons
* valuation / multiples tracking
* source-linked investment memos

## Consulting

* competitive landscapes
* market sizing
* benchmarking reports
* slide-ready research pipelines

## Research / academia

* literature reviews
* comparison matrices
* source-backed synthesis
* citation-heavy analysis

## Internal ops

* vendor comparisons
* procurement research
* strategy briefs
* automated due diligence

---

# Why Kimi K2.6 is such a strong fit

Kimi K2.6 matters here because the swarm layer needs to do three things well:

## 1. Handle large research jobs

100 companies, large tables, long documents, many parallel tasks.

## 2. Produce structured outputs consistently

Verification only works if the swarm returns machine-checkable output.

## 3. Stay useful under scale

This pattern gets stronger when you can process lots of work at once and cheaply rerun failures.

The loop is the safety layer.
The swarm still needs to be good enough to make the loop worth running.

---

# The bigger point

Everyone is racing to build bigger agent swarms.

That’s not the interesting part anymore.

The interesting part is what happens **after** generation.

The next generation of AI systems won’t be judged by:

* how many agents they launched
* how cinematic the orchestration graph looks
* how fast they can fill a dashboard with text

They’ll be judged by one much simpler question:

> **What stops bad output from shipping?**

If the answer is “a human notices it eventually,” the system is not finished.

---

# TL;DR

**The Self-Verifying Loop** is a pattern for turning agent swarms into something you can actually trust.

## It works like this:

* **Opus 4.8** plans and verifies
* **Kimi K2.6** executes in parallel
* every output is checked against live sources
* failures are rejected automatically
* rejected tasks rerun automatically
* the workflow stops only when verification is clean

## Result:

* 300 parallel agents
* ~4,000 workflow steps
* 5 live data feeds
* 100-company EV market report
* 12 bad outputs caught on pass 1
* 3 caught on pass 2
* 0 failures by pass 3

---

# Final takeaway

A raw swarm gives you **speed**.

A self-verifying loop gives you **speed you can trust**.

And that’s the difference between:

* a cool AI demo
* and a system you can actually use for serious research
