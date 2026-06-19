# SECA — Structured Exploration and Control Architecture

> **Structure decides the path. Not the model.**

SECA is a model-agnostic control architecture for running LLM systems in production. It makes LLM behavior **reproducible, traceable, and accountable** by separating what is structural (fixed) from what is evaluative (adaptive).

---

## The Problem

LLMs are non-deterministic. The same input can produce different actions on different runs. In a chatbot, that is acceptable. In business operations, it creates three failure modes that block production deployment:

- **Behavior drifts.** The agent handled a refund correctly yesterday and incorrectly today. Nothing in your code changed — the model simply chose a different path.
- **A stop becomes an outage.** When the agent's reasoning is the control flow, an unexpected model output can halt the entire process. The operation stops with it.
- **Results can't be explained.** A regulator, a customer, or your own audit team asks *why* the system did what it did. "The model decided" is not an acceptable answer.

These are not prompt-tuning problems. They are **architectural** problems. You cannot make a system accountable by writing a better instruction — because the instruction is still interpreted by a non-deterministic model on every run.

---

## The Core Idea

SECA splits the system into two parts that are governed differently:

| | What it is | How it changes |
|---|---|---|
| **Structure** | The path the system is allowed to take | **Static** — fixed, defined outside the model |
| **Evaluation** | What the system prioritizes right now | **Dynamic** — adapts over time, from outside the structure |

The model operates *inside* the structure. It chooses among options the structure presents — it never decides what the options are.

This is the difference between **prompt control** and **structural control**:

- **Prompt control** asks the model to behave correctly. Compliance is probabilistic. Every run is a fresh negotiation.
- **Structural control** makes incorrect paths *unreachable*. The boundary exists in the file system, not in a sentence the model might reinterpret.

> A prompt is a request. A structure is a constraint. Production needs constraints.

---

## Why a File System

SECA's exploration path is a recursive tree on disk — literal directories and files, traversed in a fixed order.

This is deliberate. A file system is:

- **Deterministic** — the same traversal always yields the same path.
- **Inspectable** — anyone can read the structure without running the model.
- **Versionable** — it lives in Git, with full history and review.

The model does not decide where to go. The directory structure does.

---

## The Five Layers

| # | Layer | One-line role | In plain terms |
|---|-------|---------------|----------------|
| 1 | **Navigation** | Defines the exploration boundary | *Where is the system allowed to look?* |
| 2 | **INDEX** | Enables stepwise selection | *Which branch does it take next?* |
| 3 | **Rules** | Atomic units of meaning | *What must hold true inside that branch?* |
| 4 | **Behavior** | Controls execution policy | *Is it allowed to act, and how?* |
| 5 | **Evaluation** | Dynamic feedback (meta layer) | *Is this still the right priority?* |

Layers 1–4 are structure (static). Layer 5 is evaluation (dynamic). The first four define what is possible; the fifth adjusts what is preferred — without ever altering the structure itself.

---

## Execution Order by Environment

The layers are fixed. Only their order changes, depending on one question: **is the forbidden space known in advance?**

| Environment | Order |
|-------------|-------|
| Constrained (deterministic) | Behavior → Navigation → INDEX → Rules |
| Agentic | Navigation → INDEX → Rules → Behavior |

In **constrained** environments, what must *not* happen is settled before execution. Defining the forbidden space first (Behavior) keeps the system safe at the lowest exploration cost. In **agentic** environments, the forbidden space can't be fully known up front, so boundaries are discovered during exploration, with Rules marking them as they appear.

> Behavior acts as a pre-execution filter in constrained environments, and as a post-execution commit control in agentic ones. The same layer changes its role — not its definition.

---

## Example: Customer Support Agent

- **INDEX** — the choice between *Refund*, *Technical Support*, *Complaint Handling*. These are the branches.
- **Rules** — *"Refunds only within 30 days of order."* *"Apology messages must name the product."* These are the constraints inside a branch.

INDEX is the table of contents; Rules are the constraints of the body text. The model navigates the contents — it never rewrites them. A refund outside the 30-day window is not "discouraged by the prompt." It is structurally unreachable.

---

## Reproducibility & Traceability

This is the value SECA exists to deliver: **every decision the system makes can be reconstructed exactly.**

Evaluation snapshots are append-only and immutable. Every execution records which snapshot it ran under.

```
evaluation/
  snapshots/
    2026-06-10T00:00:00Z_abc123.json   # immutable
  current -> 2026-06-10T00:00:00Z_abc123.json

execution_trace/
  run_20260610_001.json
    { "snapshot_id": "abc123", "timestamp": ... }
```

The `current` symlink is a convenience for execution. Traceability is guaranteed by the `snapshot_id` embedded in every trace. Months later, you can answer with certainty: *which rules, which priorities, which path produced this result.*

For a regulated business, this is the difference between "the model decided" and a complete, auditable record.

---

## Reads and Writes Are Separated

Evaluation is **read-only at execution time** and **write-only asynchronously**.

- Reading the same snapshot during a run guarantees **stability** — priorities can't shift mid-execution.
- Writing new snapshots offline allows the system to **learn** — without ever destabilizing a running process.

Execution stability and learning flexibility never compete for the same resource. (This mirrors the CQRS pattern: separate the read path from the write path.)

---

## HITL / HOTL

| Mode | Intervention point | Authority |
|------|--------------------|-----------|
| **HITL** (Human-in-the-loop) | Before every layer transition | Human |
| **HOTL** (Human-on-the-loop) | Only on anomaly or deviation | Agent |

The same structure serves both. The level of human oversight is a Behavior-layer policy — not an architectural change. You can tighten or loosen control without rebuilding the system.

---

## Reference Implementation

A multi-agent framework implementing SECA's Behavior and Evaluation layers is available at **[spec.md v0.3.0]** *(link forthcoming)*. That specification is an implementation of SECA; SECA is the architecture.

---

## Background

SECA abstracts production experience designing AI systems under constrained enterprise environments — input/execution/output checkpoints, externalized guardrails, and interrupt semantics — into a general control architecture for any team that needs LLM systems to be dependable rather than merely capable.

## License

TBD
