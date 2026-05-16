# FINDING-001 — BIOS Administrator credential loss

| Field | Value |
|---|---|
| **ID** | FINDING-001 |
| **Title** | BIOS Administrator password lost during initial system setup |
| **Discovered** | May 15, 2026 |
| **Resolved** | May 16, 2026 |
| **Status** | ✅ Resolved |
| **Severity** | High |
| **Asset** | Lenovo ThinkCentre M720q (Serial: MJ082ZQS) |
| **Reporter** | Cecil Sparks |

---

## Summary

During initial system provisioning on April 29, 2026, a BIOS Administrator password was set on the M720q to harden firmware-level access. The password was not recorded in a password manager or written down. When BIOS access was needed approximately two weeks later for troubleshooting USB boot media (see [FINDING-002](FINDING-002-boot-media-compatibility.md)), the password could not be recovered. The system was effectively locked out of firmware administration until a CMOS reset was performed.

---

## Impact

**Realized impact:**
- ~1 hour of troubleshooting time lost
- Physical case opening required (warranty implication negligible — out of warranty, but precedent-setting)
- All BIOS hardening had to be rebuilt from factory defaults

**Potential impact if this had been a production system:**
- Inability to enforce BIOS-level controls (e.g., disable USB boot, restrict firmware updates)
- Required physical access for remediation — not feasible for remote/branch systems
- Potential audit finding for an external compliance assessment
- Single point of failure for firmware-layer access control

---

## Root cause

**Procedural failure, not technical.**

The password was generated and entered into the BIOS during a one-off setup session. No procedure existed to:

1. Generate the credential in a password manager first, then copy it to the BIOS
2. Verify the credential was retrievable before walking away from the system
3. Tag the credential with the specific asset it protected
4. Establish an offline/paper backup for tier-1 credentials (those that gate physical recovery)

---

## Remediation performed

1. **Power off** the M720q and disconnect from AC power
2. **Open the case** — single thumbscrew on rear, slide top panel rearward
3. **Locate the CR2032 coin battery** on the motherboard near the front of the chassis
4. **Disconnect the battery** for **5 minutes** (longer than the 30 s minimum, to ensure capacitor discharge)
5. **Reconnect** the battery, reassemble the case
6. **Power on** — BIOS settings returned to factory defaults; admin password cleared
7. **Re-enter BIOS Setup** and apply the [BIOS Hardening Baseline v1.0](../baselines/bios-hardening-baseline.md)
8. **Generate a new BIOS Administrator password** in Bitwarden first
9. **Copy the password into the BIOS** from the password manager
10. **Verify** by exiting and re-entering BIOS with the new credential
11. **Create a paper backup** of the credential, sealed in tamper-evident envelope, stored in home safe

---

## Verification

- ✅ New BIOS Administrator password successfully unlocks BIOS Setup
- ✅ Credential recorded in Bitwarden vault under entry: *"M720q BIOS Administrator Password — Reset May 16 2026"*
- ✅ Paper backup created and stored
- ✅ BIOS hardening baseline re-applied (see baseline doc)

---

## Preventive controls — going forward

A new procedure is now standard for **every** credential set in this homelab:

1. **Generate first, set second.** Generate the credential in Bitwarden, then copy/paste (or carefully transcribe) into the system being configured.
2. **Verify retrievability.** Log out of the password manager and log back in to confirm the entry exists and is readable before considering the change complete.
3. **Tag with asset.** Every credential entry includes the asset name, serial number where applicable, and date set.
4. **Tier-1 credentials get paper backup.** Any credential that gates physical recovery (BIOS, BMC/iDRAC/iLO, recovery seeds, full-disk encryption recovery keys) is also written on paper and stored offline.
5. **Document the procedure.** This finding is itself the documentation that the procedure now exists.

---

## Framework mappings

### NIST Cybersecurity Framework 2.0

| Control | Description | How this finding relates |
|---|---|---|
| **PR.AC-1** | Identities and credentials are issued, managed, verified, revoked, and audited | Failure to manage the issued credential through its lifecycle |
| **PR.AC-3** | Remote access is managed | Indirectly — loss of firmware credential constrains how the asset can be remotely managed |
| **PR.IP-1** | Baseline configuration is created and maintained | The recovery exercise rebuilt the baseline; baseline doc now exists |
| **RC.RP-1** | Recovery plan is executed during or after a cybersecurity incident | CMOS reset procedure executed and documented |

### CIS Critical Security Controls v8

| Control | Description | How this finding relates |
|---|---|---|
| **5.2** | Use unique passwords | Password was unique but not retrievable |
| **5.4** | Restrict administrator privileges to dedicated administrator accounts | Administrator credential lost compromised the principle |
| **6.1** | Establish an access granting process | New procedure formalizes credential creation |

### ISO/IEC 27001:2022 Annex A

| Control | Description | How this finding relates |
|---|---|---|
| **A.5.17** | Authentication information | Direct mapping — handling of authentication secrets |
| **A.8.5** | Secure authentication | The credential itself was secure; the handling was not |

---

## Lessons learned

1. **The boring control bit first.** I was focused on BIOS hardening — UEFI vs. CSM, Secure Boot, USB enumeration — and overlooked the password I'd already set. The unglamorous procedural control (record the credential) was the one that failed.
2. **"I'll remember it" is not a control.** Memory is not an information system. Bitwarden is.
3. **Physical recovery has a cost.** Even a "minor" recovery procedure consumes time, requires physical access, and rebuilds state. In a production environment with hundreds of systems, the cost scales linearly with the number of locked-out assets.
4. **This is exactly the kind of finding a GRC analyst writes.** Procedural failures around credential management are among the most common audit findings in real-world assessments. Living it firsthand makes the abstract concept concrete.
