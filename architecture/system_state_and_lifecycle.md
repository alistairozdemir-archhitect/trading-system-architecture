# System State & Lifecycle

This document defines how I model a trading system as a state machine.
A system that can place orders is not necessarily a system that is safe to operate.

The goal: make "what state are we in?" explicit, observable, and enforceable.

---

## Core principle

A trading system should be able to answer these questions at any moment:

- What is the current operational state?
- What conditions triggered this state?
- What actions are allowed in this state?
- What is the path back to normal operation (if any)?
- Who/what has the authority to transition states?

If the system cannot answer these, it is not production-ready.

---

## State model (high level)

I use a simple but enforceable operational state model:

### 1) INIT
System is starting up. Dependencies may not be available yet.
- Allowed: load config, validate environment, warm up connections
- Not allowed: place orders

Exit condition: all required dependencies validated.

### 2) OBSERVE (Read-only)
System is ingesting data and computing signals, but cannot trade yet.
Used after startup and after any recovery.
- Allowed: subscribe data, compute indicators, generate signals
- Not allowed: place orders

Exit condition: stable data + stable clocks + stable connectivity for N cycles.

### 3) ARMED (Trade-ready, not trading)
System has passed safety gates. Trades are permitted, but no active position is required.
- Allowed: place orders if entry rules pass
- Allowed: manage existing positions (if any)

Exit condition: an order is sent or a position exists.

### 4) ACTIVE (Trading)
System has at least one active position or is actively managing orders.
- Allowed: all position/order management actions
- Must enforce: risk envelope, kill-switch rules, latency limits

Exit condition: no positions + no working orders (or transition to PAUSED/DEGRADED).

### 5) DEGRADED (Trading with restrictions)
System can still function, but one or more quality constraints are violated.
Examples: widened spreads, partial data loss, high latency, broker warnings.
- Allowed: manage existing positions
- Allowed: reduce exposure
- Not allowed: open new risk unless explicitly permitted by policy

Exit condition: stability restored for N cycles OR forced transition to PAUSED.

### 6) PAUSED (No new risk)
System is intentionally stopped from taking new trades.
Usually triggered by a policy decision, not a crash.
- Allowed: observe, compute, log
- Allowed: manage/close existing positions (optional policy)
- Not allowed: new entries

Exit condition: explicit operator resume + gates pass.

### 7) HALTED (Emergency stop)
System is in a failure mode where continuing could create uncontrolled loss.
- Allowed: cancel orders, flatten positions (policy-dependent), alert
- Not allowed: any new orders besides emergency actions

Exit condition: manual intervention + root-cause acknowledgment.

### 8) INVALID (Unsafe / inconsistent)
System state is inconsistent with reality.
Examples: position mismatch, order state desync, clock drift, corrupted input.
This is not a "wait and hope" state.
- Allowed: stop trading, reconcile, resync
- Not allowed: discretionary trading

Exit condition: reconciliation complete and verified.

---

## Transition rules (what triggers state changes)

Transitions should never be "implicit". Every transition must have:

- a trigger (event / condition)
- a reason (logged)
- a validator (gate)
- a controller (authority)

### Common triggers
- Data feed instability (missing bars/ticks, out-of-order timestamps)
- Time sync issues (clock drift, timezone mismatch, stale candles)
- Broker connectivity issues (disconnects, rejected orders, pacing violations)
- Execution anomalies (partial fills, unexpected slippage, stale quotes)
- Risk violations (exposure cap, drawdown limits, correlation spike)
- Internal invariants broken (state mismatch, NaNs, config mismatch)

---

## Operational gates (minimum checks)

Before transitioning into ARMED/ACTIVE, the system should verify:

### Data & time integrity
- timestamps monotonic within stream
- expected update cadence is met
- no material gaps for the configured timeframe
- time alignment matches instrument trading session assumptions

### Broker & execution readiness
- connection stable
- account permissions and market data subscriptions valid
- order placement test passed (paper/live per policy)
- reject/error rate below threshold

### Risk engine readiness
- exposure caps loaded
- position sizing inputs valid
- kill-switch policies loaded
- current portfolio state reconciled (positions/orders match broker)

---

## Reconciliation & recovery workflow (recommended)

When entering INVALID / HALTED:

1) Freeze new risk (block entry orders)
2) Cancel working orders (or mark for reconciliation)
3) Pull broker truth (positions, orders, executions)
4) Compare system state vs broker state
5) Resolve differences (sync or flatten per policy)
6) Require a human acknowledgement if the event was severe
7) Move to OBSERVE for N cycles
8) Re-arm only after gates pass

This prevents "silent drift" where the system continues trading with wrong assumptions.

---

## Why this matters

Most trading losses are not caused by strategy logic.
They are caused by:

- state desynchronization
- false readiness assumptions
- recovery performed without verification
- systems that cannot deliberately stop

A senior system is defined by its ability to degrade safely, stop decisively, and recover cleanly.
