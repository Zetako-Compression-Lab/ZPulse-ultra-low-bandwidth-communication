# ZPulse — ACK Loss and Cost per Confirmed Event

**Research date:** 13 October 2026  
**Status:** controlled experimental research; no production protocol change  
**Scope:** authenticated application acknowledgements, retransmission costs, failure accounting and recovery

## Research question

When the receiver has already durably accepted an event but its authenticated acknowledgement is lost, how much unnecessary traffic does the sender generate before confirmation?

The principal metric is **all emitted IP bytes divided by authenticated, durably committed application confirmations**. This includes control packets, retries, unsuccessful traffic and post-confirmation native transport traffic. When no events are confirmed, the ratio is **undefined**, not zero.

## Experimental design

The four profiles—ZPulse direct, MQTT-SN, CoAP and BPv7—used the **same ZPulse application security and persistence engine**, the same 7-byte position/status content and the same 51-byte secured event and ACK. The carrier wrappers were not complete independent native secure stacks.

- 73 experimental runs; 12,336 events submitted and accepted.
- 12,208 authenticated, persisted confirmations; **zero events accepted twice**.
- The remaining 128 unconfirmed events came exclusively from four deliberate 100%-ACK-loss controls of 32 events each.
- Screening: 128 events, three deterministic seeds (210228–210230), five conditions per profile, plus targeted controls.
- Additional validation: 1,000 events per profile with 30% application-ACK loss, one seed.
- Modeled channel: IPv4/UDP, 100 kbit/s each direction, 50 ms propagation, 250 ms simulation tick.
- Mixed fault scenario: 30% loss, 10% duplication, 20% additional delay up to 2 seconds, invalid copies, 120-second **virtual-time** outage and endpoint reopen events.

Socket operations, host encryption and SQLite persistence were exercised; fault scheduling and link timing were simulated. No physical radio, power or battery measurements were made.

## ACK-loss sensitivity — ZPulse direct

Median emitted IP bytes per confirmed event across three seeds:

| ACK loss condition | Bytes / confirmed event |
|---|---:|
| 0% (healthy) | **158.00** |
| 30% | **219.72** |
| 70% | **556.70** |

At 70% ACK loss, retransmission of events already durably accepted accounted for approximately **35.81%** of emitted bytes in the tested profile; ACK traffic accounted for approximately **50%**. These figures identify a potential optimization target, not a guaranteed achievable saving.

## Comparison across carrier profiles

**Median emitted IP bytes per confirmed event, three seeds**. Brackets show observed min–max where relevant.

| Profile | Healthy | 30% event loss | 30% ACK loss | 70% ACK loss | Mixed + outage, warm |
|---|---:|---:|---:|---:|---:|
| ZPulse direct | **158.00** | **203.05** | **219.72** | **556.70** | **604.84** |
| MQTT-SN wrapper | 243.13 | 293.30 | 340.16 | 710.70 | 782.95 |
| CoAP wrapper | 174.00 | 224.19 | 241.97 | 613.08 | 669.77 |
| BPv7 wrapper | 263.64 | 338.85 | 366.64 | 928.91 | 1,002.16 |

These are **carrier-overhead results around a shared ZPulse engine**, not evidence that the direct profile outperforms optimized end-to-end MQTT-SN, CoAP or BPv7 deployments.

## Confirmation delay after durable acceptance

Virtual seconds, median across seeds of per-run median and p95:

| Profile | 30% ACK loss: median / p95 | 70% ACK loss: median / p95 |
|---|---:|---:|
| ZPulse direct | 0.25 / 3.25 | 3.25 / 55.25 |
| MQTT-SN wrapper | 0.25 / 1.25 | **1.25 / 35.25** |
| CoAP wrapper | 0.25 / 3.25 | 3.25 / 55.25 |
| BPv7 wrapper | 0.25 / 3.25 | 3.25 / 55.25 |

MQTT-SN's tested QoS-1 ACK-publication retry behavior confirmed faster in high ACK-loss conditions, at higher traffic cost. This is a selected-profile observation, not a universal protocol ranking.

## 1,000-event validation — 30% ACK loss

All four profiles: **1,000 accepted / 1,000 confirmed; zero duplicate events accepted**.

| Profile | Total IP B / confirmation | p95 acceptance→confirmation (virtual s) | p95 enqueue→confirmation (virtual s) |
|---|---:|---:|---:|
| ZPulse direct | **231.944** | 3.25 | 44.00 |
| MQTT-SN wrapper | 350.625 | **1.25** | **35.50** |
| CoAP wrapper | 255.432 | 3.25 | 44.00 |
| BPv7 wrapper | 389.682 | 3.25 | 44.25 |

One seed and one condition: this is a validation observation, not a statistical performance guarantee.

## Failure and post-confirmation accounting

With **100% application-ACK loss**, each profile durably accepted 32 events but confirmed **zero**. Each exhausted its retry limit without false success. Total emitted IP bytes were 161,792 (direct), 300,608 (MQTT-SN), 178,176 (CoAP) and 268,864 (BPv7); **bytes per confirmation is undefined**.

In a separate MQTT-SN experimental profile with **100% PUBACK loss**, 16/16 application events were confirmed, but the carrier emitted 190,608 bytes in total, **186,592 of them after application termination**. This is a stress result of the tested native-outbox/session policy, **not** a general MQTT-SN behavior or vendor benchmark. It demonstrates why cost measurement should continue until carrier queues have drained.

## Verification and limitations

- Five targeted tests passed; one repeated 70%-ACK-loss ZPulse case reproduced 39 deterministic fields exactly.
- Mixed conditions rejected 107 invalid events and 69 invalid ACKs.
- Fifteen sender and fifteen receiver reopen events were exercised.
- 12,071 duplicate event receptions were suppressed, distinct from zero duplicate **acceptances**.
- Three seeds describe a limited range, not confidence intervals.
- Host CPU/RSS include common Python and persistence overhead; no MCU or energy claim is supported.
- The study did not alter existing codecs, security protocol, transport adapters or production engine.

## Conclusion

ACK loss is a measurable source of traffic amplification even after durable destination acceptance. Optimizing ACK handling may reduce **total bytes per authenticated confirmed event**, but any proposed improvement must be measured against latency, failure states, replay safety and post-confirmation traffic. No improvement is claimed as implemented here.

See [ACK Optimization Proposals](ack-optimization-proposals.md) and the [Research Archive](README.md).
