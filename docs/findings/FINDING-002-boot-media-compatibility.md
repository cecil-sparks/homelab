# FINDING-002 — Boot media not enumerated by host BIOS

| Field | Value |
|---|---|
| **ID** | FINDING-002 |
| **Title** | Ventoy USB boot media not enumerated by Lenovo M720q BIOS |
| **Discovered** | May 15, 2026 |
| **Resolved** | May 16, 2026 |
| **Status** | ✅ Resolved |
| **Severity** | Medium |
| **Asset** | Lenovo ThinkCentre M720q (BIOS M1UKT45A, dated 07/11/2019) |
| **Reporter** | Cecil Sparks |

---

## Summary

A USB drive prepared with [Ventoy](https://www.ventoy.net) — a multi-ISO bootloader — was not detected as a bootable device by the M720q BIOS. The drive did not appear in the boot menu, did not appear in the boot order configuration, and was not enumerated even after extensive BIOS configuration changes (CSM toggling, USB Enumeration Delay adjustments, Smart USB Protection toggling). The same physical USB drive was visible to other systems, confirming the drive itself was functional.

The issue was resolved by re-flashing the USB drive with **Rufus 4.14 in DD-mode**, writing the Proxmox ISO directly to the device byte-for-byte rather than using a multi-ISO bootloader.

---

## Impact

**Realized impact:**
- ~3 hours of troubleshooting time before pivoting approaches
- Build progress blocked until alternative boot media created
- Required research and use of a second tool (Rufus) outside the originally planned toolchain

**Potential impact in a production environment:**
- Bare-metal provisioning blocked
- Field technicians with only Ventoy drives unable to recover/install on this hardware family
- Inconsistent provisioning experience across vendor hardware

---

## Root cause analysis

### What Ventoy is

Ventoy creates a USB with two partitions:
1. A small EFI/BIOS partition with the Ventoy bootloader
2. A larger exFAT partition where the user drops `.iso` files

When the system boots from the USB, Ventoy presents a menu of available ISOs and chains into the selected one without re-flashing.

### Why it didn't work on the M720q

The M720q BIOS (firmware **M1UKT45A**, released **July 11, 2019** — see [FINDING-003](FINDING-003-outdated-firmware.md)) appears to either:

- Not properly enumerate the Ventoy partition layout, or
- Not support the specific ESP/MBR/GPT signature combination Ventoy uses, or
- Have a bug in USB enumeration that has since been patched in newer firmware

Settings tried, none of which resolved the issue:

| BIOS setting tested | Value tried |
|---|---|
| CSM Support | Enabled / Disabled |
| Boot Mode | UEFI / Legacy / Auto |
| USB Enumeration Delay | 0s / 5s / 10s |
| Smart USB Protection | Off / Read Only |
| Secure Boot | Disabled (verified) |
| Re-flashed Ventoy | GPT layout, Secure Boot–compatible build |

The drive was tested in every USB port (front and rear, USB-A 2.0 and 3.x). It enumerated on other systems but never on the M720q.

### Why DD-mode worked

Rufus DD-mode writes the source ISO **byte-for-byte** to the USB device. There is no Ventoy bootloader, no menu, no second partition. The USB becomes a literal copy of the ISO, with whatever partition layout the ISO author intended.

For Proxmox VE, that means a bootable ISOLINUX/GRUB structure that the M720q BIOS recognizes without issue. The trade-off: **one ISO per USB**, and the drive must be re-flashed to install anything else.

---

## Remediation performed

1. **Downloaded Rufus 4.14** from [rufus.ie](https://rufus.ie)
2. **Verified the Proxmox VE 9.1 ISO checksum** against the published SHA-256 on `proxmox.com`
3. **Inserted the USB drive** and launched Rufus
4. **Selected the Proxmox ISO** in the "Boot selection" dropdown
5. When Rufus prompted for write mode, **selected "Write in DD Image mode"** (not ISO mode)
6. **Confirmed** the warning that all data on the USB would be destroyed
7. **Wrote the image** (~7 minutes for a 1.4 GB ISO on USB 3.0)
8. **Booted the M720q with the USB inserted** and pressed F12 for the boot menu
9. **Selected "UEFI: USB Partition 2"** — the installer launched successfully

---

## Verification

- ✅ M720q BIOS enumerated the USB drive in the F12 boot menu after Rufus DD-mode write
- ✅ Proxmox VE 9.1 installer launched and completed successfully
- ✅ ISO checksum verified before writing (no integrity compromise)

---

## Preventive controls — going forward

1. **Validate boot media on target hardware before relying on it.** Don't assume what worked on one system will work on another, especially with multi-ISO tooling.
2. **Maintain at least two boot media tools.** Ventoy is convenient, Rufus DD-mode is reliable. Keep both in the kit.
3. **Verify ISO integrity (checksum) before writing.** Supply chain integrity is a core security control — never trust an ISO without verifying.
4. **Document hardware quirks in this repo.** This finding will save me an hour the next time I install on M720q-class hardware.
5. **Plan to update BIOS firmware** (see FINDING-003) to determine whether the underlying enumeration bug is resolved in newer releases.

---

## Framework mappings

### NIST Cybersecurity Framework 2.0

| Control | Description | How this finding relates |
|---|---|---|
| **ID.RA-1** | Asset vulnerabilities are identified and documented | Hardware/firmware compatibility issue identified |
| **PR.DS-1** | Data-at-rest is protected | Indirectly — ISO checksum verification before writing |
| **PR.IP-3** | Configuration change control processes are in place | Documented attempt and revert of multiple BIOS configurations |

### CIS Critical Security Controls v8

| Control | Description | How this finding relates |
|---|---|---|
| **3.6** | Encrypt data on end-user devices (here: verify integrity of bootable media) | ISO checksum verification before writing |
| **4.1** | Establish and maintain a secure configuration process | BIOS configurations tested and reverted methodically |
| **9.2** | Use DNS filtering services (here: trusted download sources) | Tools (Rufus, Proxmox ISO) downloaded only from official sources |

### ISO/IEC 27001:2022 Annex A

| Control | Description | How this finding relates |
|---|---|---|
| **A.8.9** | Configuration management | BIOS configuration changes tracked and reverted |
| **A.8.20** | Networks security (here: trusted download chain) | Tools sourced from official vendors only |
| **A.8.32** | Change management | Methodical change/revert during troubleshooting |

---

## Lessons learned

1. **Convenience tools have compatibility costs.** Ventoy is brilliant on systems where it works. The byte-for-byte simplicity of DD-mode is a fallback that always works because there's nothing fancy to break.
2. **Don't fight the BIOS forever.** When three hours of BIOS tweaking yield zero progress, change tools, not settings.
3. **Older firmware = older bugs.** This finding led directly to FINDING-003 — recognizing that a 6-year-old BIOS is itself a risk.
4. **Always checksum your ISO.** A corrupted or tampered ISO would have wasted even more time, or worse, installed compromised software. Verifying integrity is cheap insurance.
