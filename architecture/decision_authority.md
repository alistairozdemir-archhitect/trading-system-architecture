# Decision Authority (Who can override what)

This document defines who/what has authority to make decisions in a trading system,
and how overrides are allowed without creating chaos.

A system that trades without clear decision authority is not automated, it is accidental.

---

## The problem this solves

In live trading, decisions come from multiple places:

- strategy logic (entries/exits)
- risk engine (position sizing, exposure limits, drawdown caps)
- execution layer (retries, partial fills, routing constraints)
- monitoring layer (kill-switch, degraded mode)
- human operator (manual stop/resume, force-flat)

If these are not ordered, the system will conflict with itself.
Conflicts always show up at the worst possible time.

---

## Authority hierarchy (hard rule)

Highest authority always wins. No exceptions.

1) **Emergency / Kill-switch (Monitoring Layer)**
2) **Risk Engine / Capital Control**
3) **State Machine / Operating Mode**
4) **Execution Constraints**
5) **Strategy Logic**
6) **Optimizations / Secondary rules**

Interpretation:
- Strategy can never override risk.
- Risk can never override kill-switch.
- Execution can never "keep trying" if the system is paused/halted.
- A human can override strategy, but not ignore safety policies unless explicitly allowed.

---

## Allowed decision types

### Strategy decisions (lowest authority)
- entry/exit signals
- pyramiding or re-entry logic
- regime-based strategy selection

These must always be conditional on higher-level permission.

### Execution decisions (constraint layer)
- order type selection (limit/market)
- retry and backoff
- partial fill handling
- pacing/rate limiting
- route selection / venue constraints

Execution decisions are allowed only if they do not increase risk beyond policy.

### Risk decisions (high authority)
- position sizing
- exposure caps (per instrument, per sector, per correlation cluster)
- stop logic constraints (max loss per trade/day/week)
- drawdown governance (reduce risk, pause, halt)
- portfolio concentration rules

Risk decisions can block trades, reduce exposure, or force flattening.

### Monitoring decisions (highest authority)
- detect failure conditions
- transition system state (DEGRADED, PAUSED, HALTED, INVALID)
- trigger kill-switch actions
- notify operator and require acknowledgement

Monitoring can force a stop even if strategy is “right”.

---

## Human authority (how manual control works)

Humans should not be an ambiguous “shadow layer”.
If manual control exists, define it as explicit commands.

### Recommended operator commands
- `PAUSE_NEW_RISK`  (no new entries, manage positions allowed)
- `HALT_SYSTEM`     (cancel orders, optionally flatten)
- `RESUME`          (only after gates pass)
- `FORCE_FLATTEN`   (close positions immediately by policy)
- `ACKNOWLEDGE_EVENT` (required after severe failures)

### Human override policy
- Human can override strategy decisions at any time.
- Human can request overrides to risk/monitoring only if policy allows it.
- If policy does not allow it, the system should refuse and log the refusal.

A serious client will respect a system that refuses unsafe commands.

---

## Override safety (how to prevent abuse)

Overrides must be:

- **explicit** (a named command, not a “chatty” instruction)
- **logged** (who, when, why, what changed)
- **scoped** (duration, affected instruments, affected actions)
- **reversible** (auto-expire or explicit rollback)
- **auditable** (history must survive restarts)

### Example: safe override
"Pause new risk for 2 hours across all instruments, keep exit management enabled."

### Example: unsafe override
"Ignore risk limits today."

The system should reject unsafe overrides.

---

## Decision contracts (recommended)

To avoid conflicts, define a simple contract:

- Strategy produces *intent* (desired actions)
- Risk produces *permission* (allowed actions + max size)
- Execution produces *feasibility* (how actions can be executed safely)
- Monitoring produces *enforcement* (stop/limit/pause/halt)

If permission = 0, intent is ignored. This must be normal and expected.

---

## Why this matters

Without decision authority, you get:
- silent risk escalation
- “retry loops” that become exposure
- humans overriding systems under stress
- systems that trade while unhealthy

A senior system is one where:
- authority is explicit
- overrides are controlled
- safety always has the final word
