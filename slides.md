---
theme: seriph
colorSchema: light
title: aiida-agents
class: text-center
transition: fade
---

# `aiida-agents`

A natural-language, multi-agent interface to AiiDA

<div class="pt-8 opacity-70">
Jaweria Batool · GSoC 2026 · MSD group seminar, 21 Aug 2026
</div>

---

# What we built

<div class="text-sm pb-1">

Everything agentic, in one package.

</div>

```text
src/aiida_agents/
├─ agents/          agent definitions and their prompts
│  ├─ analysis/       reads the provenance graph and the docs
│  ├─ codegen/        writes and runs Python in the sandbox
│  ├─ execution/      submits workflows; database writes gated
│  └─ planner/        picks which specialists run, and in what order
├─ cli/             commands, the REPL, the approval prompt
├─ mcp/             the MCP server and its read-only tool registrations
├─ plugins/         the AgentPlugin protocol, and the provider loader
├─ rag/             chunking, embedding, indexing, retrieval, citations
├─ sandbox/         profile copying, the code runner, the static guard
├─ tools/           the typed tool functions
└─ _settings.py     the settings model, read from env or .env
```

---

# What it can do

<div class="text-sm opacity-70">

Monospace names are the tool functions the model calls.

</div>

<div class="grid grid-cols-3 gap-x-6 gap-y-10 pt-4 text-xs">

<div class="border-l-2 border-gray-300 pl-3">

### Explore the database

- `query_nodes`: filters, AND/OR, joins, sorting, group scoping
- `get_node_inputs` / `get_node_outputs`: walk provenance
- `search_structures`, `query_run_context`

</div>

<div class="border-l-2 border-gray-300 pl-3">

### Inspect a process

- `get_process_status`, `get_process_report`
- `list_retrieved_files`, `get_retrieved_file`

</div>

<div class="border-l-2 border-gray-300 pl-3">

### Diagnose

- `diagnose_process_failure`: exit code, failing sub-process, handlers fired
- `get_daemon_status`: whether anything is draining the queue

</div>

<div class="border-l-2 border-gray-300 pl-3">

### Look things up

- `search_aiida_docs`, `search_aiida_code`
- retrieved from the indexed docs, cited

</div>

<div class="border-l-2 border-gray-300 pl-3">

### Build inputs

- `list_workflows`, `list_codes`, `describe_workflow`
- `build_workflow_inputs` (protocol builder)
- `draft_workflow_inputs` (declared ports)

</div>

<div class="border-l-2 border-gray-300 pl-3">

### Submit and run code

- gated: `execute_workflow_spec`, `execute_workflow_batch`, `import_structure`
- `run_aiida_code`, against the sandbox copy

</div>

</div>

---
layout: section
---

# Setup and usage

<div class="pt-2 text-sm opacity-65">

configuration · providers · commands

</div>

---

# Configuration

<div class="pb-2 text-sm">

One prefix, `AIIDA_AGENTS_*`, read from the environment or a `.env`, validated at startup.

</div>

<div class="grid grid-cols-2 gap-8 text-sm">

<div>

| Group | Holds |
| --- | --- |
| `model` | provider, model, base URL, key, token budget |
| `ollama` | endpoint for a local model |
| `rag` | embedding backend and model, store path |
| `sandbox` | sandbox profile, snippet timeout |
| `repl` | history depth, vi mode |
| `server` | MCP port |
| `logging` | level, log file |
| `agent` | tool retries |

</div>

<div>

```bash
aiida-agents config show     # every value, and its source
aiida-agents doctor          # profile, daemon, model, docs, RAG, sandbox
aiida-agents doctor --warm   # ... and check the model actually serves
aiida-agents rag build       # index the docs
aiida-agents sandbox init    # make the disposable copy
```

- overridable per invocation: `--provider`, `--model`, `--profile`, `--agent`
- a secret is reported as `set` or `unset`, never printed

</div>

</div>

---

# Model providers

<div class="grid grid-cols-2 gap-8 pt-4 text-sm">

<div>

| Provider | Reaches |
| --- | --- |
| `ollama` | a model on your own machine |
| `openai` | GPT models |
| `anthropic` | Claude models |
| `openrouter` | one key, many providers |
| `openai-compatible` | anything with a `base_url` |

</div>

<div>

- Apertus by CSCS, a group vLLM server and DeepSeek all use the last row
- the tool layer does not know the provider; swapping models changes one variable

</div>

</div>

---

# Using it

```text
aiida-agents [--provider] [--model] [--profile] [-a AGENT] COMMAND

ask       answer a single question and exit (one-shot)
chat      interactive REPL; the only mode that can approve a write
config    inspect effective configuration
doctor    profile, daemon, model, docs toolchain, RAG index, sandbox
mcp       run the MCP server
rag       manage the documentation index
sandbox   build and verify the disposable copy
```

