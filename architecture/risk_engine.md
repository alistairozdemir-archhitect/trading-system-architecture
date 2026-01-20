# Risk Engine

This document defines how risk is treated as a system-wide control layer, not a per-trade calculation.

Risk is not optimized.
Risk is governed.

---

## Core Principle

The primary objective of the risk engine is survivability.

Profit is a byproduct of staying operational across regimes, not the result of aggressive exposure.

---

## Position Sizing Philosophy

Position size is determined by system state, not signal strength.

Key considerations:
- Volatility regime
- Correlation with existing exposure
- Recent drawdown depth
- Execution uncertainty

No single trade is allowed to materially impact system integrity.

---

## Drawdown Governance

Drawdown is treated as information.

As drawdown increases:
- Risk is reduced
- New exposure is restricted
- Certain strategies may be disabled entirely

There is no attempt to “trade out of drawdown”.

---

## Risk Budgeting

Risk is allocated at the portfolio level.

Strategies do not compete for capital, they operate under explicit risk budgets.

When the system approaches its risk limits:
- Position scaling is reduced
- New entries are filtered
- Capital preservation takes precedence over opportunity capture

---

## Strategy Veto Logic

Not all strategies are allowed to operate in all conditions.

The risk engine can:
- Reduce strategy weight
- Pause execution
- Fully disable a strategy

This decision is driven by regime mismatch and capital stress, not signal failure.

---

## Failure-Aware Design

The risk engine assumes:
- Signals will fail
- Execution will degrade
- Markets will behave unexpectedly

The system is designed to degrade gracefully, not catastrophically.

---

## Design Principle

A system that cannot say “no” to itself
will eventually say “goodbye” to its capital.

