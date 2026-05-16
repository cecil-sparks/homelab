# Runbook — Proxmox VE 9.x bare-metal install on Lenovo M720q

| Field | Value |
|---|---|
| **Runbook** | Proxmox VE Installation |
| **Version** | 1.0 |
| **Effective date** | May 16, 2026 |
| **Target hardware** | Lenovo ThinkCentre M720q Tiny |
| **Target software** | Proxmox VE 9.1 (or newer 9.x) |
| **Estimated time** | 60–90 minutes (excluding firmware troubleshooting) |
| **Author** | Cecil Sparks |

---

## Purpose

Provide a reproducible, end-to-end procedure for installing Proxmox VE on a Lenovo M720q from cold start to first successful web UI login.

This runbook assumes:
- Physical access to the target system
- A second computer with USB port and admin rights for preparing boot media
- A wired network connection with known gateway, DNS, and a free static IP
- The [BIOS Hardening Baseline](../baselines/bios-hardening-baseline.md) has been (or will be) applied

---

## Prerequisites

### Hardware

- [ ] Lenovo M720q (or compatible Tiny-class host)
- [ ] At least 8 GB RAM (16 GB recommended)
- [ ] At least 64 GB storage (256 GB+ recommended for real workloads)
- [ ] Wired Ethernet connection to the home/lab network
- [ ] Keyboard and monitor (or KVM) for initial install
- [ ] USB drive, 8 GB or larger
- [ ] UPS or stable AC power

### Network information (gather BEFORE starting)

| Item | How to find it |
|---|---|
| Default gateway | `ipconfig` (Windows) / `ip route` (Linux/macOS) on another LAN device |
| DNS server | Same as above, or check router admin page |
| Free static IP outside DHCP pool | Check router DHCP configuration |
| Subnet mask | Typically `/24` (255.255.255.0) on home networks |

**Do not assume `192.168.1.1` is the gateway.** Verify on a working LAN device first. AT&T Fiber, for example, uses `192.168.1.254`.

### Software

