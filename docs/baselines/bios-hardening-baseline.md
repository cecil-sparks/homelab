# BIOS Hardening Baseline — Lenovo ThinkCentre M720q

| Field | Value |
|---|---|
| **Document** | BIOS Hardening Baseline |
| **Version** | 1.0 |
| **Effective date** | May 16, 2026 |
| **Asset class** | Lenovo ThinkCentre M720q Tiny |
| **BIOS version at baseline** | M1UKT45A (07/11/2019) |
| **Author** | Cecil Sparks |
| **Review cadence** | Quarterly, or after BIOS firmware update |

---

## Purpose

Define the secure default configuration for the BIOS firmware of any Lenovo M720q in the homelab fleet. Any deviation from this baseline must be:

1. Documented as an **exception** with a stated reason
2. Tracked with a **planned remediation date**
3. Reviewed at every cadence cycle until the deviation is closed

This baseline addresses physical-layer and firmware-layer controls that the operating system cannot enforce on its own.

---

## Configuration matrix

### Tab: Devices

| Setting | Baseline value | Notes |
|---|---|---|
| USB Setup → USB Port Access | Enabled (all ports) | Disable on production hosts in untrusted areas |
| USB Setup → USB Legacy | Enabled | Required for some boot installers |
| USB Setup → USB Boot | Enabled | Required for OS install / recovery |
| Network Setup → PXE IPv4 Network Stack | **Disabled** | Reduces attack surface; not used here |
| Network Setup → PXE IPv6 Network Stack | **Disabled** | Same as above |
| Network Setup → Wake on LAN | Disabled | Re-enable later if remote wake is needed |
| ATA Drive Setup → SATA Controller | Enabled | NVMe is the boot device, but SATA controller stays standard |
| ATA Drive Setup → Configure SATA as | AHCI | Modern OSes; never RAID/IDE for this workload |
| ATA Drive Setup → Hard Disk Pre-delay | 0 sec | No spinning rust to wait for |

### Tab: Advanced

| Setting | Baseline value | Notes |
|---|---|---|
| CPU Setup → Intel Hyper-Threading | Enabled | Performance; mitigations applied at OS layer |
| CPU Setup → Intel Virtualization Technology (VT-x) | **Enabled** | Required for hypervisor (Proxmox) |
| CPU Setup → Intel VT-d | **Enabled** | Required for PCIe passthrough |
| CPU Setup → C-States | Enabled | Power savings; harmless for this workload |
| CPU Setup → Intel SpeedStep | Enabled | Power savings |
| CPU Setup → Intel Turbo Boost | Enabled | Performance |
| Intel Manageability → Intel AMT | Disabled | Reduces attack surface; not used in homelab |
| Intel Manageability → Intel ME Setup | Default | Cannot fully disable on consumer firmware |

### Tab: Power

| Setting | Baseline value | Notes |
|---|---|---|
| After Power Loss | Power On | Hypervisor should auto-recover after outage |
| Automatic Power On → Wake from S4/S5 | Disabled | Re-enable selectively if needed |
| Automatic Power On → Wake on Alarm | Disabled | Not used |

### Tab: Security

| Setting | Baseline value | Notes |
|---|---|---|
| Administrator Password | **Set** (strong, in Bitwarden + paper backup) | See [FINDING-001](../findings/FINDING-001-credential-management.md) |
| Power-On Password | Optional — not set | Adds friction for legitimate access; consider for travel laptops |
| System Management Password | Not set | Not differentiated from admin in homelab |
| Hard Disk Password | Not set | NVMe self-encrypting drives prefer OS-layer encryption |
| Password Policy → Maximum Password Length | 64 | Use the maximum allowed |
| TPM Setup → Security Chip | **Enabled (Active)** | Required for OS-layer features |
| TPM Setup → Clear Security Chip | Not invoked | Only invoke when retiring or wiping |
| Secure Boot | **Disabled (exception)** | See exception below |
| BIOS Flashback Protection | **Enabled** | Prevents unauthorized BIOS rollback/reflash |
| Smart USB Protection | **Disabled (exception)** | See exception below |
| Cover Tamper Detected | Enabled | Alerts if chassis is opened |
| Intrusion Alert | Enabled | Pairs with cover tamper |

### Tab: Startup

| Setting | Baseline value | Notes |
|---|---|---|
| CSM Support | **Enabled (exception)** | See exception below |
| Boot Mode | Auto | UEFI preferred; Auto allows fallback during install |
| Boot Priority | UEFI First | UEFI bootloaders take precedence |
| Quick Boot | Disabled | Allows full POST diagnostics on each boot |
| Boot Up NumLock | Default | Cosmetic |
| Option Key Display | Enabled | Shows F-key options at POST — useful for recovery |
| Error Beep | Enabled | Audible POST error indication |

