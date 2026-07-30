---
layout: post
title: "The Evolution of Agentic AI"
date: 2026-07-30 23:10:00 +0530
---

# The Evolution of Agentic AI - A Journey

*Every architectural era is defined by where the bottleneck sits. When you fix a bottleneck, you don't get a better version of the old thing - you get a whole new discipline. That's exactly the arc Agentic AI has walked in about four years: four bottlenecks, four disciplines.*

Told through one running analogy - **a brilliant new hire** who starts by being handed a note and ends up running a factory floor.

![Alt text](/assets/images/2026-07-30-agentic-evolution/agentic-evolution.png)

---

## The journey at a glance

| Stage | Discipline | In a phrase |
|---|---|---|
| 1 | **Prompt engineering** | Say it better |
| 2 | **Context engineering** | Give it the right material |
| 3 | **Loop engineering** | Let it work, not just answer |
| 4 | **Graph engineering** | Design the org chart, not the employee |

---

## 1. Prompt Engineering - "say it better"

**The idea.** You've hired someone extraordinarily well-read but with total amnesia: they know everything about the world, nothing about *your* world, and forget it all the moment the conversation ends.

**The craft.** So at first you obsess over *how you word the note* you hand them:

- "You are an expert lawyer."
- Show three examples.
- "Think step by step."
- "Return JSON only."

Rephrasing the same request could swing quality dramatically - and that felt like magic.

**Why it stalled.** It's brittle: change a word and the output changes. And more fundamentally, no note is ever long enough to contain what the model needs to do a real job. The wording was never the bottleneck - the model's *ignorance of your business* was.

> **Bottleneck: Instructions (the note)**

---

## 2. Context Engineering - "give it the right material"

**The shift.** Stop polishing the note; start managing *what the model actually knows the moment it answers* - feeding it your documents (RAG), past context (memory), and live data (tools).

**The engineering problem.** The context window is a **small desk with a hard size limit**. Every decision is a trade-off: what goes on the desk, in what order, summarised how much, refreshed how often.

- Too little → it hallucinates.
- Too much → it gets distracted (and expensive).
- Anyone who's built a search-ranking pipeline recognises this instantly - it's relevance engineering wearing a new hat.

**Why it stalled.** You now get a very well-informed *single* answer. But real work isn't a single answer. It's an attempt, a mistake, a correction, another attempt.

> **Bottleneck: Single-shot output (one answer)**

---

## 3. Loop Engineering - "let it work, not just answer"

**The idea.** The model stops *replying* and starts *working*:

> think → act → observe → think again - until the job is done, or you stop it.

The employee is no longer answering a question. They've been handed a task: they'll open the file, run the query, notice a number looks wrong, re-run it, and come back when it's finished.

**What engineers build here.**

- Stopping conditions, step & token budgets
- Retries with backoff
- Idempotency for actions with side effects
- Sandboxing, so a bad step can't do real damage
- Evaluation of the whole *trajectory*, not just the final message

**Why it stalled.** One loop is a single thread - linear, opaque, hard to parallelise and govern. Over long runs the desk gets cluttered and the agent drifts. And when an auditor asks *"why did it approve that refund?"*, "it thought about it for 40 steps" is not an acceptable answer in an enterprise.

> **Bottleneck: Single thread of reasoning (one loop)**

---

## 4. Graph Engineering - "design the org chart, not the employee"

**The realisation.** Real work was never a loop - it's a **graph**. You stop trying to build one genius who does everything and start designing a *workflow*.

A planner (orchestrator) routes to a tool executor, a database lookup, and an evaluator that checks progress - with a human approval step before anything final.

- **Nodes** are stations: an agent, a function, a DB call, a human.
- **Edges** are routing rules. State travels along the edges.
- You get branching, parallel fan-out & fan-in, checkpoints, resume-ability, and humans as first-class steps.

**Two kinds of "graph engineering"** - people use one term for two things:

1. **Execution graph** (workflow topology): LangGraph & similar orchestration frameworks.
2. **Knowledge graph** (domain model): GraphRAG - entities & relationships so retrieval can traverse connections (e.g. "which contracts are affected by this vendor's change?").

> **Bottleneck: Un-governed, non-deterministic workflows that are hard to audit at scale**

---

## The big picture - it's a layering chain

Each stage didn't *replace* the last - it **absorbed** it.

| Stage | Unit of design | What it controls |
|---|---|---|
| 1. Prompt engineering | Sentence | What you ask |
| 2. Context engineering | Working set | What it knows |
| 3. Loop engineering | Trajectory | How it works |
| 4. Graph engineering | Topology | How it's routed |

- Prompts didn't die - they live *inside* the nodes.
- Context engineering didn't die - it decides what each node sees.
- Loops didn't die - a node with real autonomy still runs one internally.
- What changed at each step is the **unit of design** - from the sentence, to the working set, to the trajectory, and now to the topology.

---

## The honest tension at the frontier

Graphs give you governance and repeatability. But **over-specify the graph and you've rebuilt a rigid business-process workflow with an expensive LLM bolted on - you've paid for intelligence and then forbidden it.**

The current art is knowing which parts of your process deserve a **hard-wired edge** (predictable, auditable) and which deserve a **node that's allowed to think**.

---

### The one line to remember

The field didn't get better at prompting - it kept moving the problem *up a level*: from wording the request, to feeding the model, to letting it work, to architecting the whole system.
