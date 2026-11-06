<img width="1672" height="941" alt="Киберсеть PSHMP CORE в горах" src="https://github.com/user-attachments/assets/53b8694c-1576-4484-84ab-e9a0a10d1fad" />


# PSHMP Core

**Hybrid L4-oriented overlay for multi-hop delivery under node and path degradation.**

Runs **over existing IP** — does not replace routers, OSPF, or BGP.  
Technology foundation for evaluation, pilot integration, OEM embedding, or full technology transfer.

---

## What it is

PSHMP Core is a **logical delivery layer** above ordinary IP reachability.

Under stable conditions it builds **ordered relay chains** and forwards application fragments hop by hop while preserving fragment identity.

When nodes on a path **degrade**, it can:

- detect quality loss early (K-Factor + Load),
- select alternative candidates (HybridDHT / gossip-assisted discovery),
- **rebuild the affected path** (local recovery — not a full underlay repair),
- retransmit **only missing fragments** (session Batch ACK + gap list).

It does **not** require changing Layer-3 routing.  
It does **not** restore a complete underlay partition (no IP path → no overlay path).

---

## Hybrid operating model

| Mode | When | Behaviour |
|------|------|-----------|
| **Ordered chains** | Stable network | One active multi-hop relay path. Predictable forwarding. |
| **Local mesh-style recovery** | Path / node degradation | Expand candidates, apply fallback and diversity rules, quality gate, replace the **affected** logical path. |

Normal forwarding is **single-path** (one active chain).  
Backup chains support **failover**, not parallel multipath send by default.  
“Mesh” here means **candidate selection and path reconstruction**, not flooding every fragment on all links.

---

## Core building blocks

| Block | Role |
|-------|------|
| **K-Factor** | Continuous node quality signal (loss, jitter, RTT, optional health). |
| **Load (1–5)** | Utilization class in ranking (`effectiveK = K × loadFactor`; Load 5 excluded). |
| **HybridDHT** | Production candidate source (Normal / Stress / Mesh modes). |
| **ChainManager / ChainRelay** | Build and hold primary (+ backup) chains; first-hop forwarding. |
| **Self-Healing** | Priority recovery when degradation is reported. |
| **DataPath** | Chunking, session identity, Batch ACK, gap list, targeted retransmit, backpressure. |
| **Gossip** | Peer state propagation. |
| **Coordinator (optional)** | Registration / heartbeat; **not** required on every data packet. |
| **Transport** | UDP / DTLS / TLS-TCP (config-dependent) via protocol switcher. |

---

## Architecture (short)

```text
Application / Service
        ↓
   PSHMP CORE          ← logical overlay (path + session delivery)
        ↓
 Transport carriers    ← UDP / DTLS / TLS-TCP
        ↓
      IP layer
        ↓
 Routers & underlay    ← OSPF / BGP / physical links (unchanged)
```

**Control plane:** peer state, K/Load, candidate selection, chain rebuild.  
**Data plane:** ordered relay of the same fragment identity, ACK/gaps, retransmit.

---

## What this is not

- Not a replacement for IP routing or a new global internet fabric  
- Not simultaneous multipath by default  
- Not a full PKI / complete production security product out of the box  
- Not a guaranteed SLA from lab numbers alone  
- Not “heal the cable” when underlay is fully partitioned  

Security building blocks (transport modes, selected MAC, admission options) exist; **deployment trust, keys, and policy are integrator responsibilities**.

---

## Status

**Technology asset / pilot-ready foundation** (Go implementation).

- Core control + data path implemented and linked as one ecosystem  
- Emulators and component / ecosystem tests  
- Public architecture write-up (IETF Individual Internet-Draft)  
- Suitable for **technical evaluation** and discussion of **licensing / OEM / acquisition**

Integrators are expected to tune thresholds, wire secrets, and validate on their topology.  
Lab and emulation results are **experimental observations**, not portable guarantees.

---

## Internet-Draft

Architecture overview (Informational, work in progress — **not** an IETF standard):

- [draft-kolomytsev-pshmp-core-overview-01](https://datatracker.ietf.org/doc/draft-kolomytsev-pshmp-core-overview/)  
- HTML: https://www.ietf.org/archive/id/draft-kolomytsev-pshmp-core-overview-01.html  

---

## Repository contents (this public mirror)

This public repository is oriented to **evaluation materials**:

- `README.md` — this file  
- Presentation / one-pager materials (as uploaded)  
- Links to the public Internet-Draft  

Full source tree, internal tests, and transfer package are provided under a commercial evaluation or acquisition process—not as an open-source mass product dump unless separately agreed.

---

## Commercial posture

- Technology evaluation / pilot  
- OEM embedding  
- **Full technology transfer** (preferred discussion frame: buyer control of the asset)  
- Optional bounded handoff support if agreed  

Not positioned as a mandatory long-term SaaS lock-in from the author.

---

## Contact

**Alexander Kolomytsev**  
Email: giro.pandemik@gmail.com

---

## License / rights

Copyright © Alexander Kolomytsev. All rights reserved.

Materials in this repository are for **evaluation and discussion** unless otherwise agreed in writing (commercial license or acquisition).

