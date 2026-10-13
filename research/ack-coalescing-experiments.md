# ACK Coalescing Under Network Loss — Experimental Results

**ZPulse Compression Lab | 13 October 2026**  
**Research status:** isolated experimental transport harness; no production deployment or protocol change.

## Abstract

We evaluated whether placing multiple **individually authenticated, 51-byte acknowledgement packets** into a single UDP datagram reduces total emitted bytes per durably confirmed event. On a clean simulated channel with full batches, groups of four and eight reduced total IP traffic by **13.29%** and **15.51%**, respectively. The benefit was workload-dependent: sparse traffic showed no byte savings, and an ACK-loss screening case showed **13.66% more traffic** and **60.15 seconds higher p95 confirmation latency** for maximum-eight batching. Separate two-process localhost experiments found a 9.36% median byte saving for maximum-eight batches, but increased real p95 latency. The negative findings are retained.

**No single batching configuration is established as universally better.** These results are not field-radio, WAN, power-consumption or production protocol measurements.

## Scope and methodology

- Existing codec, cryptography, application reliability engine, production UDP adapter and previous benchmark implementations remained unchanged; experiments were isolated.
- Each ACK remained the **original 51-byte authenticated ciphertext**, validated and committed independently. This is **transport coalescing**, not a new cumulative or selective ACK protocol.
- Maximum batch sizes: **1 (baseline), 4, 5, 6, 7 and 8**. Maximum eight uses 408 bytes of concatenated ACK content, within the test transport's stated 512-byte capacity. Experimental parsing was required; the production adapter does **not** support batches.
- Nominal flush timeout: 50 ms, with a separate 20 ms sensitivity study.
- Same message corpus, full authentication tags, window of 32, retry cap of 32, event lifetime of 3,600 seconds and three seeds (210228, 210229, 210230).
- Main screening: **114 runs**, **18,048 events submitted and accepted**, **18,000 confirmed**. The remaining 48 unconfirmed events occurred in deliberate total-ACK-loss controls.
- Simulated network: 100 kbit/s each direction, 50 ms propagation, 28-byte IPv4/UDP overhead per datagram and microsecond-resolution scheduler. Loss/duplication/delay applies to a whole datagram, including all ACKs in a batch.
- Primary cost: **all emitted IP bytes divided by authenticated, durably persisted confirmations**, including retries and post-termination emissions. No confirmed events means an **undefined** ratio, not zero.
- Timing in simulated cases is **virtual**; real process tests use host wall-clock timings.

## 1. Clean-channel results — full or near-full batches

128 events per run, three seeds; values below are median results.

| Max ACKs per datagram | IP bytes / confirmation | Total traffic saving | Actual ACKs / batch | p95 durable acceptance → confirmation |
|---|---:|---:|---:|---:|
| 1 (baseline) | 158.000 | 0.00% | 1.000 | 56.32 ms |
| 4 | 137.000 | **13.29%** | 4.000 | 87.52 ms |
| 5 | 135.688 | 14.12% | 4.923 | 97.92 ms |
| 6 | 134.812 | 14.68% | 5.818 | 108.32 ms |
| 7 | 134.156 | 15.09% | 6.737 | 118.72 ms |
| 8 | 133.500 | **15.51%** | 8.000 | 129.12 ms |

The byte savings come from sharing the IP/UDP header across existing ACK ciphertexts. They **do not** reduce the number of cryptographic verifications. Batching adds waiting time and can correlate failures when an entire datagram is lost.

## 2. Loss and outage screening — paired results

128 events, three seeds. Savings are paired **seed by seed**; negative savings mean extra traffic. Values are median [minimum–maximum] across seeds. Changes in p95 are differences relative to the immediate-ACK baseline.

| Condition | Max batch | Traffic saving | Change in p95 acceptance → confirmation |
|---|---:|---:|---:|
| 30% event loss | 4 | 9.03% [8.86–9.06] | +0.058 s |
| 30% event loss | 8 | 9.72% [9.31–9.74] | +0.066 s |
| 30% ACK loss | 4 | 1.94% [0.10–10.70] | +4.012 s |
| 30% ACK loss | 8 | 6.91% [−10.21–7.99] | +4.029 s |
| 70% ACK loss | 4 | 6.78% [−1.29–22.19] | +0.088 s |
| **70% ACK loss** | **8** | **−13.66% [−14.57–33.86]** | **+60.155 s** |
| Mixed faults + outage | 4 | 1.27% [−0.08–2.00] | +0.033 s |
| Mixed faults + outage | 8 | 2.56% [1.23–3.92] | +0.048 s |

Mixed faults include 30% loss each direction, 10% duplication, 20% added delay (up to 2 seconds), 2% invalid copies, a 120-second virtual outage and sender/receiver reopen. Median actual batch occupancy in this mixed condition was only **2.00** for max-four and **2.13** for max-eight.