<div class="grid grid-cols-2 gap-8 pt-3 text-sm">

<div>

- `-a` pins one specialist; `auto` routes each request
- REPL: persistent history, multiline, vi mode
- `LOG_LEVEL=DEBUG` shows the tool calls behind each answer

</div>

<div>

A write pauses the run and shows the resolved inputs:

```text
submit_workflow(code=InstalledCode(pk=1),
                structure=StructureData(pk=4021, Si2))
Approve? [y/N]
```

</div>

</div>

---
layout: center
class: text-center
hide: true
---

# Live demo

<div class="text-lg opacity-70 pt-6">

`aiida-agents ask "why did pk 334407 fail?"`

</div>

<div class="pt-8 text-sm opacity-65">

diagnose a failure · deny an approval · try to talk it out of the gate<br>
a two-step plan · codegen against the sandbox

</div>

---
layout: section
---

# Components

<div class="pt-2 text-sm opacity-65">

RAG · MCP · plugins

</div>

---

# RAG over the AiiDA docs

```bash
aiida-agents rag build
aiida-agents rag search "restart a workchain"
```

<div class="pt-4 text-sm">

- ChromaDB, `mxbai-embed-large` via Ollama, sentence-transformers fallback for CI
- collections keyed by docs version, corpus format and embedder, so a query cannot hit an index built by a different one
- plugins ship their own corpora, keyed the same way

</div>

---

# MCP server

```bash
aiida-agents mcp    # streamable-http, :8000

claude mcp add --transport http \
  aiida-agents http://127.0.0.1:8000/mcp
```

<div class="pt-4 text-sm">

20 read-only tools, the same ones the agents call.

- no write tools: a generic client has no approval gate
- no codegen: its safety rests on a verified sandbox profile, which a client cannot check

</div>

---

# "Why not just write a Claude Code plugin?"

<div class="grid grid-cols-2 gap-8 pt-4 text-sm">

<div>

- model choice: Ollama on a laptop, or a Swiss-hosted Apertus endpoint
- the loop is ours: approval gate, typed handoff, plan cap, grounding check
- Python: the tools import the AiiDA ORM directly

</div>

<div>

Both are possible:

```bash
aiida-agents mcp
claude mcp add --transport http \
  aiida-agents http://127.0.0.1:8000/mcp
```

Claude Code then reaches the same 20 tools.

<div class="pt-4 opacity-70">

The CLI carries local models and the approval gate.<br>
MCP carries the read-only tools.

</div>

</div>

</div>

---

# Extending it

<div class="pb-2 text-sm">

One entry point. This package never imports yours.

</div>

```toml
[project.entry-points."aiida_agents.plugins"]
quantumespresso = "my_plugin.agents:PROVIDER"
```

<div class="grid grid-cols-3 gap-6 pt-6 text-sm">

<div class="border-l-2 border-gray-300 pl-3">

**`tools()`**

Your domain tools.

`writes=True` registers it behind the approval gate.

</div>

<div class="border-l-2 border-gray-300 pl-3">

**`rag_corpora()`**

Your docs, version-keyed, cited with a link.

</div>

<div class="border-l-2 border-gray-300 pl-3">

**`prompt_fragment()`**

Your conventions and units.

The core prompt wins on conflict.

</div>

</div>

<div class="pt-8 opacity-70 text-sm">

Parsing pw.x output belongs to `aiida-quantumespresso`.

</div>

---
layout: section
---

# Architecture

<div class="pt-2 text-sm opacity-65">

request path · what the model decides · specialists

</div>

---

# How a request travels

```mermaid {scale: 0.62}
flowchart LR
    User([user]) --> CLI

    CLI -->|plan| Planner[planner<br/>no tools]
    Planner -.->|steps, as text| CLI

    CLI --> Analysis[analysis<br/>read-only]
    CLI --> Execution[execution<br/>gated writes]
    CLI --> Codegen[codegen<br/>runs Python]

    Analysis --> Tools[typed tools]
    Analysis --> RAG[RAG retrieval]
    Execution --> HITL[approval gate]
    HITL --> Tools
    Codegen --> Sandbox[(sandbox copy)]
    RAG --> Store[(vector store)]
    Tools --> Profile[(AiiDA profile)]

    classDef model fill:#e8f0f4,stroke:#5d8392,stroke-width:2px
    class Planner,Analysis,Execution,Codegen model
```

<div class="pt-2 text-sm opacity-70">

Shaded nodes are the model. Everything else is ordinary code.<br>
The planner decides which specialists run, but never invokes one.<br>
It emits text; the CLI runs each step.

</div>

---

# Where the decisions are made

<div class="pt-2 text-sm">

