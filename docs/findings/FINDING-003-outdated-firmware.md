# FINDING-003 — Outdated BIOS firmware on production homelab host

| Field | Value |
|---|---|
| **ID** | FINDING-003 |
| **Title** | M720q BIOS firmware 6+ years out of date |
| **Discovered** | May 16, 2026 |
| **Resolved** | Pending — planned Week 2 |
| **Status** | 🟡 Open |
| **Severity** | Medium |
| **Asset** | Lenovo ThinkCentre M720q (Serial: MJ082ZQS) |
| **Reporter** | Cecil Sparks |

---

## Summary

The Lenovo ThinkCentre M720q in service as the homelab hypervisor is running BIOS firmware **M1UKT45A**, dated **July 11, 2019**. As of the discovery date, this firmware is approximately **6 years 10 months out of date**. Lenovo has released multiple BIOS updates for this platform in the intervening period, including security advisories addressing CVEs in Intel Management Engine, processor microcode (Spectre/Meltdown variants, Downfall, Inception), and BIOS-level vulnerabilities.

The system is functional and does not show evidence of compromise, but operating outdated firmware on a hypervisor — the most privileged software on the host — represents a meaningful and unnecessary risk.

---

## Impact

**Realized impact:**
- Likely contributing factor to [FINDING-002](FINDING-002-boot-media-compatibility.md) (Ventoy USB enumeration failure)
- Unknown additional firmware-level bugs that may surface during continued use

**Potential impact:**
- **Unpatched CVEs.** Multiple CVEs affecting Intel CPUs and platform firmware have been disclosed since July 2019 (e.g., Downfall CVE-2022-40982, Inception CVE-2023-20569, various Intel ME advisories). Mitigations for many of these require updated microcode delivered via BIOS update.
- **Reduced hardware compatibility.** Newer USB devices, NVMe drives, and OS installers may not work as expected on outdated firmware.
- **Audit finding risk.** In a regulated environment (PCI DSS, HIPAA, SOC 2, ISO 27001), running multi-year-old firmware on a privileged host would be flagged.
- **Reduced effectiveness of OS-level controls.** Some OS-level security features (e.g., virtualization-based security, certain Spectre/Meltdown mitigations) depend on up-to-date microcode.

---

## Root cause

The system was acquired secondhand and put into service without a firmware update step in the provisioning runbook. The current Day 1 install runbook does **not** include a BIOS update gate before OS install.

This is itself a process gap — a "secure baseline" should include firmware as well as OS configuration.

---

## Planned remediation

**Target:** Week 2 of the homelab build.

### Pre-update preparation

1. **Identify latest available BIOS** for M720q on Lenovo's support site
2. **Download the BIOS update package** directly from `support.lenovo.com` (verify HTTPS and vendor authenticity)
3. **Read the release notes** for every version between M1UKT45A and the latest, capturing:
   - Security fixes (CVE references)
   - Compatibility fixes
   - Any "do not skip" intermediate versions (some vendors require sequential updates)
4. **Verify the SHA-256 checksum** of the downloaded BIOS update package against Lenovo's published value
5. **Document the planned rollback procedure** before starting (older BIOS image saved, recovery procedure understood)
6. **Snapshot Proxmox** before reboot — even though BIOS updates don't touch the OS disk, capture state in case the host fails to POST after update
7. **Schedule a maintenance window** during which the homelab can be down for at least 2 hours
8. **Connect the host to UPS or stable AC power** — a power loss during BIOS flash can permanently brick the system

### Update execution

1. **Boot Proxmox normally** and verify everything is healthy
2. **Shut down all VMs** (none running yet at Week 2, but procedure documented for future)
3. **Power off Proxmox cleanly** (`shutdown -h now`)
4. **Boot to BIOS** (F1)
5. **Use the in-BIOS update mechanism** if supported, or a USB-based update if not
6. **Allow the update to complete uninterrupted** (typically 5–10 minutes)
7. **Do not power off the system during the flash** under any circumstance

### Post-update verification

1. **System POSTs successfully** and shows the new BIOS version on the splash screen
2. **Re-enter BIOS Setup** and verify the [BIOS Hardening Baseline v1.0](../baselines/bios-hardening-baseline.md) is still applied
   - Some BIOS updates reset settings to defaults — re-apply baseline if needed
3. **Boot Proxmox** and verify all hardware is detected (`lspci`, `lsblk`, `ip addr`)
4. **Check `dmesg`** for new microcode messages confirming updated CPU microcode is loaded
5. **Re-test Ventoy USB** to determine whether FINDING-002 was related to firmware
6. **Update this finding** with status: Resolved, including the new BIOS version and date

---

## Compensating controls — until remediated

While this finding is open, the following controls reduce risk:

1. **Host is not exposed to the internet** — only accessible from the local network
2. **No production workloads** — system is for learning only; no sensitive data resides on it
3. **Strong BIOS Administrator password** in place ([FINDING-001](FINDING-001-credential-management.md) remediated)
4. **Physical access controls** in place (system is in a private residence)
5. **Hypervisor will be patched immediately** post-install (apt update + full-upgrade scheduled for Day 2)

---

## Framework mappings

### NIST Cybersecurity Framework 2.0

| Control | Description | How this finding relates |
|---|---|---|
| **ID.RA-1** | Asset vulnerabilities identified and documented | Outdated firmware identified |
| **PR.IP-3** | Configuration change control processes | Update procedure and rollback plan documented |
| **PR.IP-12** | A vulnerability management plan is developed and implemented | Remediation scheduled with verification steps |
| **DE.CM-8** | Vulnerability scans are performed | Manual firmware version check qualifies; automation is a future goal |

### CIS Critical Security Controls v8

| Control | Description | How this finding relates |
|---|---|---|
| **2.1** | Inventory and control of software assets | Firmware version is part of software inventory |
| **7.3** | Perform automated operating system patch management | Manually scheduled here; procedure documented for future automation |
| **7.4** | Perform automated application patch management (firmware analog) | Plan to track BIOS releases and apply on a schedule |

### ISO/IEC 27001:2022 Annex A

| Control | Description | How this finding relates |
|---|---|---|
| **A.8.8** | Management of technical vulnerabilities | Direct mapping — firmware vulnerabilities identified and remediation planned |
| **A.8.9** | Configuration management | Firmware version tracked as part of configuration |
| **A.8.32** | Change management | Update will follow documented change process |

---

## Decision log

| Date | Decision | Rationale |
|---|---|---|
| 2026-05-16 | Proceed with OS install on outdated BIOS | Risk accepted for short window; remediation scheduled Week 2; system is isolated and non-production |
| 2026-05-16 | Schedule BIOS update for Week 2 | Allows Day 1 build momentum to continue; UPS available for safe flash; rollback understood |

---

## Lessons learned

1. **Firmware is in scope.** A "secure baseline" that doesn't include firmware updates is incomplete.
2. **Acquisition matters.** Secondhand hardware comes with whatever firmware the previous owner ran. Future hardware acquisitions should include "verify firmware level and update if behind" as the first provisioning step.
3. **Document the risk acceptance.** The decision to proceed despite outdated firmware is a conscious risk acceptance. Documenting it (here, in the decision log) means I'm not pretending the risk doesn't exist — I'm acknowledging it and committing to a remediation date. That's what a real risk register looks like.
4. **One finding can predict another.** The presence of an old BIOS made FINDING-002 (Ventoy enumeration) more plausible. Mature security teams use this kind of pattern matching to triage.
