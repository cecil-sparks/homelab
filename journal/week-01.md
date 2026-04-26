# Week 1 — Pre-build Prep & Baseline Planning

**Dates:** April 25 – May 3, 2026
**Status:** 🟡 Hardware in transit
**Security+ progress:** Domain 1 — General Security Concepts (in progress)

---

## 🎯 Goals This Week

- [x] Research hardware (with vendor risk considerations)
- [x] Order Lenovo M720q + bootable USB
- [x] Prepare bootable Proxmox installer
- [ ] Begin Security+ Domain 1 study (CIA triad, control types, AAA)
- [ ] Draft preliminary asset inventory
- [ ] Watch one Proxmox install walkthrough
- [ ] Plan IP addressing and segmentation strategy

---

## 📅 Daily Log

### Saturday, April 25

#### Hardware decisions — through a GRC lens
Spent the day evaluating mini PC options. From a GRC perspective, the questions I asked weren't just "which is fastest?" but:

| Question | Why it matters in GRC |
|---|---|
| What's the vendor's update/firmware track record? | Patch management is a foundational control |
| Does the BIOS support modern security features (TPM, Secure Boot)? | Maps to CIS Control 4 — secure configuration |
| Can I disable unused interfaces and ports? | Attack surface reduction |
| Is community documentation strong? | Enables consistent, repeatable secure configuration |

**Compared:**
- **Lenovo M720q** (chosen) — strong enterprise lineage, broad community support, no NIC whitelist (lets me apply controls without vendor lock-in)
- **Dell OptiPlex 7060 Micro** — also acceptable, slightly older
- **HP EliteDesk 800 G5 Mini** — newer hardware but BIOS NIC whitelist limits my ability to apply controls; this is an example of vendor decisions creating compliance friction

#### Created the installer
- Installed Ventoy 1.1.12 on a SanDisk Ultra Flair 128GB USB 3.0
- Loaded Proxmox VE 9.1 ISO onto the drive
- Ventoy as a tool: useful for legitimate admins, but also a useful example of why **removable media** is a controlled item in most security frameworks (CIS Control 10, NIST SP 800-53 MP family). I'll document USB usage in my homelab acceptable-use policy.

#### Initial risk thinking
Before anything is even installed, I started a preliminary risk register:

| Asset | Risk | Likelihood | Impact | Initial Treatment |
|---|---|---|---|---|
| Lenovo M720q | Physical theft | Low | Medium | Locate in restricted area; full-disk encryption |
| Proxmox host | Unauthorized admin access | Medium | High | Strong password, MFA, SSH key auth |
| VM data (photos, media) | Data loss from drive failure | Medium | Medium | Snapshots + offsite backup (Phase 4) |
| Network exposure | Internet-facing services exploited | Medium | High | No port forwarding; Tailscale-only access |

Will refine this in `/controls/risk-register.md` once the system is live.

#### Security+ study
- Began Professor Messer's SY0-701 series, episode 1
- Reviewed the **CIA triad** and **control types** (preventive, detective, corrective, compensating)
- Already useful — when planning network segmentation later this week, I'm thinking about it in those terms instead of just "how do I set up a VLAN?"

---

## 🛒 Shopping Recap

| Item | Vendor | Price | Status |
|------|--------|-------|--------|
| Lenovo ThinkCentre M720q (Refurbished) | Amazon Renewed | $250.00 | Ordered, ETA May 4 |
| SanDisk Ultra Flair 128GB | Amazon | $25.99 | ✅ Received |
| **Total spent** | | **$275.99** | |

---

## 💡 Key Decisions (and the GRC Reasoning)

### Hypervisor: Proxmox over bare-metal Linux
**Why (technical):** Snapshots, VM isolation, web UI.
**Why (GRC):** Snapshots = recovery controls. VM isolation = segmentation. Auditable VM lifecycle (create, modify, delete, snapshot) is closer to how enterprise infrastructure is governed than running services directly on the OS.

### Refurbished hardware over new
**Why (financial):** Saves ~$150.
**Why (GRC):** Forces me to confront supply chain risk in a real way. I had to think about: vendor reputation (Amazon Renewed warranty), tamper-evidence (factory-resealed packaging), pre-installed software trust (I'm wiping it anyway). All of these come up in real third-party risk assessments.

### Tailscale over open port forwarding
**Why (technical):** Easier, no router config.
**Why (GRC):** Maps cleanly to **NIST CSF PR.AC-3** (remote access is managed) and **CIS Control 12.7** (use of secure VPN/zero-trust connectivity). Reduces external attack surface to zero — a control I can document and demonstrate.

---

## 🔜 Next Week

When the M720q arrives:
1. Document **physical setup** as evidence of asset deployment
2. **BIOS configuration** — enable TPM, Secure Boot, virtualization extensions; document each setting
3. **Install Proxmox VE 9.1** with full-disk encryption (recovery control + confidentiality)
4. **Initial admin account hygiene** — strong password, no shared accounts (CIS Control 5, 6)
5. **First snapshot** — my first documented recovery point
6. **Update the risk register** with anything I learned during install
7. **Start `/controls/controls-mapping.md`** with my first 5–10 mapped controls

---

## 📸 Evidence Folder

*(Will populate `/controls/evidence/` with install screenshots, BIOS settings, and config exports starting next week. Treating these as audit artifacts from day one.)*

---

## 🧠 Reflection

A homelab built without security thinking is just a server farm waiting to be embarrassing. A homelab built *with* security thinking is a portfolio piece. I'd rather move slowly and document well than move fast and have nothing to show an auditor — or a hiring manager.

— Cecil
