# forgiveness-function

You build agent fleets that need to coordinate. When one agent faults, a simple rule decides the system's future: retaliate or forgive. Common defaults can trap your fleet in cycles of blame or make it tolerate anything.

This function provides a third option: a calculated decision.

**Strategic forgiveness for autonomous agents** — a momentum-based recovery mechanism for multi-agent coordination.

---

## Try it now

Run the live reference implementation. No account required.
👉 [https://the-fleet.casey-digennaro.workers.dev/forgiveness](https://the-fleet.casey-digennaro.workers.dev/forgiveness)

Adjust transgression severity, current momentum, and equilibrium distance. It returns a boolean: forgive, or escalate.

## What it does

This is a stateless coordination function. It assumes agents are self-interested and calculates if forgiveness is the optimal *tactical* move for a given agent at that moment.

It does not use arbitrary penalty scores. The decision is based on an estimate of system progress lost by retaliating versus absorbing the fault.

**One honest limitation:** The function requires you to define and calibrate "momentum" and "equilibrium" for your specific system. Its effectiveness depends on this tuning.

## Quick Start

1.  **Fork this repository.** This is fork-first work.
2.  **Review the core function** in `index.js` (or your language's port).
3.  **Integrate it** into an agent's decision loop before retaliation logic runs.
4.  **Calibrate** the momentum and equilibrium thresholds for your fleet's dynamics.

## How it works

The function models forgiveness as a conditional boundary, not a behavioral trait. When a fault occurs, the function is called. It recommends escalation only if allowing the transgression would likely break a stable cooperative equilibrium. Otherwise, it recommends forgiveness to preserve system momentum.

## Key Features

*   **Conditional Logic** – Boundary conditions determine when forgiveness leads to better long-run outcomes than retaliation.
*   **Momentum-Aware** – Agents weigh preserving recent progress against the cost of punishment.
*   **Portable Design** – A single, stateless function with zero dependencies. Can run in any runtime (Node, Cloudflare Workers, browsers).
*   **Reference Implementation** – Comes with a working example and a test suite to verify the logic.

## Implement in Your Fleet

This repository provides the formal model and a reference implementation in JavaScript. Port it to your agent's language and integrate it into your coordination protocol. There is no lock-in, API, or proprietary runtime.

## Contributing

The project is fork-first. Fork it, run experiments, adjust the boundaries for your use case, and test it. If you find an improvement, open a pull request or start a discussion.

---

MIT License · Superinstance & Lucineer (DiGennaro et al.)

<div align="center">
  <a href="https://the-fleet.casey-digennaro.workers.dev">The Fleet</a> · <a href="https://cocapn.ai">Cocapn</a>
</div>