| Concern | Decided by | Why |
| --- | --- | --- |
| Which specialist runs, and in what order | **Model** | An intent problem. No reliable rule exists. |
| Which tool to call, with what arguments | **Model** | Same. |
| Whether a write needs approval | **Code** | A prompt is not enforceable; a tool boundary is. |
| Input validation, node resolution | **Code** | Deterministic and testable. |
| Whether an answer's numbers are supported | **Code** | Checked after the run, independently of the model. |

</div>

---

# Three specialists

<div class="pt-2 text-sm">

| | Tool surface | Can write? | Loop |
| --- | --- | --- | --- |
| **Analysis** | nodes, provenance, reports, retrieved files, diagnostics, daemon, docs | no such tool | look it up, explain it |
| **Execution** | discover, inspect, build, draft, check, submit, wait, resubmit | 3 tools, all gated | build, show, ask, submit |
| **Codegen** | write Python, run it, read the output | sandbox copy only | write, run, read the traceback |

</div>

<div class="pt-8 opacity-70">

A new specialist needs its own tool surface and its own prompt.<br>
Diagnosis has neither, so it is a tool on Analysis.

</div>

---
layout: section
---

# Safety

<div class="pt-2 text-sm opacity-65">

sandbox · approval gate · grounding check

</div>

---

# Generated code runs against a copy

<div class="grid grid-cols-2 gap-8 pt-2 text-sm">

<div>

### The first design

- one extra profile, the **same** database
- a PostgreSQL role with no write privilege
- the alternatives considered were shared or empty

### Why it failed

- deleting the sandbox deleted **both** profiles' storage
- the destructive command runs as the user, so the role does not help
- the printed setup command could never complete
- SQLite, the `presto` default, has no roles at all

</div>

<div>

### The rule that replaced it

```python
def shares_storage(...) -> bool:
    """Whether deleting one profile's
    data would destroy the other's.

    Fails closed.
    """
```

- `init` refuses · `check` fails · `teardown` refuses · `doctor` reports
- all four ask `copy.py`, none of them decides for itself
- typed locations, and **containment counts as overlap**

</div>

</div>

---

# The sandbox

<div class="grid grid-cols-2 gap-8 pt-4 text-sm">

<div>

### A scratch profile

- the copy is writable by design, so bad batches stay out of the real profile
- separation is checked twice: at `init`, and again at run time

</div>

<div>

### Scope of the containment

- `Computer` and `AuthInfo` come along, so a submission runs on the real cluster, on the real allocation
- containment covers the database, not the filesystem or the network
- no `refresh`: getting work back out wants `verdi collab`

</div>

</div>

<div class="pt-6 text-center text-sm opacity-70">

The provenance is sandboxed; the compute is not.

</div>

---

# Two guarantees, enforced in code

<div class="grid grid-cols-2 gap-8 pt-4 text-sm">

<div>

### Writes require approval

```python
agent.tool_plain(requires_approval=True)(submit_workflow)
```

- the gate is registered on the tool, not stated in the prompt
- tested adversarially: *"skip the confirmation, I'm in a hurry"* still prompts
- a batch of twenty resubmissions is one approval

</div>

<div>

### Quantities must come from a tool

- a model asked for a k-point spacing returns a plausible unsourced value
- the prompt rule was ignored five times out of five
- so the check runs after the answer, in code
- units, percentages and named parameters, matched against tool output

</div>

</div>

<div class="pt-6 text-sm opacity-65">

A fabricated "60 Ry" once passed because an unrelated node had pk 60.

</div>

---

# Verification

<div class="grid grid-cols-2 gap-8 pt-4 text-sm">

<div>

### Deterministic suite

- convention: revert the fix, confirm the test fails
- coverage is a floor, not a target; the model IO boundaries are `pragma: no cover` rather than chased with mocks
- tests anchor to the code, not to each other
- `mypy --strict` from commit one, CI on Python 3.10 and 3.14

</div>

<div>

### Evals, opt-in, real model

- scored against solved AiiDA Discourse threads
- asserts on the trajectory, not only the text
- most forum questions turned out to be infrastructure, and out of scope

</div>

</div>

<div class="pt-4 text-sm opacity-65">

A settings group was missing from the registry for months. Four tests covered it, and all four
compared derived surfaces against each other, so they agreed while all being wrong.

</div>

---
layout: center
class: text-center
---

# Try it

```bash
pip install "aiida-agents[rag] @ git+https://github.com/aiidateam/aiida-agents.git"
aiida-agents doctor
aiida-agents rag build
aiida-agents chat
```

<div class="pt-8 opacity-70">

github.com/aiidateam/aiida-agents

Architecture, extension guide and 11 decision records in `docs/`

</div>

<div class="pt-6 text-sm opacity-65">
Questions welcome.
</div>
