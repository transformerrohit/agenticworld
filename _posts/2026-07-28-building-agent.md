---
layout: post
title: "Building your First Agent"
date: 2026-07-28 23:12:00 +0530
---

# Building Your First Agent - Fully On-Prem, Open Source

*A step-by-step guide. Everything runs on your own hardware - no API keys leave the building, no data goes to a cloud model. We climb one rung at a time: talk to the model -> give it a tool -> let it run the loop -> give it memory -> watch what it does -> put a UI on it.*

> **Stack:** Python · Ollama (local model) · LangChain · LangGraph · LangSmith · Open WebUI
> **Tested against:** LangGraph v1.0 (stable), `langchain-ollama`, current `LANGSMITH_*` tracing vars.

---

## The mental model

Each tool does **exactly one job**. Conflating them is the most common source of confusion:

| Tool | Its one job | Where it fits in the agent loop |
|---|---|---|
| **Ollama** | Runs the model locally, serves it at `localhost:11434` | The **brain** |
| **A tool-calling model** (`llama3.1`, `qwen2.5`, `mistral`) | Reasons and *decides* to call tools | The brain's actual weights |
| **LangChain** | Standard interface for models, tools, messages | The **glue** |
| **LangGraph** | Runs the perceive->reason->act->observe->repeat loop | The **orchestrator** |
| **LangSmith** | Traces every step for debugging & audit | The **eyes** (observability) |
| **Open WebUI** | ChatGPT-style frontend for the model / agent | The **face** |

> **The #1 catch:** the model *must natively support tool calling*. A chat model that doesn't will silently never call your tools. Stick to `llama3.1`, `qwen2.5` / `qwen3`, or `mistral`.

---

## Step 0 - Prerequisites & install

**Install Ollama** (macOS/Linux/Windows) from ollama.com, then pull a tool-capable model:

```bash
ollama pull llama3.1        # solid tool-caller, ~4.7GB
# ollama pull qwen2.5        # also excellent at tool calling
ollama serve                 # starts the local API at http://localhost:11434
```

**Set up Python:**

```bash
python -m venv .venv && source .venv/bin/activate
pip install -U langchain langchain-ollama langgraph langsmith
```

Sanity check the model is up:

```bash
ollama run llama3.1 "Say hello in five words."
```

---

## Step 1 - Talk to the local model (no agent yet)

Before anything clever, prove the brain works. This is the same thing Open WebUI does under the hood.

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434",
    temperature=0,
)

