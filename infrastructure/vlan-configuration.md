# VLAN Configuration

The network is segmented into four isolated VLANs enforced by pfSense firewall rules. Each VLAN can reach the internet but cannot reach other VLANs unless explicitly allowed.

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| 10 | Management | 192.168.10.0/24 | Proxmox, switch, infrastructure |
| 20 | Production | 192.168.20.0/24 | Servers, Active Directory, GLPI |
| 30 | DMZ | 192.168.30.0/24 | Internet-facing services |
| 40 | Dev/Sandbox | 192.168.40.0/24 | Testing and experiments |

---

## Creating the VLANs

I created all four VLANs in pfSense under Interfaces → Assignments → VLANs, using `vtnet0` as the parent interface -- the LAN-facing NIC connected to the NETGEAR switch.

*Four VLANs defined on vtnet0 -- tags 10, 20, 30, 40.*

<img width="1523" height="517" alt="pfsense-vlan-config" src="https://github.com/user-attachments/assets/6fa4054c-578b-4647-a135-25c08c42d11e" />


---

## Interface Assignments

VLANs 20, 30, and 40 were assigned as named interfaces. VLAN 10 was not assigned as a separate interface because the existing LAN interface already serves `192.168.10.x` at `192.168.10.1`. Creating a separate `vtnet0.10` sub-interface would put two pfSense interfaces on the same subnet -- a direct conflict.

VLAN 10 stays in the definitions list so the switch can reference it when configuring the trunk port. When I deleted the VLAN 10 OPT assignment, pfSense renumbered the remaining interfaces as OPT2, OPT3, OPT4. The numbers don't matter once interfaces are renamed.

*Interface Assignments showing WAN, LAN, PROD, DMZ, DEV.*

<img width="1469" height="803" alt="pfsense-vlan-configs-complete" src="https://github.com/user-attachments/assets/9a9b6374-3a6c-423c-ac83-1c86333fdbeb" />


Each interface was given a static IP -- pfSense acts as the gateway for that VLAN and every device uses it as their default route, so the address needs to be permanent.

| Interface | VLAN | IP |
|---|---|---|
| LAN | 10 | 192.168.10.1 |
| PROD | 20 | 192.168.20.1 |
| DMZ | 30 | 192.168.30.1 |
| DEV | 40 | 192.168.40.1 |

Block private networks and Block bogon networks were left unchecked on all internal interfaces -- those options are for WAN-facing interfaces only. Enabling them internally would block traffic from your own devices.

---

## DHCP

I enabled a DHCP server on each interface with the pool set to `.100-.199`, leaving `.2-.99` open for static infrastructure assignments.

*DHCP server configured on PROD interface showing subnet and address pool.*

<img width="1538" height="928" alt="vlan-address-pool-config" src="https://github.com/user-attachments/assets/c97009a1-cac7-4b4a-82a3-fbce2651e990" />


---

## Firewall Rules

Block rules sit above the general allow rule on each interface. Rules are evaluated top to bottom -- first match wins. If the allow rule came first, every packet would match it before the block rules were ever checked.

Source was set to subnets rather than address on all rules. Using address would only match traffic from pfSense's own interface IP -- actual devices on the VLAN would bypass the rules entirely.

**PROD rules:**

| Action | Source | Destination |
|---|---|---|
| Block | PROD subnets | 192.168.10.0/24 (Management) |
| Block | PROD subnets | 192.168.30.0/24 (DMZ) |
| Block | PROD subnets | 192.168.40.0/24 (DEV) |
| Allow | PROD subnets | any |

DMZ and DEV follow the same pattern, blocking all other internal VLANs and allowing internet access.

*PROD firewall rules -- block rules above the allow rule.*

<img width="1642" height="556" alt="firewall-rules-pfsense" src="https://github.com/user-attachments/assets/2eed18e8-87e0-4b91-aead-352beb336fb5" />


I caught three errors before applying the rules:

- A typo in the PROD Management block -- `192.169.10.0` instead of `192.168.10.0`. The rule would have saved and appeared correct but blocked a non-existent network while leaving Management completely open.
- All DMZ rules had source set to "DMZ address" instead of "DMZ subnets" -- only pfSense's own interface IP would have matched, leaving all DMZ devices unaffected by the rules.
- Some DEV rules had "DEV address" instead of "DEV subnets" -- same issue. Fixed before applying.

---

## Verification

After applying everything, I confirmed all five interfaces were up with correct IPs on the pfSense dashboard.

*pfSense dashboard showing WAN, LAN, PROD, DMZ, and DEV all up.*

<img width="755" height="339" alt="pfsense-dashboard-all-interfaces-up" src="https://github.com/user-attachments/assets/c2f7c363-d559-48e4-9e0e-836494c90efd" />


DMZ and DEV showed `n/a` instead of an IP after the initial save. They had been saved but not fully applied. Going into each interface and clicking Apply Changes brought them up correctly.
