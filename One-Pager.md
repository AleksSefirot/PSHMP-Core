# PSHMP Core v3.1 — One-Pager

**When the network starts to fail — the path rebuilds itself.**

Hybrid L4 Overlay for Proactive Self-Healing and Resilient Multi-Hop Delivery

---

## The real problem

Most distributed systems still recover the old way: something breaks, the system notices, then it reacts. By that moment the user already sees freezes, gaps, or timeouts.

This is especially painful in:

- Edge and multi-site platforms
- CDN and content delivery under lossy last-mile or inter-region links
- Telecom and backhaul overlays
- VPN and secure networks stretched across unreliable segments
- Industrial and IoT environments where links are never perfect

Hardware replacement and L3 redesign are expensive. Waiting for full failure is too late. What is missing is a **software overlay layer** that lives above existing IP, continuously measures stability, and rebuilds delivery paths *before* the application feels the outage.

---

## What PSHMP Core actually does

PSHMP Core is a **hybrid L4 overlay network core**.

Under normal conditions it builds **linear multi-hop relay chains** — simple, ordered, easy to reason about. When several nodes on a path start to degrade, it does not wait for a hard failure. It switches that path into **local mesh recovery**:

1. Collects alternative candidates via DHT and Gossip
2. Applies a progressive fallback ladder (quality and diversity requirements are relaxed step by step)
3. Runs a quality gate so a clearly worse path is not installed
4. Replaces only the affected chain — the rest of the network stays untouched

The underlying IP routing does not change. Only the logical overlay path is rebuilt.

At the same time every node is scored by **K-Factor** — a continuous stability metric. That score drives both "who should be in the chain" and "when is it time to start healing". Acknowledgements use **Batch ACK + Gap List**: the receiver reports the highest contiguous sequence and the exact missing ranges, so the sender retransmits only what is lost. Control traffic stays light even when data volume is high.

An optional coordinator (including Raft) can be used when stronger consistency is needed. It is not mandatory — discovery and recovery continue through DHT and Gossip if the coordinator is unreachable.

---

## Why this is different

| Typical reactive approach | PSHMP Core |
|---------------------------|------------|
| Waits for hard failure | Starts from progressive degradation (K-Factor) |
| Global or slow reconvergence | Local path rebuild for the affected chain |
| Single "best path" mindset | Hybrid: linear chains in the common case, mesh recovery under stress |
| Little attention to correlated failures | Diversity-aware selection across failure domains |
| Per-packet or heavy ACKs | Batch ACK + explicit Gap List |
| Strong dependence on a central controller | Coordinator optional; decentralised mode remains available |
| Requires changes to L3 or hardware | Pure L4 overlay on top of existing IP |

The design goal is not "another mesh". The goal is a **practical resilience layer** that organizations can put under their existing products.

---

## How stability is measured (K-Factor)

Nodes are not treated as simply "up" or "down". PSHMP Core uses a composite score:

```text
K = 0.45 × (1 − loss / 0.35)
  + 0.25 × (1 − jitter / 800)
  + 0.20 × (1 − rtt / 1200)
  + 0.10 × battery
```

- **loss** — packet loss ratio (0…1)
- **jitter**, **rtt** — in milliseconds
- **battery** — optional node-health term (stationary nodes can fix it at 1.0)

Each term is clamped so extreme values cannot dominate. The resulting K is mapped to status bands (EXCELLENT → GOOD → ACCEPTABLE → DEGRADED → CRITICAL). Thresholds are configuration, not hard protocol constants. Integrators tune them to their traffic and topology.

When ranking candidates for a new or replacement chain the system uses a second weighted score:

```text
Score = 0.50 × KScore + 0.25 × LatencyScore + 0.15 × LossScore + 0.10 × GeoScore
```

So path decisions are driven by a continuous view of quality, not by binary reachability alone.

---

## How recovery works under stress

When three or more nodes on the same chain degrade, local mesh recovery activates. Candidates are gathered, recently failed nodes are excluded, and a **fallback ladder** is applied:

