# Phase 1 — Secure Messaging Host Proof of Concept

**Date:** 13 October 2026. **Evidence:** internal Phase 1 technical report. **Status:** trusted-host engineering PoC, not field or commercial certification.

## Implemented and exercised

A sender and receiver CLI exchanged authenticated application events over IPv4 loopback UDP. The unchanged original C codec was used for the compact application body. The engineering profile used a **51-byte authenticated event** and a **51-byte authenticated destination ACK** with full authentication tags.

The experiment included durable queues, verified destination acceptance, suppression of duplicate accepted events, replay handling, persistent sequencing and restart recovery. It used host Python, SQLite and established cryptographic libraries. It was **not** a tiny-MCU implementation.

## End-to-end results

| Run | Accepted / verified ACKed | Duplicate accepted | Event attempts | Wall elapsed | CPU |
|---|---:|---:|---:|---:|---:|
| Clean live-UTC process test | 10 / 10 | 0 | 10 | 0.519 s | 0.206 s |
| Engine/SQLite simulation | 1,000 / 1,000 | 0 | 2,645 | 73.359 s | 21.359 s |
| Actual local UDP/process fault test | **1,000 / 1,000** | **0** | 3,221 | 63.627 s | 22.373 s |
| Repeated actual UDP/process fault test | **1,000 / 1,000** | **0** | 3,156 | 65.305 s | 22.757 s |

Fault policy included 30% independent loss, 10% duplication, 20% additional delay/reordering and 2% invalid-tag variants. Sender and receiver were killed and restarted in actual process tests. The common **120-second outage was simulated-time**, corresponding to approximately 12 wall seconds; it is not a measured physical link outage.

In the first actual fault run, **35 corrupted events** and **24 invalid-tag ACKs** were rejected. No duplicate event was accepted.

## Latency and storage

| Metric | First actual fault run | Repeat |
|---|---:|---:|
| Submit-to-ACK median / p95 (virtual seconds) | 9.327 / 52.574 | 9.286 / 53.570 |
| Enqueue-to-ACK median / p95 (virtual seconds) | 368.081 / 595.170 | 370.230 / 596.805 |
| Combined endpoint storage | 1,857,707 B | 1,862,738 B |
| Peak child RSS bound | 55,230,464 B | 53,886,976 B |

Host storage includes authenticated journaling and SQLite overhead, not an embedded flash budget. Median event seal/open times in the first run were **2.203 / 2.710 μs CPU** including validation/serialization. A clean event/ACK exchange has a minimum of **158 B over IPv4/UDP** before link overhead.

## Test quality and caveats

**24 regression tests passed**, including original codec goldens, 1,000 seeded codec events, protocol fixtures, independent AEAD checks, mutation/invalid-packet tests, duplicate handling, persistence, capacity, clock and crash-boundary scenarios. Eight abrupt-exit subcases exercised sender and receiver commit boundaries.

Unsupported cases include complete simultaneous key/database/journal rollback or device cloning, operational key rotation/revocation, hardware-backed keys, public ingress, physical brownouts, constrained-device deployment, cross-platform peer validation and field network measurements.

The proof of concept **does not guarantee emergency message delivery**, qualify the cryptography for production, or replace certified distress equipment.

## Follow-on

The same secure application was subsequently tested with multiple experimental transport carriers in the [competitive study](competitive-carriers.md).
