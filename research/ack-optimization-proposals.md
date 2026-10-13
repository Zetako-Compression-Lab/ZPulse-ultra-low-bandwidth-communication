# ZPulse — ACK Optimization Research Proposals

**Research date:** 13 October 2026  
**Status:** design hypotheses and conditional calculations only — **no optimization implemented**

This note documents the follow-up research directions identified after the [ACK-loss campaign](ack-loss-research.md). **Percentages are scenario-specific estimates, not measured gains; they are not additive.**

## Summary of candidate approaches

| ID | Research approach | Conditional estimated effect | Key trade-off |
|---|---|---|---|
| O1 | Bound and coordinate native retry budgets per authenticated event identity | Not yet quantified; 21.13% of traffic in one 30%-PUBACK-loss stress case was post-application-termination | Avoid prematurely stopping a transport still needed for valid application delivery |
| O2 | Coalesce 4–8 existing encrypted ACK packets into one transport datagram | Healthy full batches: **13.29–15.51% fewer total IP bytes per confirmation** | Batching delay, correlated loss, framing limits |
| O3 | One authenticated selective ACK for 4–8 accepted events | Healthy full batches: **37.50–43.75% fewer total IP bytes per confirmation** | Requires a new protocol version and durable multi-event confirmation semantics |
| O4 | Bounded repeat of the identical cached authenticated ACK | Under modeled 70% independent ACK loss: **8.24–11.76% fewer bytes**; can add **4.62–15.38%** at 30% loss | May amplify traffic under low or correlated loss |
| O5 | Compact ACK body, retaining a full authentication tag | Healthy isolated exchange: **5.70% fewer bytes** (158→149 B) | Identity/epoch ambiguity; explicit version required |
| O6 | RTT-informed retransmission timer | Unquantified | Incorrect timing inference after retries and delayed acknowledgements |

These numbers refer to **complete modeled IP-byte costs**, not a percentage reduction in the 51-byte encrypted event itself.

## O2 — Coalescing existing encrypted ACKs

The current 51-byte authenticated ACKs would remain intact. Four or eight ACKs could share a single IPv4/UDP header instead of using separate datagrams.

Under a deliberately idealized framing assumption (no extra framing bytes, full batches):

- Four ACKs: 158 → **137 B** per confirmed event (13.29% saving).
- Eight ACKs: 158 → **133.5 B** per confirmed event (15.51% saving).

The current adapter does **not** parse such batches. A future experiment must implement bounded framing and flushing, check the transport's usable payload limit and validate each ACK independently. Sparse emergency traffic may see little or no benefit.

## O3 — Authenticated selective ACKs

A hypothetical 51-byte group ACK could confirm four or eight **individually accepted and durably committed** events using a bounded selective bitmap.

With a full batch and healthy channel:

- Four events: 158 → **98.75 B** per confirmed event (37.50% saving).
- Eight events: 158 → **88.875 B** per confirmed event (43.75% saving).

This is an analytical illustration, **not a defined wire format or measured benchmark**. A missing event must never be acknowledged merely because a later sequence was accepted. A changing bitmap must never reuse an authenticated-encryption nonce. Crash recovery, out-of-order events, replay protection and per-event persistent state require review.

This approach targets bursty/multi-event traffic, not isolated SOS messages.

## O4 — Reusing a cached ACK ciphertext

The receiver could transmit a bounded second copy of an **identical previously authenticated ACK packet**, without creating a new encryption invocation or nonce. Under a simplified independent-loss model:

| ACK loss probability | Current policy model | Two copies each attempt | First single, later two |
|---|---:|---:|---:|
| 70% | 526.67 B/confirmation | 464.71 B (11.76% less) | 483.29 B (8.24% less) |
| 30% | 225.71 B/confirmation | Higher by 15.38% | Higher by 4.62% |

These are ratios of model expectations, not transformations of the measured finite-sample results. Correlated outages invalidate the independent-loss assumption. At 100% loss, extra ACKs can increase waste without confirming anything.

All retry budgets must remain bounded across replays and restarts.

## O5 and O6 — Secondary experiments

**O5: compact ACK.** A hypothetical reduction of the ACK body from 13 to 4 bytes would yield a 42-byte encrypted ACK and a 149-byte clean IPv4/UDP event+ACK exchange, while preserving a full 16-byte authentication tag. A distinct protocol version and proof of authenticated context binding would be required.

**O6: adaptive timer.** Use only authenticated, unambiguous timing observations and bounded policies. The current loss-only campaign cannot justify a numerical gain estimate. Premature retries and overlong waiting must both be evaluated.

## Required validation before adoption

Test isolated events, full and partial bursts, sustained traffic, data loss, ACK loss, correlated loss, outages, reorder, expiry, replay, crash recovery and resource exhaustion. Measure:

1. **All emitted IP bytes per authenticated, committed confirmation**, including unsuccessful traffic and native queue drain.
2. Confirmation latency (median, p95) and failures/expiry.
3. Zero false delivery confirmations and zero duplicate accepted events.
4. CPU, memory, durable storage and retry-budget exhaustion.
5. Correct authentication, nonce uniqueness, version negotiation and downgrade rejection.

**Do not combine or advertise the above percentages as achieved results.** None of O1–O6 has been implemented or validated as a production optimization.

See [ACK Loss Research](ack-loss-research.md) and the [Research Archive](README.md).
