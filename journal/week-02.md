# Home Lab Progress Log: Infrastructure & Network Core Deployed

## 🚀 Accomplishments Over the Past 3 Days
- **Hypervisor Preparation**: Successfully finalized the Lenovo ThinkCentre M750q node running Proxmox VE 9.1.11.
- **Container Deployment**: Provisioned an isolated Ubuntu 24.04 LTS LXC container (`ID 100`) and optimized resource allocations to 2.00 GiB RAM / 512 MiB Swap.
- **Orchestration Layer**: Installed the Docker runtime engine and deployed the **Dockge** visual stack management panel to orchestrate future containerized applications.
- **Network-Wide Security**: Successfully deployed a **Pi-hole v6** container utilizing `network_mode: "host"` and automated network administration privileges (`NET_ADMIN`).
- **DHCP Migration**: Disrupted AT&T Fiber Gateway default routing loops by migrating master DHCP address allocation authority directly to the Pi-hole container (managing lease pool `.112` through `.253`).
- **Data Protection**: Captured a pristine restoration snapshot (`Week2Day3Working`) in Proxmox to safeguard the verified functional state.

## 🛠️ Technical Hurdles Overcome
- **LXC Storage Bottlenecks**: Resolved initialization failures by troubleshooting and expanding the container RAM envelope from 512MB to 2GB.
- **Port 53 Conflicts**: Disabled conflicting system services (`systemd-resolved`) to allow Pi-hole to claim the local DNS lane.
- **Routing Loop Demolition**: Overrode stubborn host-injected VPN search domains by configuring a permanent system network lock file (`/etc/.pve-ignore.resolv.conf`) pointing explicitly to public upstream nameservers.
