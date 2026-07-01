# PSHMP Core v2.5

**Proactive Self-Healing Overlay Network Core**

A lightweight Go-based overlay protocol that turns any existing IP network into a proactive self-healing mesh infrastructure.

![PSHMP Core](https://github.com/AleksSefirot/PSHMP-Core/blob/main/preview.png)

## Key Features

- **Average route recovery time**: 380 milliseconds
- **Up to 75% reduction** in ACK/service traffic
- Works as a **transparent overlay** — no hardware or network replacement required
- Strong performance under high packet loss and node churn
- Built-in security (HMAC integrity, Rate Limiting, DDoS Protection)
- WebRTC DataChannel + TURN support

## Use Cases

- CDN and Edge Computing
- VPN and secure networks
- 5G and telecom infrastructure
- IoT and Smart City projects
- Distributed systems with unstable connections

## Quick Start (Demo)

```bash
# Clone the repository
git clone https://github.com/AleksSefirot/PSHMP-Core.git

# Go to demo directory
cd PSHMP-Core

# Run demo mode
go run cmd/emulator/main.go -mode=demo
Results

Demo mode: 87.9% successful delivery
Stress mode (25% packet loss + churn): 74.6% successful delivery
Real hardware tests: 0% packet loss on 3 devices

Technical Stack

Language: Go 1.23+
Transport: UDP + WebRTC
Storage: BoltDB
Containerization: Docker + Docker Compose

Documentation

One-Pager
Technical Presentation
Test Reports

Status
Pilot-Ready MVP — ready for pilot projects and technical evaluation.
Contact
Alexander Kolomytsev
Independent Developer (Russia)
Email: giro.pandemik@gmail.com
Phone: +7 (918) 399-89-36
