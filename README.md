# AlphaBitCore Workflows & Virtual LLM Configurations

This repo contains **open-source workflow definitions** for the **AlphaBitCore (ABC)** platform: reusable, production-grade “virtual LLMs” built by composing models, tools, policies, and routing logic into **deterministic workflow graphs**.

If you’ve ever wished you could take “the way my best agent behaves” and ship it as a **versioned artifact** (with governance, auditability, and repeatability), that’s the core idea here.

---

## What is AlphaBitCore?

**AlphaBitCore** is an orchestration and governance platform for agentic AI in real-world environments—especially where you need:

- **Repeatable behavior** (not prompt roulette)
- **Controls & guardrails** (policy, compliance, data boundaries)
- **Auditability** (what ran, why it ran, what it produced)
- **Composability** (build higher-level capabilities from smaller blocks)
- **Deployment flexibility** (chat agent, API, background workflow, or embedded in other systems)

ABC treats “agent behavior” as **software**: it can be reviewed, tested, versioned, promoted through environments, and monitored.

---

## Core concepts

### 1) Workflows
A **workflow** is a directed graph of steps (nodes) connected by dependencies (edges). Each node performs a single responsibility (classify, extract, analyze, synthesize, format, call a tool, etc.).

ABC workflows are designed to be:

- **Minimal**: only the nodes needed for the job
- **Dependency-driven**: nodes run because upstream outputs require them
- **Guarded**: nodes can skip themselves based on conditions (“Execution guard”)

### 2) Composite tools
A **composite tool** is a named, reusable capability built from a workflow. It looks like a single “tool” to the caller, but internally runs a full graph.

Examples:
- `MarketingComplianceReview`
- `EarningsCallSummarizer`
- `RFPResponseGenerator`
- `PolicyAwareWebResearcher`

Composite tools are ideal when you want standardized behavior across many apps/teams.

### 3) Virtual LLMs
A **virtual LLM** is a workflow (or composite tool) that is exposed **as if it were a model**.

To a client, it behaves like “call a model and get a response,” but inside it can:
- Route to different underlying models by task type
- Enforce policies and guardrails
- Retrieve context via tools
- Run specialized analyzers
- Synthesize and format consistently

This lets you ship “the model” as **an opinionated, governed system** rather than “whatever the base model decides today.”

### 4) Execution guards (why they matter)
ABC supports conditional execution so nodes can “opt out” unless needed.

Example patterns:
- Skip heavy analysis unless the request meets criteria
- Only run web research when a classifier says it’s needed
- Only run risk checks when a strategy mentions certain assets

This keeps workflows **fast, cheap, and reliable** while still being comprehensive when required.

---

## What’s in this repo?

This repository is organized around real, reusable workflow artifacts. Each JSON file typically represents one workflow / composite tool / virtual LLM configuration.

### Included example: Marketing Compliance Review Workflow
This repo includes a workflow that performs **financial-services marketing/advertising compliance review** with guarded execution, specialized analyzers, synthesis, and strict JSON output formatting.

At a high level, that workflow:
1. **Classifies** whether the input is actually “marketing content to review” (vs. asking for advice)
2. **Fast-fails politely** when it’s not a review request
3. **Analyzes complexity & asset class** to decide which analyzers to run
4. Runs targeted analyzers (e.g., prohibited statements, performance claims, risk disclosures, testimonials, recordkeeping)
5. **Synthesizes** results into a coherent compliance review
6. **Formats** the final output as a strict discriminated-union JSON response

This is a good reference workflow for:
- Guard patterns (“only run when…”)
- Multi-analyzer designs
- Final “formatter” nodes that guarantee schema correctness

---

## How to use these configurations

### Option A — Import into AlphaBitCore
1. Deploy ABC (or use an existing ABC environment).
2. Import a workflow JSON from this repo into your registry.
3. Promote it as:
   - a **workflow** (run directly),
   - a **composite tool** (callable by agents/workflows),
   - or a **virtual LLM** (callable like a model).

### Option B — Call as a “Virtual LLM” from any OpenAI-compatible client
If your ABC deployment exposes virtual LLMs via an OpenAI-compatible API surface, you can:
- configure the base URL to ABC,
- select the virtual model name,
- call `chat.completions` (or equivalent),
- and receive consistent outputs backed by the workflow’s governance logic.

### Option C — Use inside other workflows
Workflows can call other workflows (via composite tools / virtual models), enabling “Lego bricks” for AI capability:
- Router workflows call specialized sub-workflows
- Compliance gates wrap content-generation flows
- Data extraction flows feed summarization flows

---

## Design philosophy (what to expect from these workflows)

These configs aim for:
- **Separation of concerns**: classify → analyze → synthesize → format
- **Deterministic outputs**: stable JSON schemas, consistent fields
- **Safe defaults**: reject/route/guard when unclear
- **Extensibility**: add analyzers without rewriting the whole graph
- **Operational realism**: built to be monitored, logged, and versioned

---

## Contributing

PRs are welcome—especially for:
- New workflow patterns (routing, evaluation, tool gating)
- Better schemas and output formatters
- More “reference-grade” workflows (compliance, finance, enterprise docs, support)
- Test prompts / golden outputs for regression testing

When contributing, prefer:
- Minimal nodes
- Explicit guards
- Clear schema contracts
- Versioned changes (treat workflows like code)

---

## License

This project is licensed under the **Apache License 2.0**. See the `LICENSE` file for details.

---

## Disclaimer

These workflows are provided as examples and building blocks. They may require adaptation for your environment, data, policies, and regulatory obligations. Where workflows touch compliance or regulated workflows, they should be reviewed by qualified professionals before production use.

---

## Quick glossary

- **Workflow**: A dependency graph of nodes that executes to produce an outcome
- **Composite tool**: A workflow packaged as a single callable tool
- **Virtual LLM**: A composite tool exposed as a “model” interface
- **Execution guard**: A condition that determines whether a node runs or skips
- **Analyzer**: A specialized node/tool that checks one dimension (risk, performance claims, etc.)
- **Synthesis**: Aggregation + decision logic + final structured result
