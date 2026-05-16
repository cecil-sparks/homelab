# 🛡️ Homelab — A Hands-On GRC Learning Lab

> Building a personal homelab and treating it like a small business: assessing risks, applying controls, and documenting evidence — the way a GRC analyst would.

**Status:** 🟢 **Day 1 Complete — Proxmox VE 9.1 hypervisor operational**
**Last updated:** May 16, 2026

---

## Why this repo exists

I'm transitioning from 13+ years in law enforcement (Classification Specialist) into a career in **Governance, Risk, and Compliance (GRC)**. This homelab is my way of learning the technical foundations that GRC analysts need to evaluate — virtualization, network segmentation, identity management, logging, patching, and backup — by **building and securing it myself**.

Every change in this repo is documented like an audit artifact:

- **What** was changed
- **Why** it was changed (the risk it addresses)
- **Which framework control** it maps to (NIST CSF 2.0, CIS Controls v8, ISO 27001:2022)
- **Evidence** of the change (commits, configs, screenshots)

The goal: by the time I finish my Master's in Information Systems at Tarleton State, I'll have a portfolio that shows hiring managers I can **think like an auditor** and **speak the language of security operations**.

---

## Current architecture

**Physical host:** Lenovo ThinkCentre M720q Tiny
- CPU: Intel i5-8400T (6 cores, VT-x/VT-d enabled)
- RAM: 16 GB DDR4
- Storage: WDC PC SN520 500 GB NVMe SSD
- Network: 1 GbE onboard

**Hypervisor:** Proxmox VE 9.1 (Debian 13 Trixie base)
- Host: `proxmox.lab.local`
- Management IP: `192.168.1.10/24` (static)
- Web UI: `https://192.168.1.10:8006`
- Filesystem: ext4 on local NVMe

**Network:** AT&T Fiber gateway at `192.168.1.254`

---

## What's in this repo

| Folder | What you'll find |
|---|---|
| [`docs/diagrams/`](docs/diagrams/) | Visual architecture and topology diagrams |
| [`docs/findings/`](docs/findings/) | Issues discovered during build, with framework mappings and remediation |
| [`docs/baselines/`](docs/baselines/) | Hardening baselines (BIOS, hypervisor, OS) |
| [`docs/runbooks/`](docs/runbooks/) | Reproducible step-by-step procedures |
| [`journal/`](journal/) | Day-by-day build log — what I did, what broke, what I learned |
| [`tools.md`](tools.md) | Tooling inventory |

---

## Findings tracker

| ID | Title | Severity | Status |
|---|---|---|---|
| [FINDING-001](docs/findings/FINDING-001-credential-management.md) | BIOS Administrator password loss | High | ✅ Resolved |
| [FINDING-002](docs/findings/FINDING-002-boot-media-compatibility.md) | Ventoy USB not enumerated by host BIOS | Medium | ✅ Resolved |
| [FINDING-003](docs/findings/FINDING-003-outdated-firmware.md) | BIOS firmware 6+ years out of date | Medium | 🟡 Open (Week 2) |

---

## Roadmap

### ✅ Week 1 — Foundation
- [x] Hardware provisioning and inventory
- [x] BIOS hardening baseline established
- [x] Proxmox VE 9.1 installed on bare metal
- [x] Static IP assigned, web UI reachable
- [x] Day 1 findings documented

### 🟡 Week 2 — Hardening & Remediation
- [ ] BIOS firmware update (remediate FINDING-003)
- [ ] Switch Proxmox to no-subscription repo
- [ ] Apply all available OS updates
- [ ] Install Tailscale for out-of-band access
- [ ] Re-enable Secure Boot
- [ ] First VM snapshot

### 🔵 Week 3+ — Capability Build-Out
- [ ] First VM: pfSense or OPNsense firewall
- [ ] VLAN segmentation
- [ ] Centralized logging (Wazuh or Graylog)
- [ ] Identity provider (Authentik or Keycloak)
- [ ] Backup and disaster recovery procedures

---

## Framework mapping methodology

Every finding and baseline in this repo includes mappings to:

- **NIST Cybersecurity Framework 2.0** (Identify, Protect, Detect, Respond, Recover, Govern)
- **CIS Critical Security Controls v8** (Implementation Group 1 targets)
- **ISO/IEC 27001:2022** Annex A controls

The mappings aren't theoretical — they show how a real-world configuration decision satisfies (or falls short of) a published control. This is the same exercise GRC analysts perform daily.

---

## About me

I'm Cecil — based in Dallas, TX. Spent 13+ years in law enforcement as a Classification Specialist, Detention Training Officer, and Fiscal Division Specialist. Finishing my B.A.A.S. at Tarleton State University in July 2026, then starting a Master's in Information Systems with a cybersecurity specialization.

You can follow the build in real time on X: [@CueTechPro](https://x.com/CueTechPro)

---

## License

See [LICENSE](LICENSE).
