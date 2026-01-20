# Failure Modes

This document outlines failure scenarios that do not appear in backtests but routinely destroy live trading systems.

Most systems do not fail loudly.
They fail silently.

---

## The Backtest Illusion

Backtests assume:
- Perfect data
- Instant execution
- Stable system state
- No operational degradation

Live systems violate all of these assumptions.

---

## Silent Failures

Silent failures are the most dangerous.

Examples:
- Stale market data feeding valid-looking signals
- Order rejections that do not propagate correctly
- Partial fills misclassified as full exposure
- Position state drifting from broker reality

These failures do not crash the system.
They slowly corrupt it.

---

## Execution Degradation

Execution quality degrades before performance does.

Key risks:
- Latency spikes during volatility
- Slippage exceeding modeled bounds
- Liquidity disappearing exactly when exits are required

Backtests never warn you about this.

---

## State Desynchronization

System state can diverge across:
- Strategy logic
- Risk engine
- Broker positions
- Internal accounting

Once state desync occurs, the system may act on false assumptions.

This is a primary cause of catastrophic loss.

---

## Infrastructure Fatigue

Long-running systems accumulate risk over time.

Examples:
- Memory leaks
- Unhandled edge cases
- Logging failures masking earlier warnings
- Monitoring blind spots

Failure is often a slow drift, not a sudden break.

---

## Defensive Design Philosophy

The system is designed under the assumption that:
- Something is always slightly wrong
- Perfect information does not exist
- Safety must be explicit, not assumed

Failure-aware systems survive longer than optimized ones.

---

## Design Principle

If you cannot enumerate how your system can fail,
you have not designed it.

