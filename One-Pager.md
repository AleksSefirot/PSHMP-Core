# PSHMP Core — One-Pager

**Hybrid L4-oriented overlay for multi-hop delivery under node and path degradation**  
Over existing IP — not instead of it. Technology foundation for pilot evaluation and transfer.

| | |
|---|---|
| **Author** | Alexander Kolomytsev |
| **Status** | Technology asset / pilot-ready foundation (Go) |
| **Public architecture** | [draft-kolomytsev-pshmp-core-overview-01](https://datatracker.ietf.org/doc/draft-kolomytsev-pshmp-core-overview/) (Informational Internet-Draft; not an IETF standard) |
| **Contact** | giro.pandemik@gmail.com |

---

## Problem

Multi-hop delivery often fails in practice not because “the Internet is down,” but because **intermediate nodes degrade**: loss, jitter, overload, partial failures. The path is still reachable, yet quality collapses. Typical reactions are late (timeouts), coarse (full resend), or manual.

IP routing keeps **reachability**. It does not by itself provide an application-facing layer that **repairs a logical relay path** and **retransmits only missing fragments** under degradation.

---

## What PSHMP Core is

A **logical delivery overlay** above ordinary IP:

- Builds **ordered relay chains** in normal conditions (single active path).
- Treats **progressive degradation** as a signal (K-Factor quality + Load class).
- **Rebuilds the affected path** using diversity-aware candidates (HybridDHT / gossip-assisted discovery, local recovery).
- Delivers in **sessions**: chunks, Batch ACK, gap list, **targeted retransmission**, backpressure.
- Optional coordinator for membership — **not** required on every data packet.
- Transports: UDP / DTLS / TLS-TCP (config-dependent).

**Does not** replace routers, OSPF, or BGP.  
**Does not** restore a full underlay partition.  
**Does not** claim simultaneous multipath send by default (backup chains = failover, not traffic split).

---

## How value appears (hypotheses for pilot — not guaranteed ROI)

| Mechanism | Business-facing effect |
|-----------|----------------------|
| Local path repair | Less “whole chain down” from one weak hop; fewer manual path resets |
| Targeted retransmit | Less bandwidth and time vs resending the full object |
| Load-aware ranking | Prefer not to place overloaded nodes (Load 5 excluded) into new chains |
| Overlay on existing IP | No forklift L3 redesign; software layer on hosts/gateways/edge |
| Asset transfer model | Buyer can own and extend the stack instead of long vendor lock-in |

Measured impact depends on **buyer topology, traffic, and configuration**. Lab/emulation figures are experimental, not portable SLAs.

---

## Architecture in one glance

```text
Application / Service
        ↓
   PSHMP CORE          control: K/Load, candidates, chain rebuild
                       data: ordered relay, ACK/gaps, retransmit
        ↓
 Transport (UDP / DTLS / TLS-TCP)
        ↓
 IP → Routers / underlay (unchanged)
```

---

## What the buyer evaluates

| Included in the technology story | On the integrator / buyer |
|----------------------------------|---------------------------|
| Linked core (chain, DHT, healing, DataPath, gossip, optional coordinator) | Secrets, trust policy, key lifecycle |
| Emulators and technical tests (as provided in evaluation package) | Field pilot on real topology |
| Architecture docs + public Internet-Draft overview | Capacity plan, SLO, compliance |
| Honest limits (single-path default, security building blocks ≠ full PKI) | Production hardening and ops |

---

## Commercial posture

- **Preferred frame:** technology evaluation → **full transfer / acquisition** (buyer control).  
- Also possible: OEM embedding, scoped pilot.  
- Not positioned as mandatory perpetual SaaS dependency on the author.

Public GitHub mirror holds evaluation-oriented materials (README, one-pager, presentation). Full source and transfer package under NDA / commercial process.

---

## One sentence

**PSHMP Core keeps multi-hop delivery useful when hops degrade—by repairing the logical path and resending only missing pieces—on top of the IP network you already have.**
