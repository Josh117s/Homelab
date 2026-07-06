# Switch Configuration

The NETGEAR GS308EP is an 8-port managed switch that enforces the VLAN segmentation at the physical layer. Once configured, devices are isolated to their assigned VLAN based on which port they plug into.

<img width="2373" height="941" alt="Netgear-switch-dashboard" src="https://github.com/user-attachments/assets/ad1cb3b3-44a4-4da8-9015-bad9795d2b34" />

---

## Accessing the Switch

The switch management interface is a web GUI accessible over HTTP. It received a DHCP lease from pfSense on the Management VLAN at `192.168.10.106`. Before any VLAN configuration was applied, all traffic was untagged and landed on the LAN interface by default -- which is why the switch was reachable at a `192.168.10.x` address right away.

Default credentials are printed on a label on the bottom of the switch.

---

## Selecting the VLAN Mode and Creating VLANs

The switch offers five VLAN modes. Basic 802.1Q was selected because it supports Access and Trunk port configuration with proper 802.1Q tagging, the same standard pfSense uses. Port-based VLAN modes don't use 802.1Q tags, which means pfSense would have no way to identify which VLAN incoming frames belong to.

Four VLANs were added to match the definitions in pfSense. The IDs must match exactly, if the switch tags a frame with ID 20 and pfSense has no interface configured for VLAN 20, the frame gets dropped. VLAN 1 is the factory default and can't be deleted. It carries no active traffic in this setup.

| VLAN ID | Name |
|---|---|
| 10 | Management |
| 20 | Production |
| 30 | DMZ |
| 40 | Dev-Sandbox |

*Basic 802.1Q selected as the VLAN mode. Edit VLAN tab showing all 
four VLANs added to match pfSense.*

<img width="1208" height="1173" alt="vlans-added-switch" src="https://github.com/user-attachments/assets/cc359163-5855-4134-b19a-6fb69962fecd" />



---

## Port Configuration

Port 1 connects to the GMKtec via `nic1` and carries all four VLANs simultaneously, it's the trunk port. Every other port is an access port assigned to a single VLAN. Devices plugged into access ports send and receive normal untagged traffic where the switch handles the tagging invisibly.

| Port | Mode | VLAN | Connected To |
|---|---|---|---|
| 1 | Trunk | All VLANs | GMKtec nic1 (pfSense uplink) |
| 2 | Access | 10 -- Management | Management PC |
| 3 | Access | 10 -- Management | Spare |
| 4 | Access | 20 -- Production | Spare |
| 5 | Access | 30 -- DMZ | Spare |
| 6 | Access | 40 -- Dev | Spare |
| 7 | Access | 10 -- Management | Spare |
| 8 | Access | 10 -- Management | Spare |

Multiple ports were assigned to Management because infrastructure devices that need to reach Proxmox, pfSense, and each other all belong on that VLAN. Additional ports are reserved for devices added as the lab grows.

*Switch port configuration -- port 1 as trunk, remaining ports as access.*

<img width="610" height="794" alt="switch-vlan-config" src="https://github.com/user-attachments/assets/e8d672f0-5af1-40f8-824a-0243a433dca8" />


---

## Verification

After saving the switch configuration, I ran `ipconfig /release` and `ipconfig /renew` on the management PC to confirm it was getting a correct lease from pfSense:

```
Connection-specific DNS Suffix: home.arpa
IPv4 Address: 192.168.10.103
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

The `home.arpa` DNS suffix confirms pfSense's domain configuration is propagating correctly to clients. This confirms physical port tagging, the trunk, pfSense VLAN interfaces, and DHCP are all working correctly end to end.

*ipconfig output showing correct Management VLAN lease from pfSense.*

<img width="749" height="386" alt="pc_dhcp_lease_switch_implementation" src="https://github.com/user-attachments/assets/425e2fd6-87e0-4ada-a3b3-dd0510228a12" />
