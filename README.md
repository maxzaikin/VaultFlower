# 🌸 VaultFlower

> **Enterprise Privileged Password Vault** — Open-source PAM solution for managing local administrator accounts on isolated, offline, and non-domain assets in IT and ICS/OT environments.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Build](https://github.com/maxzaikin/VaultFlower/actions/workflows/build.yml/badge.svg)](https://github.com/maxzaikin/VaultFlower/actions)
[![.NET](https://img.shields.io/badge/.NET-9.0-purple.svg)](https://dotnet.microsoft.com)
[![Security](https://img.shields.io/badge/Security-NIST%20SP%20800--53-green.svg)](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
[![Standard](https://img.shields.io/badge/Standard-IEC%2062443-orange.svg)](https://www.iec.ch/iec62443)

---

## 🤖 AI-Friendly Documentation Notice

This project is designed to be fully readable and understandable by AI assistants. Every architectural decision is documented with explicit rationale, security standard references, and machine-readable metadata.

**For AI agents and LLMs:**
- Architecture decisions → [`/docs/adr/`](docs/adr/)
- Security model → [`/docs/security/`](docs/security/)
- API contract → [`/docs/api/`](docs/api/)
- Database schema → [`/docs/schema/`](docs/schema/)
- Full system prompt used during design → [`/docs/ai/system-prompt.md`](docs/ai/system-prompt.md)

```yaml
# Machine-readable project metadata
project:
  name: VaultFlower
  type: PAM (Privileged Access Management)
  domain: cybersecurity
  subdomain: credential-lifecycle-management
  target_environments: [IT, ICS, OT, air-gapped, offline]
  security_standards: [NIST-SP-800-53, FSTEC, IEC-62443, NERC-CIP]
  license: Apache-2.0
  language: en
  stack: [dotnet9, postgresql17, rabbitmq4, hashicorp-vault, blazor-server]
```

---

## 🎯 The Problem VaultFlower Solves

Most enterprise PAM solutions handle domain-joined machines well. But critical infrastructure has a different reality:

```
❌ Air-gapped OT networks that cannot join a domain
❌ Legacy SCADA systems with no network connectivity
❌ Industrial controllers (PLC, RTU) with local accounts only
❌ "Break glass" accounts where nobody knows the password
❌ Manual password rotation with no audit trail
❌ No structured process for offline asset credential management
```

**VaultFlower fills this gap** — providing full credential lifecycle management for assets that classical PAM cannot reach, with security architecture designed to meet the highest classification standards.

---

## ✨ Key Features

### Security Architecture
- **Data Fragmentation (B++ level)** — credentials, assets, and identities stored in three physically separated databases. No single database contains a complete picture.
- **Zero-persistence** — plaintext passwords assembled in memory only, shown once, never written to disk or logs.
- **Dual Control** — no single operator can retrieve a credential. Two independent MFA authorizations always required.
- **Shamir Secret Sharing** — Vault master key split across 5 administrators, requires 3 to unseal.
- **Immutable Audit Log** — append-only PostgreSQL schema, no UPDATE/DELETE ever granted.

### Credential Lifecycle
- ✅ Encrypted password vault (AES-256-GCM + envelope encryption)
- ✅ Check-out / check-in workflow with mandatory justification and TTL
- ✅ Automatic password rotation (online via WinRM/SSH, domain via LDAP/ADSI)
- ✅ Offline/air-gapped rotation via structured Maintenance Tasks
- ✅ Password history validation (configurable, default 10 passwords)
- ✅ Owner-approved workflow with SSO-gated approval links

### Compliance & Audit
- ✅ SIEM integration via Syslog CEF (RFC 5424) for every lifecycle event
- ✅ Compliance reporting (NIST SP 800-53, FSTEC, IEC 62443)
- ✅ Signed physical form upload for offline task completion
- ✅ Per-fragment audit logging across all three databases
- ✅ Out-of-hours access detection and alerting

### Enterprise Features
- ✅ Multi-tenant architecture with full isolation
- ✅ Location → System → Zone → Asset hierarchy (ISA/IEC 62443)
- ✅ Criticality levels per system (IEC 62443 Security Levels)
- ✅ Plugin-based MFA (TOTP, WebAuthn, Mifare Smartcard)
- ✅ Domain account rotation (AD/LDAP with owner notification)
- ✅ OpenTelemetry observability with PII masking

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        vfw-core                             │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌────────────────────────┐   │
│  │ Blazor   │  │ .NET 9   │  │   Worker               │   │
│  │ Server   │  │ Minimal  │  │   Workflow Engine       │   │
│  │ Portal   │  │ API      │  │                        │   │
│  └──────────┘  └──────────┘  └────────────────────────┘   │
│        │              │                    │                │
│        └──────────────┴────────────────────┘               │
│                             │                               │
│                      RabbitMQ (mTLS)                        │
│                             │                               │
│  ┌──────────┐  ┌──────────┐ │ ┌──────────┐  ┌─────────┐  │
│  │Postgres  │  │Postgres  │ │ │Postgres  │  │HashiCorp│  │
│  │Assets DB │  │Secrets DB│ │ │Identity  │  │Vault    │  │
│  │          │  │          │ │ │DB        │  │(KMS)    │  │
│  └──────────┘  └──────────┘ │ └──────────┘  └─────────┘  │
│                             │                               │
└─────────────────────────────│───────────────────────────────┘
                              │ mTLS
                      ┌───────┴──────┐
                      │   APP01      │
                      │  Rotation    │
                      │  Agent       │
                      │  (WinRM/SSH/ │
                      │   LDAP)      │
                      └──────────────┘
```

**Core principle:** No single component ever has access to the complete data picture without explicit authorization through HashiCorp Vault as an independent trusted intermediary.

For full architecture documentation → [`/docs/architecture/`](docs/architecture/)

---

## 🔐 Security Standards Compliance

| Standard | Coverage | Documentation |
|---|---|---|
| **NIST SP 800-53 Rev 5** | AC-2, AC-3, AC-17, AU-2, AU-9, IA-2, IA-5, SC-28 | [`/docs/security/nist-mapping.md`](docs/security/nist-mapping.md) |
| **FSTEC (Russia)** | ОПС.1, ОПС.2, ИАФ.1, ИАФ.6, УПД.1, РСБ.1 | [`/docs/security/fstec-mapping.md`](docs/security/fstec-mapping.md) |
| **ISA/IEC 62443** | Zones & Conduits, Security Levels SL1-SL4 | [`/docs/security/iec62443-mapping.md`](docs/security/iec62443-mapping.md) |
| **NERC CIP** | CIP-004, CIP-007, CIP-010 | [`/docs/security/nerc-cip-mapping.md`](docs/security/nerc-cip-mapping.md) |

---

## 🛠️ Technology Stack

| Component | Technology | Version |
|---|---|---|
| Backend API | ASP.NET Core Minimal API | .NET 9 |
| Admin Portal | Blazor Server + SignalR | .NET 9 |
| Workflow Engine | .NET Worker Services | .NET 9 |
| Rotation Agent | .NET Windows Service + gMSA | .NET 9 |
| Message Bus | RabbitMQ (mTLS) | 4.0 |
| Primary Database | PostgreSQL | 17 |
| Secrets / KMS | HashiCorp Vault OSS | Latest |
| Service Discovery | HashiCorp Consul | Latest |
| Object Storage | MinIO | Latest |
| Observability | OpenTelemetry → Jaeger + VictoriaMetrics + Grafana | Latest |
| Authentication | Kerberos SSO + Password + MFA Plugins | — |
| SIEM | Syslog RFC 5424, CEF format | — |
| Reverse Proxy | nginx | Alpine |

---

## 🗂️ Repository Structure

```
VaultFlower/
├── .github/
│   └── workflows/
│       └── build.yml              # CI pipeline
├── docs/
│   ├── adr/                       # Architecture Decision Records
│   ├── api/                       # API contract documentation
│   ├── architecture/              # System architecture diagrams
│   ├── schema/                    # Database schema documentation
│   ├── security/                  # Security standards mapping
│   ├── ui/                        # UI flows and wireframes
│   └── ai/                        # AI-friendly metadata and system prompt
├── src/
│   ├── VaultFlower.Api/           # .NET 9 Minimal API
│   ├── VaultFlower.Portal/        # Blazor Server Admin UI
│   ├── VaultFlower.Worker.Workflow/  # Workflow Engine
│   ├── VaultFlower.Worker.Rotation/  # Rotation Agent (Windows)
│   ├── VaultFlower.Agent.Workstation/ # Workstation Agent (Smartcard)
│   ├── VaultFlower.Core/          # Domain models, encryption logic
│   ├── VaultFlower.Contracts/     # RabbitMQ message contracts
│   └── VaultFlower.Infrastructure/ # DB, Vault, Syslog, MinIO
├── plugins/
│   ├── VaultFlower.Plugin.Mfa.Totp/
│   ├── VaultFlower.Plugin.Mfa.WebAuthn/
│   └── VaultFlower.Plugin.Mfa.Smartcard/
├── deploy/
│   ├── docker/                    # Docker Compose files
│   ├── vault/                     # HashiCorp Vault HCL policies
│   ├── consul/                    # Consul configuration
│   ├── rabbitmq/                  # RabbitMQ definitions
│   └── nginx/                     # nginx configuration
└── tests/
    ├── unit/
    ├── integration/
    └── security/
```

---

## 🚀 Quick Start

> ⚠️ **Prerequisites:** Active Directory domain, PKI/CA, Docker, .NET 9 SDK

### 1. Clone the repository

```bash
git clone https://github.com/maxzaikin/VaultFlower.git
cd VaultFlower
```

### 2. Configure environment

```bash
cp deploy/docker/.env.example deploy/docker/.env
# Edit .env with your AD domain, PKI endpoints, and network settings
```

### 3. Start infrastructure

```bash
docker compose -f deploy/docker/docker-compose.infra.yml up -d
```

### 4. Initialize Vault (Shamir unseal)

```bash
# Follow the Vault initialization guide
# docs/operations/vault-init.md
```

### 5. Start application services

```bash
docker compose -f deploy/docker/docker-compose.app.yml up -d
```

Full deployment guide → [`/docs/operations/deployment.md`](docs/operations/deployment.md)

---

## 🌸 Open Core Model

VaultFlower follows the **Open Core** business model:

| Feature | Community | Enterprise |
|---|---|---|
| Core PAM engine | ✅ Free | ✅ Free |
| Password MFA | ✅ Free | ✅ Free |
| WinRM/SSH rotation | ✅ Free | ✅ Free |
| Single tenant | ✅ Free | ✅ Free |
| Community support | ✅ Free | ✅ Free |
| TOTP MFA plugin | — | 💰 Licensed |
| WebAuthn plugin | — | 💰 Licensed |
| Smartcard (Mifare) plugin | — | 💰 Licensed |
| Multi-tenant | — | 💰 Licensed |
| LDAP/AD rotation | — | 💰 Licensed |
| Compliance reports | — | 💰 Licensed |
| Custom rotation plugins | — | 💰 On request |
| Priority support | — | 💰 On request |

**Enterprise inquiries** → [GitHub Issues](https://github.com/maxzaikin/VaultFlower/issues) with label `enterprise`

---

## 🤝 Contributing

VaultFlower welcomes contributions from the security and DevOps community.

Before contributing, please read:
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — contribution guidelines
- [`docs/architecture/decisions.md`](docs/architecture/decisions.md) — architectural constraints
- [`docs/security/principles.md`](docs/security/principles.md) — non-negotiable security rules

**Good first issues** → [GitHub Issues](https://github.com/maxzaikin/VaultFlower/issues?q=label%3A%22good+first+issue%22)

---

## 📖 Documentation

Full documentation is available on the [project Wiki](https://github.com/maxzaikin/VaultFlower/wiki).

| Section | Description |
|---|---|
| [Architecture](https://github.com/maxzaikin/VaultFlower/wiki/Architecture) | System design, components, data flow |
| [Security Model](https://github.com/maxzaikin/VaultFlower/wiki/Security-Model) | Encryption, MFA, audit, compliance |
| [Database Schema](https://github.com/maxzaikin/VaultFlower/wiki/Database-Schema) | All three databases documented |
| [API Reference](https://github.com/maxzaikin/VaultFlower/wiki/API-Reference) | Full REST API documentation |
| [Deployment Guide](https://github.com/maxzaikin/VaultFlower/wiki/Deployment) | Infrastructure setup and configuration |
| [Operations](https://github.com/maxzaikin/VaultFlower/wiki/Operations) | Vault unseal, backup, monitoring |

---

## 📜 License

VaultFlower is licensed under the **Apache License 2.0**.

See [`LICENSE`](LICENSE) for the full license text.

---

## 👤 Author

**Maxim Zaikin** — Information Security Professional

Building VaultFlower because existing PAM solutions don't satisfy my ambition when it comes to ICS/OT environments and air-gapped assets.

- GitHub: [@maxzaikin](https://github.com/maxzaikin)
- LinkedIn: [Maxim Zaikin](https://linkedin.com/in/maxzaikin)

---

## ⭐ Star History

If VaultFlower solves a problem you've faced in production, please consider giving it a star. It helps the project reach more security professionals who need it.

---

*VaultFlower — Because every privileged account deserves a lifecycle.*