```text
Level 0: MinAvgK ≥ 0.70, diversity required
Level 1: MinAvgK ≥ 0.65, diversity required
Level 2: MinAvgK ≥ 0.60, diversity required
Level 3: MinAvgK ≥ 0.52, diversity required
Level 4: MinAvgK ≥ 0.42, diversity optional
```

Early levels insist on quality and spread across failure domains (subnet, region, provider, AZ). Later levels prefer any working path over silence. A quality gate still blocks a clearly inferior replacement. The result is graceful degradation instead of an abrupt collapse of the delivery path.

Priorities and cooldowns (CRITICAL / HIGH / MEDIUM / LOW) prevent the system from thrashing when many nodes fluctuate near a threshold.

---

## Architecture at a glance

| Layer | What lives there |
|-------|------------------|
| **Control Plane** | Optional Coordinator, Raft, Sharding, K-Factor Engine, Self-Healing Engine, Mesh State Machine |
| **Data Plane** | Chain Relay, PathFinder, Local Mesh Recovery, DataPath, Batch ACK + Gap List, Retransmit Cache |
| **Decentralization** | Kademlia-style DHT, Gossip (push + anti-entropy), Discovery facade, Membership |
| **Security & Transport** | TLS/DTLS, HMAC, Rate Limiting, optional Proof-of-Work, transport abstraction (UDP / DTLS / TLS / WebRTC) |
| **Infrastructure** | Persistence, configuration, Prometheus metrics, structured logging |

---

## What the numbers mean (and what they do not)

The Go implementation (v3.1) has been exercised through **large-scale network emulation** (thousands of nodes) and limited real-network experiments. Under the tested scenarios the system showed recovery behaviour on the order of hundreds of milliseconds, continued delivery under elevated loss and churn, and a substantial reduction of acknowledgement traffic with Batch ACK + Gap List (favourable cases approaching the order of 75%).

These are **experimental observations**, not contractual guarantees. Real performance depends on topology, load, and configuration. Any serious deployment must calibrate thresholds and validate on the target infrastructure. That is by design: PSHMP Core is a foundation to be tuned, not a black-box appliance with fixed SLAs.

---

## Who benefits

- **CDN and Edge platforms** — better behaviour under lossy or fluctuating paths without redesigning the whole delivery network
- **Telecom and backhaul** — local recovery when segments degrade, without waiting for global reconvergence
- **VPN and secure overlay vendors** — self-healing multipath capability on top of existing tunnels
- **Industrial and IoT** — more resilient transport where links are inherently unstable
- **System integrators and OEMs** — a ready resilience layer to embed into larger products
- **Strategic buyers** — a complete technology asset (implementation + architecture + IETF drafts + design history) for acquisition or exclusive licensing

---

## Business model

PSHMP Core is offered as a **technology foundation**, not as a finished mass-market product.

| Model | Description |
|-------|-------------|
| **Evaluation / Pilot** | Access for technical assessment and controlled trials on the customer's infrastructure |
| **Commercial license** | Right to use and integrate in production products |
| **OEM embedding** | Integration into a partner's commercial offering under agreed terms |
| **Technology acquisition** | Full IP transfer: current core, prior versions, design drafts, documentation, and related materials |

The goal of an engagement is usually one of three outcomes: a successful pilot that leads to a license, a long-term OEM relationship, or a clean technology acquisition.

---

## Current status

- Implementation: **PSHMP Core v3.1** (Go)
- Validation: large-scale emulation + limited real-network tests
- Documentation: architecture materials + Individual Internet-Drafts on IETF Datatracker
- Positioning: **pilot-ready technology foundation**

It is ready for serious technical evaluation. It is not claimed to be a turnkey product with fixed production SLAs out of the box.

---

## Next step

If your organization builds or operates systems where connectivity is imperfect and recovery time matters, the practical next step is a focused technical discussion and, if there is mutual interest, a scoped pilot on your topology.

**Alexander Kolomytsev**  
giro.pandemik@gmail.com

Repository: https://github.com/AleksSefirot/PSHMP-Core  
IETF drafts: search "kolomytsev pshmp" on [datatracker.ietf.org](https://datatracker.ietf.org)

---

*PSHMP Core — a resilience layer for networks that cannot afford to wait until the path is already dead.*
