# pfSense Setup

pfSense CE 2.8.1 runs as a VM inside Proxmox, handling all routing, firewall rules, and DHCP for the environment.

---

## Creating the VM

The VM needs two NICs, one for WAN, one for LAN so pfSense has a separate interface on each side of the network. Both use VirtIO drivers.

*firewallvm hardware tab - UEFI, q35, VirtIO SCSI, both NICs. net0 on vmbr0 (LAN), net1 temporarily on vmbr0 until vmbr1 was created.*

<img width="2538" height="1027" alt="net1-pfSense-vmbr0-to-vmbr1" src="https://github.com/user-attachments/assets/931cae08-d630-4550-aed7-68024c4e3e79" />


Note: Pre-Enroll Keys was unchecked during VM creation - it defaults to on but Secure Boot keys are Windows-only and will prevent pfSense from booting.

*After moving net1 to vmbr1 - net0 stays on vmbr0 (LAN), net1 is now on vmbr1 (WAN).*

<img width="2554" height="677" alt="net-pfSense-vmbr1-after" src="https://github.com/user-attachments/assets/40095b96-7249-4a77-b8c4-820cacf5efe9" />


---

## Installing pfSense

Downloaded the ISO from Netgate, extracted it, and uploaded it to Proxmox so it could be mounted to the VM.

Changed the LAN IP from the default `192.168.1.1` to `192.168.10.1` during installation. The ISP router uses `192.168.1.x` -- two devices acting as routers on the same subnet means neither can properly route traffic, so a different subnet was needed.

ZFS was selected as the filesystem during installation for its checksumming, snapshots, and copy-on-write -- more reliable than ext4 for a system handling all network traffic.

The DHCP pool was set to `.100-.199`, deliberately leaving `.2-.99` open for static infrastructure assignments like Proxmox at `.10` and future servers.

After install, the console confirmed both interfaces:

```
WAN → vtnet1 → 192.168.1.237/24 (DHCP)
LAN → vtnet0 → 192.168.10.1/24 (static)
```

*pfSense console after installation confirming WAN and LAN are both up.*

<img width="2557" height="1068" alt="pfSense_install_confirm_lan-wan blurred" src="https://github.com/user-attachments/assets/8f37db87-b75d-4f17-b635-88bb8eb593f5" />


---

## Setting Up the WAN Bridge

Both NICs started on `vmbr0` because `vmbr1` didn't exist when the VM was created. With both on the same bridge pfSense couldn't actually separate or filter traffic between WAN and LAN. Traffic would cross the bridge without going through the firewall.

Created `vmbr1` in Proxmox and attached it to `nic0`, which connects to the ISP router. Then moved `net1` over. Proxmox supports live NIC edits so no reboot was needed.

`nic0` was showing as DOWN despite a cable being plugged in. `ip link show` confirmed the interface existed and the driver was loaded but it had never been brought up. Proxmox only activates interfaces attached to a configured bridge. Ran `ip link set nic0 up` to confirm the physical link was fine, then creating `vmbr1` made it permanent.

| Bridge | NIC | Role |
|---|---|---|
| vmbr0 | nic1 | LAN -- NETGEAR switch |
| vmbr1 | nic0 | WAN -- ISP router |

*Proxmox network page -- vmbr0 on nic1 (LAN), vmbr1 on nic0 (WAN), both active.*

<img width="2556" height="358" alt="Step 6 — pfSense network Configuration (Pre WAN Migration)" src="https://github.com/user-attachments/assets/8ffcf4e5-a00f-4e2a-b8b8-22d3a23b6c16" />

---

## Proxmox IP Migration

Proxmox was still on the ISP router's subnet at `192.168.1.73`. Once pfSense took over routing, anything on `192.168.1.x` couldn't reach the Management VLAN at `192.168.10.x`, Proxmox would become unreachable. Updated `/etc/network/interfaces` with `sed` to move it to `192.168.10.10`:

```bash
sed -i 's/192.168.1.73\/24/192.168.10.10\/24/' /etc/network/interfaces
sed -i 's/gateway 192.168.1.1/gateway 192.168.10.1/' /etc/network/interfaces
```

*The interfaces file showing the sed commands used to migrate Proxmox onto the Management VLAN.*

![interfaces file](insert)

---

## Web GUI and Initial Configuration

The GUI is at `https://192.168.10.1`. Connected the management PC through the NETGEAR switch to get on the LAN side, confirmed it got a `192.168.10.x` address from pfSense DHCP, then opened the browser.

First things done in the setup wizard:
- Changed the admin password -- the default is publicly documented
- Enabled RFC1918 and bogon blocking on WAN -- drops spoofed private IPs and invalid address ranges before they hit any firewall rules
- Set the domain to `home.arpa` -- reserved for home networks, guaranteed not to collide with public domains

After a Proxmox reboot, pfSense didn't come back up and I lost all network access -- without Start at Boot enabled, any Proxmox reboot leaves pfSense stopped and takes down the entire network. Enabled it in the VM options.

*firewallvm Options tab -- Start at Boot set to Yes.*

![start at boot](insert)

Client PCs were getting `192.168.1.x` addresses instead of `192.168.10.x`. The ISP router was plugged into the NETGEAR switch and winning the DHCP race over pfSense. Removing it from the switch fixed it.

*pfSense dashboard showing all interfaces up after initial configuration.*

![pfSense dashboard](insert)
