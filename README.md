# SECA — Structured Exploration and Control Architecture

> **Structure decides the path. Not the model.**

SECA is a model-agnostic control architecture for LLM systems in business environments. It guarantees consistency and reproducibility by separating what is structural (and therefore fixed) from what is evaluative (and therefore adaptive).

## Why

LLMs are non-deterministic by nature. In consumer applications, this is a feature. In business operations — refunds, compliance, customer communication — it is a liability. Most agent frameworks try to constrain model behavior through prompting alone, which means the exploration path itself is decided by the model on every run.

SECA inverts this. The exploration path is decided by structure — a recursive, file-system-like tree that exists outside the model. The model selects within boundaries; it never defines them.

## Core Principles

**Order is static.** The exploration path is fixed by structure, traversed recursively. The LLM does not decide where to go; the structure does.

**Priority is dynamic.** What matters most depends on the user's purpose and responsibility. This is managed by the Evaluation layer — from outside the structure, never by mutating it.

**Reads and writes are separated.** Evaluation is read-only at execution time (stability) and write-only asynchronously (adaptability). Execution stability and learning flexibility never compete for the same resource. The design is close in spirit to CQRS.

Static structure × dynamic evaluation: the structure never changes, which guarantees reproducibility; the evaluation axis acts on it from outside, which produces adaptability.

## The Five Layers

| # | Layer | Role |
|---|-------|------|
| 1 | **Navigation** | Defines the exploration boundary |
| 2 | **INDEX** | Enables stepwise selection |
| 3 | **Rules** | Atomic units of meaning |
| 4 | **Behavior** | Controls execution policy |
| 5 | **Evaluation** | Dynamic evaluation and feedback (meta layer) |

## Execution Order by Environment

| Environment | Order |
|-------------|-------|
| Constrained (deterministic) | Behavior → Navigation → INDEX → Rules |
| Agentic | Navigation → INDEX → Rules → Behavior |

The order inverts because of one question: **is the forbidden space known in advance?**

In constrained environments, what must *not* happen is settled before execution. Defining the forbidden space first (Behavior) minimizes the trade-off between exploration cost and safety. In agentic environments, the forbidden space cannot be fully determined up front — boundaries are discovered through exploration, with Rules marking them as they appear.

> Behavior acts as a pre-execution filter in constrained environments, and as a post-execution commit control in agentic ones. The same layer changes its role — not its definition.

This is why all five layers stay fixed while only the order inverts.

## Example: Customer Support Agent

- **INDEX** — the choice between *Refund*, *Technical Support*, and *Complaint Handling*. These are branches.
- **Rules** — *"Refunds only within 30 days of order"*, *"Apology messages must name the product"*. These are constraints inside a branch.

INDEX is the table of contents; Rules are the constraints of the body text. The model navigates the contents — it never rewrites them.

## Reproducibility

Evaluation snapshots are append-only and immutable. Every execution records which snapshot it read.

```
evaluation/
  snapshots/
    2026-06-10T00:00:00Z_abc123.json   # immutable
  current -> 2026-06-10T00:00:00Z_abc123.json

execution_trace/
  run_20260610_001.json
    { "snapshot_id": "abc123", "timestamp": ... }
```

The symlink is a convenience. Traceability is guaranteed by the `snapshot_id` embedded in every execution trace — any past run can be reconstructed exactly, including the evaluation axes it ran under.

## HITL / HOTL

| Mode | Intervention point | Authority |
|------|--------------------|-----------|
| HITL | Before every layer transition | Human |
| HOTL | Only on anomaly or deviation | Agent |

The same structure serves both. The mode is a Behavior-layer policy, not an architectural change.

## Reference Implementation

A multi-agent framework implementing SECA's Behavior and Evaluation layers is available at **[spec.md v0.3.0]** *(link forthcoming)*. That specification is an implementation of SECA; SECA is the architecture.

## Background

SECA abstracts production experience designing AI systems under constrained enterprise environments — input/execution/output checkpoints, externalized guardrails, and interrupt semantics — into a general control architecture.

## License

TBD
