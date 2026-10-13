# ZPulse — Public Research Results (October 2026)

**Status:** experimental software research; not a production or certified emergency communication system.  
**Research date:** 13 October 2026.  
**Scope:** public measurements and limitations only. Source code, private protocol internals, key management details, fixtures and test harnesses are not released.

## Executive summary

ZPulse investigates compact, authenticated messaging under constrained and disrupted connectivity. The research has progressed from a compact position-and-status representation to a host-based, encrypted and persistently acknowledged messaging proof of concept.

**The main result is engineering feasibility, not a proven commercial or protocol-level advantage.** A compact 7-byte application representation becomes a 51-byte authenticated event in the tested secure host profile. Under the tested fault conditions, all 1,000 events were acknowledged with no duplicate events accepted. Competitive transport experiments show lower overhead for direct carriage of the same secure payload, but do not establish superiority over complete native MQTT-SN, CoAP or BPv7 systems.

## Research milestones

| Milestone | Publicly reportable result | Important boundary |
|---|---|---|
| Compact application representation | 7-byte position + predefined-status research configuration | Quantized, controlled-loss position; no complete transport/security framing |
| Codec audit | Standard profiles stable in the tested corpus; high-width 32-bit variant revealed re-encode drift | Original implementation remains unchanged |
| Experimental correction | Zero drift observed in 3.1 million seeded paired-profile cases | Research correction only; no production migration |
| Secure host proof of concept | 51-byte authenticated event and 51-byte authenticated acknowledgement | Trusted-host configuration, not a portable MCU or production security certification |
| Host fault tests | 1,000/1,000 acknowledged in two disrupted UDP process tests; zero duplicate events accepted | Local OS UDP, injected faults and accelerated simulated outage |
| Competitive carrier benchmark | All four profiles confirmed 1,000/1,000 events in clean and disturbed scenarios | Same ZPulse security and reliability engine used across all profiles |

## Secure messaging proof of concept

A host-based sender and receiver were tested with authenticated encrypted events, destination acknowledgements, persistent queue state and duplicate suppression.

- Authenticated application event: **51 bytes**.
- Authenticated application ACK: **51 bytes**.
- Minimum IPv4 + UDP accounting: **79 bytes per datagram**, or **158 bytes per clean event/ACK exchange** before link-layer overhead.
- 24 regression tests passed in the final Phase 1 run.
- Two independent disrupted local UDP/process runs each acknowledged **1,000/1,000** events and accepted **zero duplicate events**.
- Fault campaigns included simulated packet loss, duplication, reordering, corrupted packets, invalid acknowledgements, process restarts and a 120-second **simulated-time** network outage.

These results do **not** establish guaranteed delivery on physical radio or satellite links, resistance to jamming, hardware energy consumption, embedded portability, or production-ready key lifecycle and recovery.

## Competitive carrier study

### Comparison design

The **same ZPulse 51-byte authenticated payload and common application-level reliability engine** were carried directly over UDP or wrapped in experimental MQTT-SN, CoAP and BPv7 carrier profiles. Each condition used 1,000 events, matching useful information, shared fault policy and common application acknowledgement requirements.

This measures the **incremental overhead and timing behavior of the chosen carriers**, not a head-to-head comparison of independent native end-to-end secure stacks.

The network was modeled as IPv4/UDP at 100 kbit/s per direction, 50 ms propagation and 250 ms simulation ticks. The disturbed condition used 30% packet loss, 10% duplication, 20% additional delay, invalid copies, and a common 120-second simulated-time outage. Byte counts include transmitted events, acknowledgements, control/setup and retransmissions plus IPv4/UDP headers; they exclude radio MAC/PHY and other infrastructure.

### Clean network — 1,000 confirmed events

| Carrier profile | Total bytes emitted | Bytes / confirmed event | Median enqueue-to-confirmation (virtual seconds) | p95 (virtual seconds) |
|---|---:|---:|---:|---:|
| ZPulse direct | 158,000 | 158.00 | 11.75 | 22.25 |
| CoAP + common application | 174,000 | 174.00 | 11.75 | 22.25 |
| MQTT-SN + common application | 242,144 | 242.14 | 13.375 | 24.00 |
| BPv7 + common application | 265,444 | 265.44 | 12.00 | 22.50 |

