# Competitive Carrier Benchmark — Direct, CoAP, MQTT-SN, BPv7

**Date:** 13 October 2026. **Evidence:** internal competitive benchmark report. **Status:** controlled experimental comparison, not native-stack superiority claim.

## Comparison contract

Each profile carried **the same 51-byte authenticated ZPulse application message** and used the **same application-level durable reliability engine**. The study measures incremental carrier overhead and selected timing behavior.

Workload: 1,000 position/preset events, equivalent precision, matching event/ACK semantics. Network: modeled IPv4/UDP, 100 kbit/s per direction, 50 ms propagation, 250 ms ticks. Disturbed trace: 30% loss, 10% duplication, 20% delayed scheduling, invalid copies and a common 120-second virtual-time outage. Counts include emitted IP/UDP headers, data, ACK, controls, setup and retransmissions; radio MAC/PHY is excluded.

All four profiles confirmed **1,000/1,000 events**, with zero accepted application duplicates in both healthy and disturbed cases.

## Healthy condition

| Profile | Total emitted bytes | Bytes / confirmed event | Median enqueue→ACK (virtual s) | p95 (virtual s) |
|---|---:|---:|---:|---:|
| Direct ZPulse | **158,000** | **158.00** | 11.75 | 22.25 |
| CoAP wrapper | 174,000 | 174.00 | 11.75 | 22.25 |
| MQTT-SN wrapper | 242,144 | 242.14 | 13.375 | 24.00 |
| BPv7 wrapper | 265,444 | 265.44 | 12.00 | 22.50 |

## Disturbed condition

| Profile | Total emitted bytes | Bytes / confirmed event | Median enqueue→ACK (virtual s) | p95 (virtual s) |
|---|---:|---:|---:|---:|
| Direct ZPulse | **351,708** | **351.71** | 183.25 | 246.75 |
| CoAP wrapper | 388,332 | 388.33 | 183.50 | 247.00 |
| MQTT-SN wrapper | 663,114 | 663.11 | **162.00** | **195.00** |
| BPv7 wrapper | 590,346 | 590.35 | 183.75 | 247.25 |

Relative to these **specific wrappers**, direct carriage used approximately 9.4% fewer bytes than CoAP, 40.4% fewer than BPv7 and 47.0% fewer than MQTT-SN. This must **not** be represented as the same percentage advantage over optimized native end-to-end implementations.

MQTT-SN confirmed faster in the disturbed scenario at greater byte cost. QoS retransmission behavior and cold-start timing both affect that result. Pre-established sessions need a separate campaign.

## Resource observations

| Profile | Host CPU healthy | Host CPU disturbed | Disturbed worker RSS | Disturbed endpoint storage |
|---|---:|---:|---:|---:|
| Direct | 16.098 s | 21.476 s | 94.12 MiB | 2,044,789 B |
| CoAP | 18.223 s | 29.587 s | 95.84 MiB | 2,093,941 B |
| MQTT-SN | 27.862 s | 45.233 s | 94.83 MiB | 1,970,392 B |
| BPv7 | 14.958 s | 19.051 s | 93.94 MiB | 2,034,920 B |

CPU includes common encryption, persistence and orchestration. RSS is whole-worker memory. Different libraries and runtime scheduling make these unsuitable for protocol-intrinsic CPU or MCU rankings.

**Energy:** illustrative models only; no physical power measurement or selected radio.

## Critical interpretation

- These are **experimental carriers around a common ZPulse engine**, not complete independent secure native stacks.
- The experiment used host sockets and simulated network timing; no field radio, satellite or multi-hop DTN deployment was tested.
- Only limited repetition was performed; no statistical cross-stack performance ranking is established.
- MQTT-SN's QoS behavior introduces a latency/traffic trade-off worth studying.
- Security properties of native MQTT-SN, CoAP/DTLS/OSCORE and BPv7/BPSec stacks were **not** independently compared end to end.

## Next research

Native complete-stack baselines with equivalent security/durable delivery, multi-seed repetitions, warmed sessions, real identical radio hardware and measured current/energy are required before commercial comparative claims.
