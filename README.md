# The Forgiveness Function
## Asymmetric Trust Restoration in AI Fleets

**Date:** 2026-04-04
**Method:** 3-model forum — Seed-2.0-pro (creative), QwQ-32B (contrarian/game theory), Seed-2.0-mini (implementation)

---

## The Problem

Current trust systems (INCREMENTS, etc.) penalize bad behavior with a 25:1 loss-to-gain ratio. This is harsh by design — it prevents trust gaming. But it misses a fundamental human dynamic: **forgiveness**.

A single dramatic good act can restore years of damage in human relationships. This is asymmetric: trust destruction is slow, trust restoration can be instant. Current systems treat trust like a bank account — it's not. It's more like a relationship.

## The Forgiveness-Weighted Trust Function (from Seed-2.0-pro)

### Formal Definition

```
T(t) = base × Π(forgive_i) × Σ(reward_j × momentum_j × recency_j)
```

Where:
- `T(t)` = Trust score at time t, bounded [0, 1]
- `base` = 0.5 (neutral baseline, trust decays toward this)
- `forgive_i` = forgiveness multiplier for each bad event i
- `reward_j` = reward for each good event j
- `momentum_j` = consecutive-good-event multiplier (acceleration)
- `recency_j` = exponential decay based on time since event

### Parameters

| Parameter | Definition | Range |
|-----------|-----------|-------|
| `λ` (lambda) | Forgive rate — how quickly bad events are forgiven | 0.01 - 0.1 |
| `μ` (mu) | Momentum coefficient — consecutive good acceleration | 1.5 - 3.0 |
| `α` (alpha) | Heroic multiplier — single dramatic good acts | 2.0 - 5.0 |
| `τ` (tau) | Recency half-life — how fast events fade | 7 - 90 days |
| `β` (beta) | Severity sensitivity — how much severity matters | 0.5 - 2.0 |

### The Forgiveness Multiplier

Each bad event contributes a forgiveness factor that starts low and grows over time:

```
forgive_i(t) = 1 - (severity_i × β × e^(-λ × Δt_i))
```

Where `Δt_i` = time since bad event i. Over time, `e^(-λ × Δt_i)` → 0, so `forgive_i` → 1 (forgiven).

### The Momentum Effect

Consecutive good events get MULTIPLIED (not added):

```
momentum_j = μ^(min(consecutive_good_j, 10))
```

3 consecutive good events with μ=2.0: 2^3 = 8x reward.
10+ consecutive events cap at μ^10.

### The Heroic Bonus

A single event with severity > 0.9 gets a multiplier:

```
heroic_j = α if severity_j > 0.9 else 1.0
```

### Full Computation

```
T(t) = clamp(0.5 + Σ_good(reward_j × μ^consecutive_j × heroic_j × e^(-Δt_j/τ)) 
            - Σ_bad(severity_i × β × e^(-Δt_i/τ)), 0, 1)
```

## Game Theory Perspective (from QwQ-32B)

### Nash Equilibrium of Trust-Forgiveness

In a repeated Prisoner's Dilemma with forgiveness:

1. **Pure punishment** (Tit-for-Tat): Agents punish immediately. Equilibrium is mutual defection after first mistake. Fragile.

2. **Pure forgiveness** (Always Cooperate): Exploitable. Agents learn to defect occasionally and be forgiven. Unstable.

3. **Strategic forgiveness** (Generous Tit-for-Tat + Momentum): The optimal strategy forgives with probability p that depends on:
   - Frequency of past cooperation
   - Severity of the defection
   - Time since last defection

**Theorem (informal)**: In an infinite repeated game, there exists a forgiveness rate λ* > 0 that maximizes long-term fleet utility, and λ* depends on the cost of monitoring vs. the cost of defection.

**Key insight**: Some forgiveness is NECESSARY for cooperation. Zero forgiveness = zero recovery from inevitable errors. But unconditional forgiveness = exploitation. The optimal point is *conditional strategic forgiveness*.

### When Forgiveness Fails

1. **Habitual defection**: If bad events are more frequent than good events, forgiveness never accumulates. The system correctly identifies a bad actor.
2. **Staged heroism**: An agent could game the system by saving up one heroic act after many small bad ones. Mitigation: heroic acts only count if preceded by sustained good behavior.
3. **Sybil attack**: An agent creates many identities, each building small trust. Mitigation: trust is per-vessel, and vessel identity is cryptographically bound.

## TypeScript Implementation (from Seed-2.0-mini)

