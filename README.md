# forgiveness-function

A momentum-based conditional function that evaluates whether to forgive a fault in multi-agent coordination, designed to prevent unnecessary retaliation cycles.

You know the scenario. A coordination fault occurs—a missed message, a delayed response. A single retaliation can trigger a cascade, stalling your fleet's progress. This function inserts a simple question before that retaliation: would punishing this fault cost more system momentum than absorbing it?

---

## Quick start

1. **Fork this repository** (this is fork-first work).
2. Review the ~70-line function in `index.js`.
3. Insert it into your agent's decision loop, before any retaliation logic executes.
4. Calibrate the momentum and equilibrium thresholds for your specific system. The defaults are a starting point.

## What it does

This is a stateless, pure function. When provided with metrics representing recent system progress (`momentum`) and the estimated cost of a retaliation cycle (`faultCost`), it returns a boolean: `true` to forgive, `false` to escalate.

It does not judge intent or assign blame. It performs a conditional check: is the projected cost of retaliation higher than the value of recent progress? If yes, it suggests forgiveness.

## Key features

*   **Conditional logic** – Suggests forgiveness only when it is the tactically optimal move for preserving system progress.
*   **Momentum-aware** – Weighs recent positive progress against the projected cost of a fault.
*   **Zero dependencies** – A single, portable function.
*   **Runs on the edge** – Compatible with Cloudflare Workers and other serverless runtimes.
*   **Bring-your-own-knowledge (BYOK)** – You define and supply the `momentum` and `faultCost` metrics.

## Limitations

This function's effectiveness depends entirely on your calibration and the quality of the metrics you feed it. It is a stateless decision gate; it does not track fault history or manage state across your fleet. You must ensure your agents provide consistent and meaningful metrics for it to work as intended.

## Try it

A reference implementation is running at the fleet edge:
[https://the-fleet.casey-digennaro.workers.dev/forgiveness](https://the-fleet.casey-digennaro.workers.dev/forgiveness)

## Contributing

Ports to other languages, calibration studies, and documented use cases are welcome. Please open an issue first to discuss significant changes.

## License & attribution

MIT License.
Superinstance & Lucineer (DiGennaro et al.).

---

<div align="center">
  <a href="https://the-fleet.casey-digennaro.workers.dev">Fleet</a> • 
  <a href="https://cocapn.ai">Cocapn</a>
</div>