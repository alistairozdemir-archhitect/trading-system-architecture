# Execution Realism

This document describes how I treat execution as a first-class architectural concern, not an implementation detail.

Backtests assume fills.
Markets do not.

---

## Core Assumption

Execution is adversarial.

Latency, slippage, partial fills, and rejections are not edge cases, they are the baseline reality of live trading.

Any system that assumes otherwise is fragile by design.

---

## Latency Awareness

Latency is not just network delay.

It includes:
- Data timestamp vs decision time
- Decision time vs order submission
- Order submission vs exchange acknowledgement
- Acknowledgement vs actual fill

Strategies that rely on precise bar closes or exact trigger prices are structurally incompatible with live execution.

---

## Slippage as a System Property

Slippage is not random noise.

It emerges from:
- Order size relative to liquidity
- Market regime (trend vs chop)
- Volatility expansion
- Execution timing

I treat slippage as a structural tax on aggressiveness, not a parameter to be “tuned away”.

---

## Partial Fills & Order State

Orders are stateful processes, not atomic events.

The system must correctly handle:
- Partial fills
- Delayed fills
- Cancel / replace scenarios
- Mismatch between expected and actual exposure

Position state is derived from execution feedback, never assumed.

---

## Failure-Tolerant Execution

Execution failure modes are expected.

Design considerations include:
- Safe retry logic
- Idempotent order handling
- Clear reconciliation between intended and actual positions
- Explicit handling of exchange or broker anomalies

A system that cannot safely fail should not trade real capital.

---

## Design Principle

Backtests describe intent.
Execution reveals truth.

Architecture must be built around what the market actually delivers, not what the strategy hopes for.

