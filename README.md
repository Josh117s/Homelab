# Homelab

I built this lab to gain hands-on experience with enterprise networking and infrastructure. It runs on a GMKtec M5 Ultra with Proxmox as the hypervisor and pfSense handling all routing and firewall rules. The network is segmented into four VLANs enforced at both the firewall and switch level.

---

## Technologies

| Component | Details |
|---|---|
| Hypervisor | Proxmox VE 9.1 |
| Firewall / Router | pfSense CE 2.8.1 |
| Switch | NETGEAR GS308EP |
| Hardware | GMKtec M5 Ultra (Ryzen 7 7730U, 16GB RAM, 512GB NVMe) |

---

## Network

Four VLANs segment the lab into isolated zones. Traffic between them is enforced by pfSense firewall rules.

| VLAN | Subnet | Purpose |
|---|---|---|
| Management (10) | 192.168.10.0/24 | Proxmox, switch, infrastructure |
| Production (20) | 192.168.20.0/24 | Servers, Active Directory, GLPI |
| DMZ (30) | 192.168.30.0/24 | Internet facing services |
| Dev/Sandbox (40) | 192.168.40.0/24 | Testing and experiments |

*Network topology diagram coming soon*

*pfSense dashboard showing all interfaces up*

---

## Documentation

- [Proxmox Setup](infrastructure/proxmox-setup.md)
- [pfSense Setup](infrastructure/pfsense-setup.md)
- [VLAN Configuration](infrastructure/vlan-configuration.md)
- [Switch Configuration](infrastructure/switch-configuration.md)

---

## In Progress

- [Active Directory Project](active-directory-project/README.md) -- Windows Server 2022, domain controller, GPO, PowerShell automation
- [Ticketing System](ticketing-system-project/README.md) -- GLPI with Active Directory integration