print(llm.invoke("Explain what an AI agent is in one sentence.").content)
```

**Important Point:** right now this is a *pure LLM* - smart, but no hands and no memory (rung 1 of the LLM->agent ladder). It can only produce text.

---

## Step 2 - Give it a tool (and watch it *decide* to use one)

Do **not** skip straight to the agent abstraction. Show the raw mechanic first, or students will treat the loop as magic.

Define tools with the `@tool` decorator. **The docstring is not optional** - the model reads it to decide when and how to call the tool.

```python
from langchain_core.tools import tool
from langchain_ollama import ChatOllama
from langchain_core.messages import HumanMessage

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two integers together.

    Args:
        a: the first integer
        b: the second integer
    """
    return a * b

@tool
def word_count(text: str) -> int:
    """Count the number of words in a piece of text.

    Args:
        text: the text to count words in
    """
    return len(text.split())

llm = ChatOllama(model="llama3.1", temperature=0)
llm_with_tools = llm.bind_tools([multiply, word_count])

response = llm_with_tools.invoke("What is 23 times 17?")
print(response.tool_calls)
# -> [{'name': 'multiply', 'args': {'a': 23, 'b': 17}, 'id': '...', 'type': 'tool_call'}]
```

Notice: the model didn't compute the answer - it **emitted a structured request** to call `multiply`. Now *you* run the tool and feed the result back, by hand, so the full round-trip is visible:

```python
tools_by_name = {"multiply": multiply, "word_count": word_count}
messages = [HumanMessage("What is 23 times 17?")]

ai_msg = llm_with_tools.invoke(messages)   # model decides
messages.append(ai_msg)

for call in ai_msg.tool_calls:             # we execute
    selected = tools_by_name[call["name"]]
    tool_msg = selected.invoke(call)       # returns a ToolMessage
    messages.append(tool_msg)

final = llm_with_tools.invoke(messages)    # model reads the result, answers
print(final.content)                       # -> "23 times 17 is 391."
```

**Important point:** *that four-step dance - decide -> execute -> feed back -> answer - is the entire agent loop.* Everything after this just automates it.

---

## Step 3 - Let LangGraph run the loop (this is the agent)

Now hand the loop to LangGraph's prebuilt ReAct agent. It does exactly what you just did by hand, repeatedly, until the task is done.

```python
from langgraph.prebuilt import create_react_agent
from langchain_ollama import ChatOllama

llm = ChatOllama(model="llama3.1", temperature=0)

agent = create_react_agent(
    model=llm,
    tools=[multiply, word_count],
    prompt="You are a helpful assistant. Use tools when they help.",
)

for chunk in agent.stream(
    {"messages": [{"role": "user",
                   "content": "What is 23 times 17, and how many words are in that question?"}]},
    stream_mode="updates",
):
    print(chunk)
```

You'll see it reason, call `multiply`, observe the result, call `word_count`, observe again, then compose a final answer - the **perceive -> reason -> act -> observe -> repeat** loop, automated. **This is the moment it becomes an agent.**

> `create_react_agent` lives in `langgraph.prebuilt`. If a tutorial you find online uses `set_entry_point()` or hand-rolls a `StateGraph` for a basic agent, it predates v1.0 - you don't need that for a simple agent.

---

## Step 4 - Give it memory

Right now each call starts from zero (a colleague with amnesia). Add a **checkpointer** and a `thread_id`, and the agent remembers within a conversation thread - this is short-term / working memory.

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.prebuilt import create_react_agent

checkpointer = MemorySaver()   # in-memory; swap for a DB-backed saver in production
agent = create_react_agent(
    model=llm,
    tools=[multiply, word_count],
    checkpointer=checkpointer,
)

config = {"configurable": {"thread_id": "user-123"}}

agent.invoke({"messages": [{"role": "user", "content": "My name is John."}]}, config)
resp = agent.invoke({"messages": [{"role": "user", "content": "What's my name?"}]}, config)
print(resp["messages"][-1].content)   # -> recalls "John"
```

**Important point:** the `thread_id` *is* the memory boundary. Same id -> same conversation. For persistence across restarts, swap `MemorySaver` for a database-backed checkpointer (Postgres/SQLite savers exist) - that's the jump from working memory to long-term memory.

---

## Step 5 - Turn on the eyes (LangSmith observability)

An autonomous system you can't see is a liability. LangSmith traces every reasoning step and tool call - no code changes needed, LangChain/LangGraph auto-instrument once the environment variables are set:

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=lsv2_...            # from smith.langchain.com
export LANGSMITH_PROJECT=onprem-agent-demo
export LANGSMITH_ENDPOINT=https://api.smith.langchain.com
```

Re-run any script above, then open the project in LangSmith. You'll see the full run tree: the prompt sent to the model, each tool call with its arguments and output, and the final answer nested inside.

**Important point:** this is the "log every tool call" discipline made real. When an agent misbehaves, the trace is the *only* way you'll reconstruct what it actually did.

> **On-prem note:** vanilla LangSmith is a hosted (cloud) service - the traces leave your network. For a truly air-gapped setup you have two honest options: **self-hosted LangSmith** (available on the enterprise plan, runs in your own cluster), or export traces via **OpenTelemetry** into an observability stack you already run. If "on-prem" is a hard requirement, decide this *before* you standardise on cloud LangSmith.

---

## Step 6 - Put a face on it (Open WebUI)

**6a - Chat with the raw model immediately.** Open WebUI is a ChatGPT-style UI that auto-detects Ollama:

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

Open `http://localhost:3000` and your local models are already in the dropdown. This gives you a friendly frontend for the **model** - but note it's talking to the raw model, *not your agent* (no tools, no memory logic).

**6b - Expose the *agent* through the same UI.** Open WebUI speaks the OpenAI API protocol, so wrap your LangGraph agent behind a minimal OpenAI-compatible endpoint:

```python
# agent_server.py - expose the LangGraph agent as an OpenAI-compatible endpoint
from fastapi import FastAPI
from pydantic import BaseModel
import time, uuid

from langgraph.prebuilt import create_react_agent
from langchain_ollama import ChatOllama
# from your_tools import multiply, word_count   # reuse the tools from Step 2

llm = ChatOllama(model="llama3.1", temperature=0)
agent = create_react_agent(model=llm, tools=[multiply, word_count])

app = FastAPI()

class Message(BaseModel):
    role: str
    content: str

class ChatRequest(BaseModel):
    model: str
    messages: list[Message]
    stream: bool = False

@app.get("/v1/models")
def list_models():
    return {"object": "list",
            "data": [{"id": "onprem-agent", "object": "model", "owned_by": "you"}]}

@app.post("/v1/chat/completions")
def chat(req: ChatRequest):
    result = agent.invoke({"messages": [m.model_dump() for m in req.messages]})
    answer = result["messages"][-1].content
    return {
        "id": f"chatcmpl-{uuid.uuid4().hex}",
        "object": "chat.completion",
        "created": int(time.time()),
        "model": req.model,
        "choices": [{
            "index": 0,
            "message": {"role": "assistant", "content": answer},
            "finish_reason": "stop",
        }],
    }
```

Run it and register it in Open WebUI:

```bash
pip install fastapi uvicorn
uvicorn agent_server:app --host 0.0.0.0 --port 8000
```

Then register it in Open WebUI (steps as of v0.9.x):

1. Avatar (bottom-left) -> **Admin Panel**
2. **Settings** tab -> **Connections**
3. In the **OpenAI API** section, click the **+** (plus) icon
4. **URL:** `http://host.docker.internal:8000/v1` - use `host.docker.internal`, not `localhost`, since Open WebUI runs in Docker and the shim runs on the host
5. **API Key:** any non-empty dummy string (e.g. `sk-noauth`) - the field is required but the shim doesn't check it
6. Click **Verify Connection**, then **Save**

Your agent - tools, memory, and all - now appears as a selectable "model" (`onprem-agent`) in the chat dropdown.

## Where this maps to the LLM->Agentic ladder

| What we added | Rung |
|---|---|
| Steps 1 | Raw **LLM** |
| Step 2-3 | **Tools** + the agent loop = **AI Agent** |
| Step 4 | **Memory** |
| Step 5 | **Governance & Observability** |
| Step 6 | Delivery surface |

---

## The pitfalls to call out

- **Model choice is destiny.** Small or non-tool-tuned models call tools poorly or not at all. Start with `llama3.1` or `qwen2.5`; only shrink the model once the logic works.
- **Docstrings are prompts.** The `@tool` docstring and type hints are what the model reads to decide *when* and *how* to call it. Vague docstring -> wrong tool calls.
- **Show the manual loop before the abstraction.** Step 2 by hand, *then* Step 3. Students who skip this treat `create_react_agent` as a black box.
- **`thread_id` is the memory key.** Reuse it to continue a conversation; change it to start fresh.
- **Set a recursion limit.** Agents can loop. `agent.invoke(..., {"recursion_limit": 10})` stops runaway reasoning.
- **Observability isn't optional for production.** If it can act autonomously, you must be able to see what it did - decide your on-prem tracing story early.
