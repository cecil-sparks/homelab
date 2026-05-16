# Week 1 · Day 1 — Proxmox install marathon

**Date:** May 15–16, 2026
**Duration:** ~6 hours (with multiple stuck points)
**Outcome:** ✅ Proxmox VE 9.1 operational on bare metal

---

## TL;DR

Spent most of the night fighting BIOS boot media issues on a Lenovo M720q. Ventoy refused to enumerate. Discovered I'd lost the BIOS Administrator password from initial setup back in April. Performed a CMOS reset, rebuilt the BIOS hardening baseline from scratch, flashed a Rufus DD-mode USB, and successfully installed Proxmox VE 9.1. Web UI reachable from my laptop at `https://192.168.1.10:8006`.

Three findings documented. One still open.

---

## What I was trying to do

Install Proxmox VE 9.1 on the M720q so I'd have a working hypervisor as the foundation for the rest of the homelab build.

## What actually happened

### Stuck Point 1: Ventoy USB not detected by BIOS

I'd prepped a Ventoy USB weeks ago expecting it to "just work." It didn't — the M720q BIOS wouldn't list it as a boot option at all. Tried every USB port, toggled CSM, adjusted USB Enumeration Delay, disabled Smart USB Protection. No dice.

→ Documented as [FINDING-002](../docs/findings/FINDING-002-boot-media-compatibility.md)

### Stuck Point 2: Locked out of BIOS

Tried to dig deeper into BIOS settings and realized I'd set an Administrator password back on April 29 during initial setup and never wrote it down. Classic credential management failure on a system I own.

→ Documented as [FINDING-001](../docs/findings/FINDING-001-credential-management.md)

**Remediation:** Powered off, opened the case (single thumbscrew + slide top), disconnected the CR2032 coin battery for 5 minutes, reconnected, reassembled. CMOS reset to defaults. Set a new password and saved it to Bitwarden immediately, with a paper backup in my safe.

### Stuck Point 3: Rebuilding BIOS hardening

After the CMOS reset, every BIOS setting was back to factory defaults. Walked through every tab and re-applied a documented hardening baseline. Where I had to leave a control in a less-hardened state (e.g., CSM enabled for installer compatibility), I documented the exception and the remediation date.

→ See [BIOS Hardening Baseline v1.0](../docs/baselines/bios-hardening-baseline.md)

### Stuck Point 4: Network gateway assumption

During the Proxmox installer's network step, I'd planned to use `192.168.1.1` as the gateway. Checked my actual router and discovered AT&T Fiber's gateway is `192.168.1.254`. Updated the config in the installer.

**Lesson:** Verify the actual network before committing values in an installer. A gateway typo would have left the host unreachable.

---

## Configuration as deployed

| Setting | Value |
|---|---|
| Hostname | `proxmox.lab.local` |
| Management IP | `192.168.1.10/24` |
| Gateway | `192.168.1.254` |
| DNS | `192.168.1.254` |
| Filesystem | ext4 |
| Disk | WDC PC SN520 500 GB NVMe |
| Web UI | `https://192.168.1.10:8006` |

---

## What I learned

1. **Credential management is the boring control that bites first.** Losing the BIOS password cost me an hour and required physical access. In a corporate environment this would have been a P1 ticket and a security incident.
2. **Boot media compatibility is real.** Ventoy's clever multi-ISO trick relies on BIOS features that not every system implements. Rufus DD-mode writes the ISO byte-for-byte and "just works" — at the cost of one ISO per USB.
3. **Always verify the network before you commit it.** Don't assume default gateways.
4. **Document as you go, not after.** Every screenshot, every setting, every dead-end gets logged. That's the audit trail.

---

## What's next

**Tomorrow (Week 1 Day 2):**
- Log in to Proxmox web UI and explore the interface
- Switch from the enterprise repo to the no-subscription repo
- Run `apt update && apt full-upgrade -y`
- Install Tailscale for out-of-band access (so I don't need to be on the home LAN to reach the host)
- Take a snapshot of the clean-install state

**This week:**
- Plan firewall VM (pfSense or OPNsense)
- Sketch VLAN segmentation plan
- Start drafting Week 1 retrospective

**Open finding to address:**
- [FINDING-003](../docs/findings/FINDING-003-outdated-firmware.md) — BIOS firmware from July 2019. Needs an update before this host runs anything that matters.

---

## Sleep status

🟢 Earned it.
