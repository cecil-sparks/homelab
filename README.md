# 🛡️ Homelab — A Hands-On GRC Learning Lab

> Building a personal homelab and treating it like a small business: assessing risks, applying controls, and documenting evidence — the way a GRC analyst would.

**Status:** 🟡 Pre-launch — hardware arriving early May 2026
**Career goal:** Aspiring GRC analyst, focused on IT audit & compliance
**Currently studying:** [CompTIA Security+ (SY0-701)](https://www.comptia.org/certifications/security)

---

## 🎯 Why This Repo Exists

Most homelab repos document *how* to build things. This one focuses on **why** — the risk decisions, control choices, and compliance considerations behind every part of the build.

I'm pivoting into Governance, Risk, and Compliance (GRC). To understand what I'll one day audit, I need to first understand how IT systems are actually designed, secured, and operated. This homelab is my classroom.

Every service I deploy is documented with:

- **Risk assessment** — what could go wrong, how likely, how bad
- **Controls applied** — what I did to prevent or detect it
- **Framework mapping** — how those controls align to NIST CSF, CIS Controls v8, and ISO 27001:2022
- **Evidence** — screenshots, configs, logs that prove the control works

---

## 🖥️ The Environment Under Audit

| Component | Model | Specs | Cost |
|-----------|-------|-------|------|
| Server | Lenovo ThinkCentre M720q Tiny | Intel i5-8400T (6 cores), 16GB DDR4, 500GB NVMe | $250 |
| Bootable USB | SanDisk Ultra Flair 128GB USB 3.0 | Proxmox installer | $26 |
| **Total investment** | | | **$276** |

---

## 🧱 Architecture