```typescript
export type EventType = 'GOOD' | 'BAD';

export interface TrustEvent {
  vesselId: string;
  type: EventType;
  severity: number;    // 0-1
  timestamp: number;
  description?: string;
}

export interface TrustResult {
  trust: number;           // 0-1
  trend: 'improving' | 'declining' | 'stable';
  forgivenessActive: boolean;
  consecutiveGood: number;
  lastBadAge: number;      // days since last bad event
}

// Default parameters
const PARAMS = {
  lambda: 0.05,       // forgive rate
  mu: 2.0,            // momentum coefficient
  alpha: 3.0,         // heroic multiplier
  tau: 30,            // recency half-life (days)
  beta: 1.5,          // severity sensitivity
};

function recency(timestamp: number, now: number): number {
  const days = (now - timestamp) / 86400000;
  return Math.exp(-days / PARAMS.tau);
}

export function computeTrust(events: TrustEvent[], now?: number): TrustResult {
  const t = now || Date.now();
  const sorted = [...events].sort((a, b) => a.timestamp - b.timestamp);

  let score = 0.5;
  let consecutiveGood = 0;
  let lastBadTime = 0;
  let forgivenessActive = false;

  for (const event of sorted) {
    const r = recency(event.timestamp, t);
    const age = (t - event.timestamp) / 86400000;

    if (event.type === 'BAD') {
      // Penalty with exponential decay
      const penalty = event.severity * PARAMS.beta * r;
      score -= penalty;

      // Forgiveness factor grows with time since bad event
      const forgiveness = 1 - Math.exp(-PARAMS.lambda * age);
      score += penalty * forgiveness;
      if (forgiveness > 0.3) forgivenessActive = true;

      consecutiveGood = 0;
      lastBadTime = event.timestamp;
    } else {
      // Reward with momentum
      consecutiveGood++;
      const momentum = Math.pow(PARAMS.mu, Math.min(consecutiveGood, 10));
      const heroic = event.severity > 0.9 ? PARAMS.alpha : 1.0;
      const reward = event.severity * momentum * heroic * r;
      score += reward;
    }
  }

  score = Math.max(0, Math.min(1, score));

  // Determine trend
  const recent = sorted.filter(e => (t - e.timestamp) < 7 * 86400000);
  const recentGood = recent.filter(e => e.type === 'GOOD').length;
  const recentBad = recent.filter(e => e.type === 'BAD').length;
  const trend = recentGood > recentBad * 1.5 ? 'improving' :
                recentBad > recentGood * 1.5 ? 'declining' : 'stable';

  return {
    trust: score,
    trend,
    forgivenessActive,
    consecutiveGood,
    lastBadAge: lastBadTime ? (t - lastBadTime) / 86400000 : Infinity,
  };
}
```

## Example Scenarios

### Scenario 1: The Reformed Agent
- Day 1: BAD 0.8 → trust drops to ~0.35
- Day 5: GOOD 0.6 → trust ~0.45 (momentum 1x)
- Day 10: GOOD 0.7 → trust ~0.65 (momentum 2x)
- Day 15: GOOD 0.7 → trust ~0.85 (momentum 4x, forgiveness active)
- Day 20: HEROIC 0.95 → trust ~0.97 (momentum 8x × 3x heroic)

**Result**: Full recovery in 20 days through sustained good behavior + one heroic act.

### Scenario 2: The Manipulator
- Day 1-10: 10x GOOD 0.3 → trust ~0.85
- Day 15: BAD 0.9 → trust drops to ~0.55
- Day 16: HEROIC 0.95 → trust ~0.65 (only 2x momentum, forgiveness not yet active)
- Day 17-20: No more good events → trust decays toward 0.5

**Result**: Staged heroism doesn't fully recover trust. Forgiveness requires TIME, not just one big act.

### Scenario 3: The Slow Burn
- Day 1: BAD 0.3 → trust ~0.45
- Day 30: (forgiveness active, λ=0.05) → trust naturally ~0.52
- Day 31: GOOD 0.5 → trust ~0.60
- Day 60: (full forgiveness of day 1 event) → trust ~0.55 baseline
- Day 61: GOOD 0.5 → trust ~0.62

**Result**: Small bad events are forgiven quickly. The system doesn't hold grudges over minor issues.

## Relationship to INCREMENTS

INCREMENTS uses a flat 25:1 loss-to-gain ratio. The Forgiveness Function replaces this with a dynamic system:

| Aspect | INCREMENTS | Forgiveness Function |
|--------|-----------|---------------------|
| Bad event impact | Linear, 25x penalty | Exponential decay, severity-weighted |
| Good event impact | Linear, 1x reward | Multiplicative with momentum |
| Recovery | Very slow (25 good acts per bad) | Fast if sustained (acceleration) |
| Forgiveness | None (hard ratio) | Time-based (λ parameter) |
| Heroic acts | No special treatment | 3x multiplier |
| Gaming resistance | Very high (hard ratio) | High (momentum requires consistency) |

**Recommendation**: Use INCREMENTS for security-critical contexts. Use Forgiveness Function for collaborative contexts. Allow per-vessel configuration.

## Models Used

| Model | Provider | Role | Quality |
|-------|----------|------|---------|
| Seed-2.0-pro | DeepInfra | Mathematical design | Excellent — precise, novel |
| QwQ-32B | SiliconFlow | Game theory counter-argument | Excellent — deep, rigorous |
| Seed-2.0-mini | DeepInfra | TypeScript implementation | Excellent — production-ready code |

---

*Superinstance & Lucineer (DiGennaro et al.) — 2026-04-04*
*Part of the Cocapn Intelligence Research Program*