At 70% ACK loss and seed 210229, maximum-eight batching increased traffic from **509.797 to 584.063 bytes per confirmation**, with p95 worsening by **80.206 seconds**. The shared loss of multiple ACKs in one datagram can amplify recovery cost.

These are descriptive results over three seeds, not confidence intervals or a general statistical ranking.

## 3. Validation at 1,000 events

One seed (210228), one run per condition. Every variant confirmed **1,000/1,000 events**, with zero duplicate accepted events.

| Condition | Max batch | IP bytes / confirmation | Paired saving | p95 acceptance → confirmation | p95 enqueue → confirmation |
|---|---:|---:|---:|---:|---:|
| Clean | 1 | 158.000 | 0.00% | 0.056320 s | 6.110320 s |
| Clean | 4 | 137.000 | 13.29% | 0.087520 s | 6.135200 s |
| Clean | 8 | 133.500 | 15.51% | 0.129120 s | 6.151520 s |
| 30% ACK loss | 1 | 224.518 | 0.00% | 3.056322 s | 19.750248 s |
| 30% ACK loss | 4 | 187.416 | 16.53% | 3.074882 s | 18.434726 s |
| 30% ACK loss | 8 | 173.016 | 22.94% | 3.086801 s | 16.872486 s |

The observed loss rate differed between variants because batching changes the number of datagrams exposed to the fault model. Therefore, the 16–23% savings in the 30%-loss validation are **not** a universal structural saving.

A follow-up on the previously unfavorable 70%-ACK-loss seed 210229 at 1,000 events found savings of **17.73%** for max-four and **20.32%** for max-eight. That seed was chosen **after** observing its poor 128-event result; this is a sensitivity follow-up, not independent confirmation. It does not erase the earlier regressions.

## 4. Sparse traffic and flush-delay trade-off

At **one event every two seconds**, all three configurations (1, 4, 8) emitted **158 bytes per confirmed event**: only one ACK was available at each flush. The batching variants increased p95 acceptance-to-confirmation from **56.32 ms to 106.32 ms**.

With a **20 ms flush** on a clean channel, max-eight averaged four ACKs per batch, yielding **137 B/confirmation** and **88.56 ms** p95, compared with **133.5 B** and **129.12 ms** at 50 ms flush. Batch limit and flush timeout must be tuned jointly to the workload.

A receiver reopen while three ACKs remained in the volatile batching buffer caused no accepted-event loss; all 64 events per test were ultimately confirmed by the original persistent retry path. This was a controlled reopen, **not** a physical power-loss test.

## 5. Separate-process localhost experiments

Six runs of 256 events, two repetitions per variant, sender and receiver in separate real processes using local UDP and host clocks, without simulated loss, delay or bandwidth limits. All **1,536/1,536 events** were authenticated, accepted and confirmed, with zero duplicate accepted events.

| Max batch | IP bytes / confirmation (median [range]) | Paired saving (median [range]) | Observed ACKs / batch | Real p95 acceptance → confirmation |
|---|---:|---:|---:|---:|
| 1 | 162.012 [161.703–162.320] | 0.00% | 1.000 | 91.55 ms |
| 4 | 169.523 [160.797–178.250] | **−4.65% [−10.23–0.94]** | 2.194 | 174.01 ms |
| 8 | 146.848 [146.539–147.156] | **9.36% [9.34–9.38]** | 2.035 | 173.55 ms |

These timings include real CPU, filesystem synchronization and host scheduling. The small number of runs does not establish statistical significance. They are **not** radio or WAN benchmarks.

## 6. Negative controls and integrity

- In the 114 main runs: **18,048 accepted, 18,000 confirmed**, with 48 unconfirmed only in intentional total-ACK-loss controls; **zero duplicate acceptances**.
- **54 invalid events and 51 invalid ACK entries** rejected.
- Eight focused tests passed, including invalid authentication inside an otherwise valid batch and replayed batches.
- With 100% ACK loss, all 16 tested events were accepted but **none confirmed** in any variant. Every variant reached the retry limit; no false success was reported.
- Original production components and previous experiments remained unchanged, with **74 protected file hashes** matching before and after.

## 7. Interpretation and next experiments

ACK coalescing can save IP header bytes for **bursty or sustained event traffic**. It can worsen confirmation latency and, under some loss traces, increase total traffic. For sparse or urgent messages, immediate ACKs are a safer baseline.

Next work should evaluate deadline-aware batching, multiple loss models (including correlated failures), multi-seed scale-up, queue behavior after crash and real transport/energy measurement. An adaptive policy should be treated as a **new hypothesis** until implemented and validated.

**No production coalescing variant has been enabled. No secure protocol change, new codec version or hardware claim is implied.**

---

*ZPulse is an experimental research project of Zetako Compression Lab. Public results are reported without proprietary source code or implementation internals. The numerical experiments are documented here, but the private harness is not available for independent execution.*
