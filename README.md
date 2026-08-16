# ZPulse

## Meaning in a few bytes.

**ZPulse is an ultra-low-bandwidth communication protocol developed and under active development within Zetako Compression Lab.**

It explores how essential operational meaning can remain communicable when bandwidth, energy, connectivity or device resources become severe constraints.

ZPulse is designed around a simple principle:

> When the full message cannot reasonably travel, transmit the information that matters most.

Public ZPulse documentation intentionally describes the **engineering objective, application domains and measured outcomes** without publishing the proprietary wire format, semantic dictionaries, protocol profiles or encoding internals.

---

## Where ZPulse fits inside Zetako

ZPulse is part of a broader Zetako research continuum around **communication, data reduction and efficient representation**.

| Technology | Focus |
|---|---|
| **ZNode** | Sovereign human communication and collaboration infrastructure |
| **ZPulse** | Ultra-low-bandwidth communication for constrained and degraded links |
| **ZNano** | Lossless compression for structured embedded telemetry |
| **ZCaps** | General-purpose adaptive lossless compression research |
| **ZChain** | Blockchain-native lossless compression |

The relationship is deliberate.

**ZNode** addresses rich organizational communication when normal infrastructure is available.

**ZPulse** addresses the opposite edge of the problem: environments where only a very small amount of information may be practical to transmit.

**ZNano** addresses machine-generated structured telemetry where exact reconstruction must be preserved while reducing transport cost.

ZCaps and ZChain extend Zetako's work on efficient representation into general-purpose and blockchain-specific data paths.

The common thread is **moving useful information efficiently under real engineering constraints**.

---

## What ZPulse communicates

The current research includes compact representations of operational information such as:

- geographic position;
- predefined intent or status;
- distress and assistance states;
- field-operation events;
- small device-originated operational signals.

The protocol is designed for deterministic, very small application payloads suitable for constrained embedded endpoints.

A representative ZPulse research configuration can carry **position plus predefined operational meaning in only a few bytes**. Exact protocol construction and profile definitions are intentionally not published in this repository.

---

## Relevant environments

ZPulse research is particularly relevant where connectivity can be intermittent, expensive, energy-limited or extremely narrow:

- maritime and offshore operations;
- distress and emergency communication;
- disaster-response environments;
- remote field operations;
- satellite-connected systems;
- LPWAN and constrained radio links;
- compact embedded or battery-powered devices.

ZPulse is **not presented as a replacement for certified maritime distress systems, EPIRBs, GMDSS equipment or regulated emergency infrastructure**.

---

## Maritime research continuity at Zetako

Maritime communication is not a new problem area for Zetako.

Zetako previously participated in **Plug and Play Maritime**, in an ecosystem that included maritime partners such as **Euronav**, CMB, DXC Technology and the Port of Antwerp. Zetako also carried out a **proof of concept with Euronav** focused on remote vessel connectivity under severe bandwidth constraints.

That earlier work followed a different Zetako technology path and should **not** be interpreted as a claim that ZPulse itself was deployed in the Euronav proof of concept.

It is relevant because it demonstrates continuity in the underlying research question: **how to move useful information when connectivity is limited**.

Public references:

- [Plug and Play Maritime — Batch announcement](https://www.linkedin.com/posts/plugandplaymaritime_openinnovation-maritime-plugandplay-activity-6873941758451957760-oSbh)
- [Silicon Luxembourg — Zetako Makes Its Way To The Maritime Sector](https://www.siliconluxembourg.lu/zetako-makes-its-way-to-the-maritime-sector/)

---

## ZPulse + ZNode

ZPulse can form the constrained edge of a richer sovereign communication environment.

```text
constrained field endpoint
          |
          | ultra-small operational signal
          v
 radio / LPWAN / satellite transport
          |
          v
      gateway layer
          |
          v
        ZNode
          |
          +-> alerting
          +-> operational context
          +-> coordination
          +-> audit and response workflows
```

In this model, **ZPulse is concerned with getting essential information across the constrained link**, while **ZNode provides the richer environment in which people can coordinate and manage the response**.

The transport and gateway architecture are deliberately not fixed by the public protocol description.

---

## Relationship to ZNano

ZPulse and ZNano are complementary but solve different problems.

**ZNano** focuses on reducing structured telemetry while preserving exact reconstruction.

**ZPulse** focuses on extremely compact operational communication where the communicating endpoints share enough context to represent the required meaning efficiently.

A useful distinction is:

```text
ZNano  -> preserve the structured data, reduce the bytes
ZPulse -> preserve the required operational meaning, minimize transmission
```

---

## Engineering characteristics

Publicly disclosed characteristics of the ZPulse research include:

- ultra-small deterministic application payloads;
- embedded-oriented implementation work;
- integer-oriented processing suitable for constrained systems;
- no dependence on rich-message infrastructure at the edge;
- transport-agnostic research across constrained communication environments;
- single-message operation that does not require a long communication history to express an event.

Detailed codec construction, bit allocation, semantic tables, wire compatibility rules, security profiles and optimization techniques remain internal research material.

---

## Research and development

ZPulse remains under active development within Zetako Compression Lab.

Current research areas include constrained semantic communication, embedded implementation, maritime and emergency use cases, transport integration, gateway architecture and integration with the wider Zetako communication stack.

This public repository is an **engineering evidence and positioning layer**, not a publication of the complete proprietary protocol specification.

---

## Zetako

ZPulse is developed by **Zetako S.à r.l.**, Luxembourg.

- Website: https://zetako.ai/
- Engineering organization: https://github.com/Zetako-Compression-Lab
- Contact: contact@zetako.ai

---

<sub>Public examples describe research directions and specific engineering contexts. ZPulse is not represented as certified emergency equipment, a guarantee of radio range, or a replacement for regulated maritime distress infrastructure.</sub>
