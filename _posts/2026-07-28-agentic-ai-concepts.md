---
layout: post
title: "Agentic AI Concepts"
date: 2026-07-28 19:28:00 +0530
---

# Agentic AI - The Big Picture

*A plain-language walkthrough of the core concepts: what an agent is, how it thinks and works, and what makes it reliable in production.*

![Alt text](/assets/images/2026-07-28-agentic-ai-concepts/agentic-concepts.png)

---

## 1. Agentic AI - Concepts

An LLM on its own just answers questions. An *agent* is what you get when you point that brain at a goal and let it act on its own: it looks at the situation, thinks, makes a plan, does something, checks the result, and adjusts - with barely any hand-holding.

The core loop is the whole idea:

> **perceive -> reason -> act -> learn -> repeat**

**Why now?** Because the brains (LLMs) finally got good enough at reasoning, the tools to wire them up matured, and businesses genuinely want work done without a human in every loop.

---

## 2. Agent Architecture - Building Blocks

This is what's actually *inside* an agent. Inputs come in (a user, sensors, data, events), and the agent runs them through five parts:

- **Perception** - understand what's being asked
- **Reasoning** - the LLM decides what to do
- **Planning** - break it into steps
- **Memory** - hold context, short and long term
- **Action / Tools** - actually go do it

Underneath sits a knowledge store it can pull facts from. This panel is just naming the organs of the thing - the same loop from panel 1, drawn as machinery.

---

## 3. Types of Agents

Not all agents are equally clever, and that's on purpose - you match the type to the job.

| Type | What it does | Good for |
|---|---|---|
| **Reactive** | Responds to current input, no memory | Simple rules; fast |
| **Conversational** | Remembers the chat | Support, Q&A |
| **Goal-based** | Plans toward an outcome | Multi-step tasks |
| **Multi-agent (MAS)** | Several specialized agents cooperating | Complex, divided work |
| **Autonomous** | Runs with almost no oversight | Monitoring, self-driving processes |
| **Hybrid** | Mixes rules + LLM + ML | Reliability + flexibility |

**The lesson:** more autonomy isn't always better - pick the simplest type that does the job.

---

## 4. Agent Workflow - Deep Dive

This zooms into one full pass of the loop, step by step:

1. **Goal** - get the objective from a user or system
2. **Perception** - gather context from memory, tools, environment
3. **Reasoning** - figure out intent and constraints
4. **Planning** - break the goal into actionable steps
5. **Action** - execute a step using tools / APIs
6. **Observation** - look at the result and feedback
7. **Reflection & Learning** - update memory, improve the plan

...then repeat until the goal is achieved.

Made concrete: the goal is *"analyze sales data and email me insights,"* so it fetches the data, thinks, plans four steps, runs Python, checks *"was the email actually sent?"*, and stores what it learned.

**The observation and reflection steps are what separate a real agent from a script** - it notices when something went wrong and corrects.

---

## 5. Memory in Agentic AI

Memory is what turns a goldfish into a colleague.

- **Short-term (working)** - the current conversation; lives in the context window
- **Long-term (persistent)** - stored outside the model in databases, survives across sessions
- **Episodic** - "what happened last time"; past experiences it can learn from
- **Semantic** - plain facts and domain knowledge

Because you can't cram everything into the context window, there are strategies to manage it: **summarize** old chats, **embed + retrieve** only the relevant bits, use a **memory graph** of related facts, or a **hybrid** of these.

**This is usually where real agent projects succeed or fail** - get memory wrong and the agent feels forgetful and unreliable.

---

## 6. Tools & Actions

An agent that can only talk is a chatbot; tools are what let it *do things in the real world* - call APIs, run code, search, read/write databases, send Slack messages, drive a browser.

The pattern is always the same:

> **LLM decides it needs a tool -> calls it with inputs -> gets the output -> continues**

**Best practices** (the part experience teaches you the hard way):

- Describe your tools clearly, with proper schemas
- Validate inputs and handle errors
- Limit what the agent is allowed to touch, for safety
- **Log every single tool call** - because when an autonomous system misbehaves, the logs are the only way you'll ever understand what it did

---

## The thread tying it together

| Panels | Theme |
|---|---|
| 1-2 | **What an agent is** |
| 3-4 | **How it thinks and works** |
| 5-6 | **What makes it reliable in production** |

Memory so it doesn't forget, and disciplined tool use so it doesn't go off the rails - that's what carries an agent from an impressive demo to something you can actually trust in production.