### Tab: Exit

| Setting | Baseline value | Notes |
|---|---|---|
| OS Optimized Defaults | Enabled (Win10/11 64-bit) | Sets sensible defaults for modern OSes |
| Load Default Settings | Not invoked unless rebuilding | Triggers full reset to vendor defaults |

---

## Documented exceptions

Each exception below represents a setting that deviates from the ideal hardened state. Each has a stated reason and a planned remediation.

### Exception 1 — CSM Support enabled

| Field | Value |
|---|---|
| **Setting** | Startup → CSM Support |
| **Baseline (ideal)** | Disabled |
| **Current state** | Enabled |
| **Reason** | Some installer media (and the troubleshooting workflow that resolved [FINDING-002](../findings/FINDING-002-boot-media-compatibility.md)) required CSM to enumerate boot media reliably during install |
| **Risk** | Allows non-UEFI boot path; reduces protection against pre-boot threats |
| **Compensating control** | Boot Priority set to "UEFI First"; Boot Mode set to "Auto" so UEFI is preferred whenever available |
| **Planned remediation** | After Week 2 BIOS firmware update (FINDING-003), retest with CSM disabled |
| **Remediation target** | Week 2, Day 2 |

### Exception 2 — Secure Boot disabled

| Field | Value |
|---|---|
| **Setting** | Security → Secure Boot |
| **Baseline (ideal)** | Enabled |
| **Current state** | Disabled |
| **Reason** | Disabled during install troubleshooting to eliminate variables; not yet re-enabled |
| **Risk** | Allows boot of unsigned bootloaders/kernels; reduces protection against rootkits |
| **Compensating control** | BIOS Administrator password set; physical access controls in place |
| **Planned remediation** | Week 2 — verify Proxmox + GRUB chain works with Secure Boot enabled, then enable |
| **Remediation target** | Week 2, Day 3 |

### Exception 3 — Smart USB Protection disabled

| Field | Value |
|---|---|
| **Setting** | Security → Smart USB Protection |
| **Baseline (ideal)** | Read Only (or Disable USB Storage) on production hosts |
| **Current state** | Disabled |
| **Reason** | Required during boot media troubleshooting (FINDING-002) |
| **Risk** | Allows arbitrary USB storage access; data exfiltration vector for attackers with physical access |
| **Compensating control** | Physical access controls; cover tamper detection enabled |
| **Planned remediation** | Set to "Read Only" after Week 2 build is stable |
| **Remediation target** | Week 2, Day 5 |

---

## Verification procedure

To verify a system meets this baseline:

1. Boot system and enter BIOS Setup (F1)
2. Authenticate as BIOS Administrator
3. Walk every tab and compare each setting to the matrix above
4. For each deviation, confirm it matches a documented exception in this baseline
5. If a deviation is found that is **not** documented as an exception, raise a finding
6. Sign and date the verification (initials + date in BIOS administrator log)

---

## Framework mappings

### NIST Cybersecurity Framework 2.0

| Control | Description | How baseline relates |
|---|---|---|
| **PR.AC-1** | Identities and credentials are issued, managed, verified, revoked, and audited | BIOS administrator password set and managed |
| **PR.AC-3** | Remote access is managed | Wake-on-LAN and PXE disabled |
| **PR.AC-5** | Network integrity is protected | PXE stacks disabled; no firmware-layer network exposure |
| **PR.DS-1** | Data-at-rest is protected | TPM enabled to support OS-layer encryption |
| **PR.IP-1** | Baseline configuration is created and maintained | This document |
| **PR.PT-3** | Principle of least functionality | Unused features (PXE, AMT, etc.) disabled |

### CIS Critical Security Controls v8

| Control | Description | How baseline relates |
|---|---|---|
| **4.1** | Establish and maintain a secure configuration process | This document |
| **4.6** | Securely manage enterprise assets and software | Firmware-layer hardening |
| **4.8** | Uninstall or disable unnecessary services on enterprise assets | PXE, AMT disabled |
| **5.4** | Restrict administrator privileges to dedicated administrator accounts | BIOS administrator password gates firmware admin |
| **12.1** | Ensure network infrastructure is up-to-date | Aspirational here; FINDING-003 tracks the firmware gap |

### ISO/IEC 27001:2022 Annex A

| Control | Description | How baseline relates |
|---|---|---|
| **A.5.17** | Authentication information | BIOS administrator credential managed |
| **A.8.5** | Secure authentication | Strong BIOS password; TPM enabled |
| **A.8.9** | Configuration management | This baseline document |
| **A.8.20** | Networks security | PXE/network stacks disabled |
| **A.8.24** | Use of cryptography | TPM enabled to support OS-layer crypto |

---

## Change log

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-05-16 | Initial baseline established post-CMOS reset and Day 1 install |
