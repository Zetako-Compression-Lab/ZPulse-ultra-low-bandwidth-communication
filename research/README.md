# ZPulse Research Archive — October 2026

This directory presents the **complete public-facing record of the October 12–13 research campaigns**, without releasing proprietary implementation code.

| Study | Date | Focus |
|---|---|---|
| [Phase 0 — Baseline and audit](phase-0-codec-audit.md) | 12 Oct | Original codec characterization, stability, sanitizer and build tests |
| [Phase 0.5 — Protocol experiments](phase-0-5-protocol-research.md) | 12 Oct | Drift root cause, experimental correction, semantic packing, AEAD and fault simulation |
| [Phase 1 — Secure host proof of concept](phase-1-secure-messaging.md) | 13 Oct | Actual UDP processes, authenticated delivery, restart recovery, fault injection |
| [Competitive carrier benchmark](competitive-carriers.md) | 13 Oct | ZPulse direct vs MQTT-SN, CoAP and BPv7 experimental wrappers |
| [ACK loss and confirmation-cost research](ack-loss-research.md) | 13 Oct | 73 runs, ACK loss, retransmission amplification and delivery-cost accounting |
| [ACK optimization proposals](ack-optimization-proposals.md) | 13 Oct | Six research hypotheses with conditional savings and safety trade-offs |
| [ACK coalescing experiments](ack-coalescing-experiments.md) | 13 Oct | 114-run campaign, full and sparse batches, loss regressions and two-process tests |
| [Combined overview](2026-10-zpulse-results.md) | 13 Oct | Executive synthesis and research limitations |

**Evidence status:** All numeric outcomes are transcribed from internal experimental reports. This public archive does not include the private source, runnable harness, raw cryptographic fixtures or complete machine logs. Consequently, readers can audit the methodology and numerical claims, but cannot independently reproduce the proprietary implementation from this repository alone. No claims of field certification, independent security audit or market superiority are made.
