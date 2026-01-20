# Monitoring and Kill Switch Logic

This document defines how the system is observed, when it is stopped, and who has decision authority.

A system that cannot stop itself safely is not production-ready.

---

## Monitoring Is Not Performance Tracking

Monitoring exists to detect failure, not to admire returns.

Primary objectives:
- Detect abnormal behavior early
- Identify state corruption
- Prevent small issues from compounding

---

## What Is Monitored

Key monitoring domains:

- Market data integrity (staleness, gaps, anomalies)
- Execution quality (slippage, fill ratios, latency)
- Position consistency (strategy vs broker state)
- Risk exposure (aggregate, per-instrument, correlation)
- System health (process stability, logging continuity)

If something cannot be observed, it cannot be controlled.

---

## Kill Switch Philosophy

The kill switch exists to protect capital, not ego.

It is triggered when:
- Assumptions are violated
- State integrity is compromised
- Risk becomes unquantifiable

Stopping the system is a success condition, not a failure.

---

## Automated vs Human Authority

The system distinguishes between:

- **Automatic stops**
  - Data feed failure
  - Position desynchronization
  - Risk limit breaches

- **Human-in-the-loop decisions**
  - Execution degradation
  - Structural regime uncertainty
  - Prolonged abnormal behavior

Automation handles the obvious.
Humans handle the ambiguous.

---

## Cooldown and Restart Logic

After a kill event:
- No immediate restart
- System state must be reviewed
- Root cause must be understood

Restarting without understanding is equivalent to doubling risk.

---

## Design Principle

If you do not know when your system should stop,
it will decide for you, at the worst possible moment.
