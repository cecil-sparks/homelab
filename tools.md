# 🧰 Tools & Software

The software stack I'm using to learn, build, and document this homelab — organized by purpose. Every tool here is free for personal use, and most are industry-standard in real GRC and IT audit work.

---

## 🛡️ Security & Identity

| Tool | Purpose | GRC Mapping |
|------|---------|-------------|
| [Bitwarden](https://bitwarden.com) | Password manager — strong unique credentials for every service | CIS Control 5 (Account Management), NIST CSF PR.AC-1 |
| [Tailscale](https://tailscale.com) | Zero-trust VPN for secure remote access to the homelab | NIST CSF PR.AC-3, CIS Control 12.7 |

---

## 📝 Documentation & Knowledge

| Tool | Purpose | Why it matters in GRC |
|------|---------|------------------------|
| [Obsidian](https://obsidian.com) | Local Markdown note-taking — Security+ notes, control mappings, journal drafts | Auditors live in documentation; building this habit early is rehearsal for the job |
| [VS Code](https://code.visualstudio.com) | Code editor with Markdown live preview, Git integration, YAML/JSON support | Daily driver for editing repo content and config files |
| [GitHub Desktop](https://desktop.github.com) | GUI for cloning, committing, and pushing to this repo | Faster iteration than editing in the browser |

**VS Code extensions I use:** Markdown All in One · markdownlint · GitLens · YAML

---

## 🔍 Compliance & Assessment Tools

These are the resume-worthy ones — actual tools used in real GRC and audit work.

| Tool | Purpose | Use Case |
|------|---------|----------|
| [CIS-CAT Lite](https://www.cisecurity.org/cybersecurity-tools/cis-cat-lite) | Free configuration assessment scanner against CIS Benchmarks | Will scan Proxmox host and VMs; produce baseline compliance reports as evidence |
| [OpenSCAP / SCAP Workbench](https://www.open-scap.org) | Industry-standard compliance scanner; SCAP is a NIST-developed standard | U.S. federal agencies use this — strong resume signal |
| [Nmap](https://nmap.org) (with Zenmap GUI) | Network discovery and inventory | Asset inventory (CIS Control 1) for home network and homelab subnets |
| [Wireshark](https://www.wireshark.org) | Packet analyzer | Useful for Security+ exam objectives and validating segmentation later |

---

## 🔌 Remote Access & File Transfer

| Tool | Platform | Purpose |
|------|----------|---------|
| Built-in `ssh` | macOS / Linux | SSH into Proxmox host and VMs |
| [PuTTY](https://www.putty.org) | Windows | SSH client |
| [WinSCP](https://winscp.net) | Windows | Drag-and-drop file transfer over SSH |
| [Cyberduck](https://cyberduck.io) | macOS | Drag-and-drop file transfer over SSH |

---

## 📚 Study Resources

| Resource | Why |
|----------|-----|
| [Professor Messer SY0-701](https://www.professormesser.com) | Best free Security+ video course |
| [NIST CSF 2.0](https://www.nist.gov/cyberframework) | The framework I'm mapping every control to — ~30 pages, free |
| [CIS Controls v8](https://www.cisecurity.org/controls) | The 18 prioritized controls; focus on Implementation Group 1 (IG1) first |
| [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/) | Official admin guide for the hypervisor |

---

## 🗓️ Pre-Build Tool Prep Schedule

The week of April 26 – May 3, 2026, while I wait for the M720q to arrive:

| Day | Plan |
|-----|------|
| Sun Apr 26 | Install Bitwarden, Obsidian, VS Code, GitHub Desktop |
| Mon Apr 27 | Clone this repo locally; watch Professor Messer Episodes 1–3 |
| Tue Apr 28 | Install Tailscale on laptop + phone; create account |
| Wed Apr 29 | Download CIS-CAT Lite; run a baseline scan on my own laptop to learn the tool |
| Thu Apr 30 | Read NIST CSF 2.0 Core — the six functions (Govern, Identify, Protect, Detect, Respond, Recover) |
| Fri May 1 | Read CIS Controls v8 — focus on IG1 |
| Sat May 2 | Skim Proxmox installation chapter |
| Sun May 3 | Light review; prep workspace for M720q arrival |

---

## 🎯 Why This List Exists

A GRC analyst is judged not just on what they know, but on what they've **touched**. Listing CIS-CAT, OpenSCAP, Nmap, and the major frameworks on a resume is one thing — having actually run them, documented findings, and remediated issues in a homelab is what gets the interview.

Every tool above will eventually produce evidence that lives in this repo:

- Scan reports → `/controls/evidence/scans/`
- Configuration baselines → `/controls/baselines/`
- Network inventory → `/controls/asset-inventory.md`
- Risk register → `/controls/risk-register.md`

— Cecil