- [ ] Proxmox VE 9.x ISO from [proxmox.com/downloads](https://www.proxmox.com/en/downloads)
- [ ] SHA-256 checksum from the same page
- [ ] Rufus 4.x from [rufus.ie](https://rufus.ie) (Windows)
  *(or `dd` / Etcher on Linux/macOS)*
- [ ] Bitwarden (or equivalent password manager) open and ready to store the root password

---

## Procedure

### Phase 1 — Prepare boot media

1. **Download the Proxmox VE ISO** from the official site
2. **Verify the SHA-256 checksum**:
   ```powershell
   # Windows PowerShell
   Get-FileHash -Algorithm SHA256 .\proxmox-ve_*.iso
   ```
   Compare against the published value on the download page. **Do not proceed if mismatched.**
3. **Launch Rufus 4.x**
4. **Insert the USB drive** (8 GB+) — Rufus should auto-detect it
5. In Rufus:
   - **Device:** Your USB drive
   - **Boot selection:** Click "SELECT" and choose the Proxmox ISO
   - **Partition scheme:** GPT
   - **Target system:** UEFI (non-CSM)
   - Click **START**
   - When prompted "ISO Image mode" or "DD Image mode" — choose **DD Image mode**
   - Confirm the warning that the USB will be wiped
6. **Wait ~5–10 minutes** for the write to complete
7. **Safely eject** the USB drive

### Phase 2 — Prepare the host

1. **Apply the [BIOS Hardening Baseline](../baselines/bios-hardening-baseline.md)** if not already applied
2. **Connect the host** to:
   - Wired Ethernet
   - Monitor
   - Keyboard
   - AC power (preferably through UPS)
3. **Insert the USB boot drive** into a rear USB port (more reliable than front ports on M720q)
4. **Power on** and press **F12** to open the boot menu
5. **Select** the entry labeled `UEFI: USB Partition 2` (or similar)
   - If the USB doesn't appear, see [FINDING-002](../findings/FINDING-002-boot-media-compatibility.md) for compatibility troubleshooting

### Phase 3 — Proxmox installer

1. At the Proxmox boot screen, **choose "Install Proxmox VE (Graphical)"**
2. **Read and accept** the EULA
3. **Target Hard Disk:**
   - Select the internal NVMe drive (verify size matches expectations)
   - Click **Options** if you want non-default filesystem; default `ext4` is fine for a single-disk homelab host
4. **Country / Time zone / Keyboard:** Set appropriately
5. **Password and email:**
   - **Root password:** Generate a strong password in Bitwarden first, then paste
   - **Email:** Use a real address you check — Proxmox sends alerts here
6. **Management network configuration:**
   - **Management interface:** Should auto-detect the wired NIC
   - **Hostname (FQDN):** `proxmox.lab.local` (or your chosen FQDN)
   - **IP Address (CIDR):** `192.168.1.10/24` (or your chosen free static IP)
   - **Gateway:** Your **verified** default gateway (e.g., `192.168.1.254`)
   - **DNS Server:** Same as gateway, or `1.1.1.1` / `9.9.9.9` for public DNS
7. **Summary screen:** Review every value. The gateway field is the most common error source.
8. Click **Install**
9. Wait 10–15 minutes for the install to complete
10. When prompted, click **Reboot**
11. **Remove the USB drive** as the system reboots

### Phase 4 — First boot verification

1. The system should boot to a text-mode login banner showing:
2. **From a separate computer on the same network**, open a browser to `https://<your-proxmox-ip>:8006`
3. **Accept the self-signed certificate warning** (this is expected on a fresh install)
4. **Log in:**
- Username: `root`
- Password: (from Bitwarden)
- Realm: `Linux PAM standard authentication`
5. **Verify** you reach the Proxmox dashboard

If you see a "No valid subscription" popup — that's expected. Click **OK** to dismiss. Proxmox is free; the subscription is optional commercial support.

### Phase 5 — Immediate post-install (within first hour)

1. **Document the install completion** in `journal/`
2. **Confirm the root password is saved** in Bitwarden
3. **Switch from enterprise to no-subscription repo** (so `apt update` doesn't fail):
- In the web UI, navigate to your node → **Updates → Repositories**
- Disable `pve-enterprise` and `ceph-quincy-enterprise`
- Add the `pve-no-subscription` repo
4. **Run system updates** via the node's Shell:
```bash
apt update
apt full-upgrade -y
reboot
```
5. **Take note** of the BIOS firmware version on the host — if outdated, raise a firmware-update finding (see [FINDING-003](../findings/FINDING-003-outdated-firmware.md))

---

## Verification checklist

After completing this runbook, all of these should be true:

- [ ] Host POSTs and boots to the Proxmox login banner
- [ ] Web UI reachable at `https://<ip>:8006`
- [ ] Login with `root` succeeds
- [ ] `ip addr` on the host shows the correct static IP
- [ ] `ping <gateway>` succeeds
- [ ] `ping 1.1.1.1` succeeds (internet reachable)
- [ ] `ping <gateway-domain>` succeeds (DNS working) — e.g., `ping google.com`
- [ ] Root password is saved in password manager
- [ ] Install completion is logged in `journal/`

---

## Troubleshooting

### USB drive not in boot menu

See [FINDING-002](../findings/FINDING-002-boot-media-compatibility.md). Most likely causes:

1. USB written in ISO mode instead of DD mode — re-flash with Rufus DD mode
2. BIOS firmware too old to enumerate the USB — see [FINDING-003](../findings/FINDING-003-outdated-firmware.md)
3. USB Boot disabled in BIOS — check Devices → USB Setup

### Installer hangs on "Loading initial ramdisk"

Typically a corrupt ISO or write error. Re-verify ISO checksum and re-flash USB.

### Network setup fails / can't reach web UI after install

1. Verify you used the **correct gateway** (not assumed `.1`)
2. From another LAN device: `ping <proxmox-ip>` — does it respond?
3. On the host (via attached keyboard): `ip addr` — is the IP set correctly?
4. On the host: `ip route` — is the default route correct?
5. Re-edit `/etc/network/interfaces` if needed, then `systemctl restart networking`

### Locked out of BIOS Administrator account

See [FINDING-001](../findings/FINDING-001-credential-management.md) for CMOS reset procedure.

---

## Rollback procedure

If the install fails or the system needs to be rebuilt from scratch:

1. Boot from the USB installer again
2. Choose the same target disk — Proxmox installer will reformat it
3. **Note:** All data on the target disk is destroyed by reinstallation. There is no "in-place repair" path for Proxmox.

For systems with existing data, ensure backups are verified **before** reinstalling.

---

## Framework mappings

### NIST Cybersecurity Framework 2.0

| Control | Description | How runbook relates |
|---|---|---|
| **PR.IP-1** | Baseline configuration is created and maintained | Reproducible install procedure |
| **PR.IP-3** | Configuration change control processes | Versioned runbook, change log below |
| **RC.RP-1** | Recovery plan is executed | Rollback procedure documented |

### CIS Critical Security Controls v8

| Control | Description | How runbook relates |
|---|---|---|
| **4.1** | Establish and maintain a secure configuration process | This runbook |
| **11.1** | Establish and maintain a data recovery process | Rollback section |

### ISO/IEC 27001:2022 Annex A

| Control | Description | How runbook relates |
|---|---|---|
| **A.5.37** | Documented operating procedures | This document |
| **A.8.32** | Change management | Versioning + change log |

---

## Related documents

- [BIOS Hardening Baseline](../baselines/bios-hardening-baseline.md)
- [FINDING-001 — Credential management](../findings/FINDING-001-credential-management.md)
- [FINDING-002 — Boot media compatibility](../findings/FINDING-002-boot-media-compatibility.md)
- [FINDING-003 — Outdated firmware](../findings/FINDING-003-outdated-firmware.md)

---

## Change log

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-05-16 | Initial runbook based on Day 1 install experience |

---

**Runbook author:** Cecil Sparks II
**First validated:** May 16, 2026 (the hard way — see linked findings)
**Next review:** After Week 2 hardening cycle
