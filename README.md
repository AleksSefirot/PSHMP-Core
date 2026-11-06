<img width="1672" height="941" alt="Киберсеть PSHMP CORE в горах" src="https://github.com/user-attachments/assets/53b8694c-1576-4484-84ab-e9a0a10d1fad" />


# PSHMP Core

**Proactive Self-Healing Mesh Protocol**

Hybrid L4 overlay network core for resilient multi-hop data delivery over existing IP infrastructure.

PSHMP Core is a **technology foundation** designed for organizations building high-availability communication platforms, edge networks, distributed content delivery, telecommunications overlays, and resilient transport systems. It adds adaptive relay paths, proactive recovery, and efficient acknowledgements **without replacing Layer 3 routing or existing hardware**.

---

## Why it exists

Distributed systems rarely enjoy perfect connectivity. Packet loss, jitter, node failures, overloaded links, and partial outages appear in cloud, edge, industrial, and telecom environments. Traditional recovery is often reactive: the path fails first, then the system reacts. By that time applications already feel the impact.

PSHMP Core treats **progressive degradation** as a first-class signal. Nodes continuously evaluate stability. When quality falls, the system starts reconstructing the affected delivery path—often before the path is completely unusable. The result is an overlay that keeps traffic moving with less disruption and less dependence on any single central controller.

---

## Hybrid operating model

| Mode | When | Behaviour |
|------|------|-----------|
| **Linear chains** | Stable network | Ordered multi-hop relay paths. Simple, predictable, optionally coordinated. |
| **Local mesh recovery** | Multiple nodes degrade (typically ≥3) | Collect candidates (DHT + Gossip), apply progressive fallback, quality gate, replace only the affected path. |

This hybrid approach keeps the common case simple while providing a robust escape hatch when several relays fail together. The underlying IP network remains unchanged—only the logical overlay path is rebuilt.

---

## Core capabilities

### K-Factor — continuous stability metric
Composite score from packet loss, jitter, RTT, and optional node-health indicators. Used both for **selecting relays** and for **early degradation detection**. Tunable weights and status bands (EXCELLENT → CRITICAL).

### Proactive Self-Healing
Priority-based recovery (CRITICAL / HIGH / MEDIUM / LOW) with adaptive cooldown. Reconstruction starts from degradation signals, not only from hard failures—shortening the window of visible service impact.

### Local Mesh Recovery + Fallback Ladder
Multi-level recovery that gradually relaxes quality and diversity requirements. Early levels demand high average K-Factor and failure-domain diversity; later levels prefer any working path over silence. A **quality gate** prevents installing a clearly worse path.

### Diversity-aware selection
Nodes can be grouped by logical failure domain (subnet, region, provider, AZ). During recovery the system prefers candidates from different groups to reduce correlated failures.

### Batch ACK + Gap List
Receivers report the highest contiguous sequence and explicit missing ranges instead of acknowledging every packet. Reduces control-plane overhead while enabling precise retransmission. Experimental observations under favourable traffic patterns show substantial savings (approaching the order of 75% in tested scenarios).

### Decentralised discovery
**Gossip** (push + anti-entropy) spreads node state and K-Factor updates. **Kademlia-style DHT** supplies candidates when a coordinator is unreachable. An optional Raft-based coordinator can provide stronger consistency when desired—it is not mandatory.

### Transport abstraction
UDP, DTLS, TLS, WebRTC DataChannels, and other bidirectional transports. TURN assistance where NAT traversal is required. Path logic stays independent of the concrete transport.

---

## Architecture layers

| Layer | Main components |
|-------|-----------------|
| **Control Plane** | Optional Coordinator, Raft cluster, Sharding, K-Factor Engine, Self-Healing Engine, Mesh State Machine |
| **Data Plane** | Chain Relay, PathFinder, Local Mesh Recovery, DataPath, Batch ACK + Gap List, Retransmit Cache |
| **Decentralization** | DHT (Kademlia-style), Gossip, Discovery facade, Membership |
| **Security & Transport** | TLS/DTLS, HMAC, Rate Limiting, optional Proof-of-Work, transport abstraction |
| **Infrastructure** | Persistence, Config, Prometheus metrics, structured logging |

---

## Reference formulas (experimental model)

**K-Factor**

```text
K = 0.45 × (1 − loss/0.35)
  + 0.25 × (1 − jitter/800)
  + 0.20 × (1 − rtt/1200)
  + 0.10 × battery
```

Terms are clamped to [0, 1]. Weights and normalisation constants are tunable per deployment.

**Candidate scoring**

```text
Score = 0.50 × KScore
      + 0.25 × LatencyScore
      + 0.15 × LossScore
      + 0.10 × GeoScore
```

**Mesh recovery fallback ladder (reference)**

```text
Level 0: MinAvgK ≥ 0.70, diversity required
Level 1: MinAvgK ≥ 0.65, diversity required
Level 2: MinAvgK ≥ 0.60, diversity required
Level 3: MinAvgK ≥ 0.52, diversity required
Level 4: MinAvgK ≥ 0.42, diversity optional
```

---

## Experimental validation

The Go implementation (v3.1) has been exercised primarily through **large-scale network emulation** (up to 5 000+ nodes) and limited real-network experiments. Under the tested scenarios the system demonstrated:

- recovery behaviour on the order of hundreds of milliseconds
- continued delivery under elevated loss and node churn
- substantial reduction of acknowledgement traffic with Batch ACK + Gap List

**These are experimental observations, not protocol guarantees.** Real performance depends on topology, traffic, and configuration. Physical validation and calibration are the responsibility of the integrating organization.

---

## Target use cases

- Edge and multi-site computing
- CDN and distributed content delivery
- Telecommunications and backhaul overlays
- VPN and secure overlay networks
- Industrial / IoT infrastructure with unstable links
- Hybrid cloud and geographically distributed services
- Any platform that needs multi-hop delivery with faster recovery than pure reactive failover

---

## Internet-Drafts (IETF)

Design principles and architecture are documented as Individual Internet-Drafts:

- [draft-kolomytsev-pshmp-core](https://datatracker.ietf.org/doc/draft-kolomytsev-pshmp-core/)
- [draft-kolomytsev-pshmp-overview](https://datatracker.ietf.org/doc/draft-kolomytsev-pshmp-overview/)
- `draft-kolomytsev-pshmp-core-overview-00` (submission in progress)

---

## Status

**Pilot-ready technology foundation — v3.1**

- Core mechanisms implemented in Go
- Emulation suite (demo / stress / extreme / realistic / live modes)
- Documentation and IETF drafts
- Suitable for evaluation, pilot integration, commercial licensing, OEM embedding, or full technology acquisition (including prior versions and design history)

PSHMP Core is **not** positioned as a finished mass-market product. Integrators are expected to tune thresholds, run physical tests, and adapt the system to their environment.

---

## Commercial options

- Commercial licensing
- OEM / product integration
- Pilot and proof-of-concept deployments
- Technology acquisition / IP transfer

Organizations interested in evaluation or partnership are welcome to contact the author.

---

## Contact

**Alexander Kolomytsev**  
Email: [giro.pandemik@gmail.com](mailto:giro.pandemik@gmail.com)

---

## License

Copyright © Alexander Kolomytsev. All rights reserved.

Source code and materials are provided for evaluation purposes only unless otherwise agreed under a commercial license or acquisition agreement.