### Disturbed network — 1,000 confirmed events

| Carrier profile | Total bytes emitted | Bytes / confirmed event | Median enqueue-to-confirmation (virtual seconds) | p95 (virtual seconds) |
|---|---:|---:|---:|---:|
| ZPulse direct | 351,708 | 351.71 | 183.25 | 246.75 |
| CoAP + common application | 388,332 | 388.33 | 183.50 | 247.00 |
| MQTT-SN + common application | 663,114 | 663.11 | **162.00** | **195.00** |
| BPv7 + common application | 590,346 | 590.35 | 183.75 | 247.25 |

In the disturbed scenario, the direct carrier emitted approximately **9.4% fewer bytes than CoAP**, **40.4% fewer than BPv7** and **47.0% fewer than MQTT-SN** in these particular experimental wrappers. This is **not** evidence that ZPulse is that percentage more efficient than optimized native implementations of those protocols.

MQTT-SN's selected QoS behavior achieved faster confirmation in the disturbed condition while using more transmitted bytes. Cold-start/session timing affects that comparison; warm-session and independent-native-stack comparisons remain open.

### Host resource observations

The disturbed runs reported approximately 21.5 seconds of aggregate host CPU time for the direct profile, 29.6 for CoAP, 45.2 for MQTT-SN and 19.1 for BPv7. These measurements depend on the chosen Python libraries, scheduling and shared application engine; they **must not** be interpreted as protocol-intrinsic CPU rankings. Worker RSS was approximately 94–96 MiB, not a standalone embedded footprint.

No physical energy, battery-life, radio range, or provider cost measurements have been performed.

## Compact structured messaging

A deterministic structured-event research representation was tested on a 2,048-tuple synthetic corpus:

- Core structured body: **8 bytes**.
- One-parameter structured body: **13 bytes**.
- Equivalent optimized bit packing: **the same 8–13 bytes**.
- Existing preset subset: **7 bytes** for the defined position-plus-index case.

The research did not demonstrate a compression advantage over equivalent optimized binary packing. The potential value of structured semantics is clearer interpretation and extensibility, subject to domain-specific validation.

## Research limitations

1. **No native-stack victory claim.** The competitive study wrapped the same ZPulse secure application in carrier profiles; it did not compare fully independent, production-optimized and equally secured MQTT-SN, CoAP or BPv7 deployments.
2. **No physical network evidence.** The benchmark used host sockets and simulated network time/faults, not LoRa, satellite, maritime VHF or measured RF links.
3. **No embedded power evidence.** Host CPU/RSS and theoretical energy estimates cannot establish MCU footprint or battery endurance.
4. **No production security certification.** The proof of concept lacks a fully qualified provisioning, rotation, revocation, cloning/rollback recovery and deployment lifecycle.
5. **No emergency-service certification.** ZPulse is not an EPIRB, GMDSS component or replacement for regulated rescue equipment.
6. **No verified commercial differentiation.** A smaller wrapper around the same payload is not proof of a new compression principle or a commercially meaningful advantage.

## Next validation priorities

- Compare complete native stacks with equivalent authentication, durable delivery and recipient-confirmation guarantees.
- Repeat cold-start and pre-established-session scenarios over multiple seeds and independent runs.
- Measure identical workloads on real constrained hardware, including radio airtime, current draw and end-to-end delivery cost.
- Investigate the trade-off between acknowledgement latency, retransmission overhead and total bytes per authenticated delivered event.
- Validate a concrete industrial or emergency communication problem before productization.

## Disclosure policy

This repository shares high-level research findings, measured aggregate results, methods and limitations. It intentionally does **not** publish the proprietary codec implementation, binary wire specification, protocol internals, semantic dictionaries, private test fixtures, credentials, cryptographic material or production source.

**Research evidence, not a product claim.** ZPulse is under development at Zetako Compression Lab.
