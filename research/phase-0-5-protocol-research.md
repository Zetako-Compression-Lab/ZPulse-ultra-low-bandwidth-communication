# Phase 0.5 — Protocol Research and Validation

**Date:** 12 October 2026. **Evidence:** internal Phase 0.5 report. **Status:** isolated experiments; no production wire-format freeze.

## Codec stability investigation

The original 32-bit codec's drift was traced to quantization and inverse-rounding effects in high-width coordinate profiles. In the selected-point census, **4,432 of 123,008** original emitted frames drifted on decode/re-encode. Standard configurations were stable.

A separate experimental decoder preserved original encoder output and tested a representative-coordinate adjustment. Across **3,100,000 seeded paired-profile cases**, the original implementation exhibited **119,085** drifts; the experiment exhibited **zero**, with **zero changes to encoder wire bytes**. This is sampled regression supported by domain reasoning, not exhaustive byte-stream validation. A distinct revision would be required before any changed decoding contract could ship.

## Secure envelope candidates

All candidates in the study used authenticated encryption with **full 16-byte tags**. Totals exclude network/carrier framing.

| Candidate | Header | Preset event | ACK | Structured no parameter | Structured with signed16 parameter |
|---|---:|---:|---:|---:|---:|
| A — minimal context-bound | 7 B | **36 B** | 28 B | 37 B | 42 B |
| B — standard secure | 22 B | **51 B** | 51 B | 52 B | 57 B |
| C — extended | 36 B | **87 B** | 65 B | 88 B | 93 B |

These were experimental designs, not certified wire formats. A depends on pre-provisioned context and persistent sequencing; B was selected for subsequent host proof-of-concept research. Smaller packets are not automatically safer, and no truncated tags or proprietary cryptography were used.

## Equivalent structured event packing

On a synthetic **2,048-tuple** corpus, half with a signed16 parameter:

| Body representation | Observed bytes |
|---|---:|
| Experimental structured bit packing | 8–13 |
| Equivalent optimized bit packing | **8–13** |
| Byte-aligned binary | 13–16 |
| Canonical CBOR array | 8–21 |
| Explicit-field Protobuf | 12–25 |
| Meshtastic private Data carrying CBOR | 13–26 |

The experimental and equivalent optimized bit-packing entries use the same implementation; **no compression advantage over equivalent optimal bit packing was demonstrated**. The 7-byte preset subset is less expressive and should not be compared as equivalent to all structured tuples. The Meshtastic wrapper was not a complete radio integration.

## Host CPU observations

Native C median CPU time per operation (five batches of 200,000):

| Implementation | Encode | Decode |
|---|---:|---:|
| Reference codec | 35.160 ns | 17.535 ns |
| Original 32-bit codec | 37.325 ns | 18.285 ns |
| Experimental representative decoder | 37.615 ns | 34.350 ns |

The experimental decoder adds decode overhead. Host timing cannot be extrapolated to embedded power use.

## Security experiment

Native libsodium experimental envelope results:

| Candidate | Event / ACK | Seal median | Open median |
|---|---:|---:|---:|
| A | 36 / 28 B | 217.63 ns | 223.55 ns |
| B | 51 / 51 B | 224.28 ns | 226.26 ns |
| C | 87 / 65 B | 240.59 ns | 252.19 ns |

An official RFC vector passed; **1,401 negative fixtures** were rejected; six event/ACK fixtures verified independently. These do not constitute an independent security audit.

## Fault model

Ten seeded simulated runs delivered **1,000/1,000 events exactly once**, under modeled 30% independent event/ACK loss, 10% duplication, reordering, 120-second outage and two restarts. Sixteen focused scenarios passed. This was a **model**, not an actual power-loss or physical radio experiment.

## RF estimates, not measurements

An illustrative LoRa airtime calculation (BW 125 kHz, CR 4/5, stated packet assumptions) gave:

| Candidate | SF7 event / ACK | SF12 event / ACK |
|---|---:|---:|
| A | 77.056 / 66.816 ms | 1,974.272 / 1,646.592 ms |
| B | 102.656 / 102.656 ms | 2,465.792 / 2,465.792 ms |
| C | 153.856 / 123.136 ms | 3,612.672 / 2,793.472 ms |

No actual radio airtime, current or battery endurance was measured. Calculations omit actual carrier overhead and depend on specified PHY settings.

## Next step

The candidate B secure host design was implemented and tested in [Phase 1](phase-1-secure-messaging.md). Internal sources and scripts remain private.
