# forgiveness-function ✦

Prevents retaliation cascades in multi-agent systems by calculating the tactical efficiency of forgiveness.

Before your agent escalates over a fault, this function checks if the cost of punishment would waste more system momentum than absorbing the error. It is a single, stateless check you insert into your agent's decision loop.

---

## Why It Exists
Multi-agent systems often fail from accidental feedback loops: one penalty triggers another, collapsing cooperation. Manually patching each edge case is fragile. This function provides a general, arithmetic check for those moments.

---

## Quick Start
1.  **Fork this repository.** This is fork-first work; you own your own calibration.
2.  Read the ~70-line function in `index.js`.
3.  Insert it directly before any retaliation logic in your agent's decision cycle.
4.  Calibrate the two constants for your fleet. The defaults are tuned for clusters of 5 to 50 agents.

## How It Works
Pass the function two numbers:
- **Recent Momentum**: Measurable work completed by the system in the last N cycles.
- **Retaliation Cost**: The projected full cost of executing one penalty cycle.

It returns a boolean: `true` to forgive, `false` to proceed with escalation. It contains no state, memory, or side effects.

## What Makes This Different
1.  **Not a Pacifist**: It only advises forgiveness when punishment is systemically inefficient. Faults that truly impact momentum will still be escalated.
2.  **Stateless**: It holds no grudges and builds no reputation scores. Each decision is based solely on current system conditions.
3.  **Advisory**: It is a check, not an override. Your agent retains final authority.

## Key Features
*   **Tactical, Not Moral**: Framed purely as a resource efficiency calculation.
*   **Momentum-Aware**: Weighs recent, actual progress against projected disruption.
*   **Zero Dependencies**: One file. Copy it anywhere.
*   **Portable**: Runs on Cloudflare Workers, Node.js, and any serverless environment.
*   **BYOMetrics**: You supply the metrics. The function makes no assumptions about your fleet's architecture.

## Limitations
This function will not correct poorly designed incentives or prevent deliberate attacks. **If the projected cost of retaliation is underestimated by more than 30%, its forgiveness advice can become counterproductive, potentially increasing systemic instability.** It is a circuit breaker for unintended cascades, not a comprehensive conflict-resolution system.

## Live Reference
A live edge implementation is available for testing:  
[https://the-fleet.casey-digennaro.workers.dev/forgiveness](https://the-fleet.casey-digennaro.workers.dev/forgiveness)

## Contributing
Ports to other languages, data from production calibrations, and documented case studies of cascade failures are welcome. Please open an issue before submitting major changes.

## License
MIT License. Open source for any use.

Attribution: Superinstance and Lucineer (DiGennaro et al.)

<div style="text-align:center;padding:16px;color:#64748b;font-size:.8rem"><a href="https://the-fleet.casey-digennaro.workers.dev" style="color:#64748b">The Fleet</a> &middot; <a href="https://cocapn.ai" style="color:#64748b">Cocapn</a></div>