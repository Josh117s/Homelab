# Proxmox Setup

Proxmox VE 9.1 installed as a bare-metal hypervisor on the GMKtec M5 Ultra. All lab services run as virtual machines on top of it.

---

## Hardware

| Component | Details |
|---|---|
| Host | GMKtec M5 Ultra Mini PC |
| CPU | AMD Ryzen 7 7730U (8 cores, 16 threads) |
| RAM | 16GB DDR4 |
| Storage | 512GB NVMe SSD |
| NICs | 2x Realtek r8169 Gigabit Ethernet, 1x WiFi (unused) |

---

## Installation

Flashed the Proxmox ISO using Ventoy. Rufus was the first attempt but DD mode makes the USB read-only after flashing, which blocked editing boot parameters when the installer hit issues.

The installer went to a black screen on first boot due to an AMD Ryzen APU graphics driver conflict. Fixed by injecting the `nomodeset` kernel parameter through a Ventoy config file.

---

## Initial Configuration

**Timezone** 

installer defaulted to Pacific/Honolulu. Set post-install:

```bash
timedatectl set-timezone America/Los_Angeles
```

**Repositories** 

Proxmox ships pointing at the enterprise repo which requires a paid subscription. Every `apt update` fails with 401 Unauthorized without one. Most guides show commands targeting `.list` files. Those don't exist on Proxmox 9.1, which uses DEB822 `.sources` format instead. Used `ls` and `cat` to find the actual files, then disabled the enterprise repos and added the free one.


<img width="900" height="367" alt="repo_migration_output" src="https://github.com/user-attachments/assets/ee41d06c-efd3-4d2a-9633-ca1cd40674ac" />


**Updates** 

ran `apt full-upgrade -y` after the repo fix.


<img width="2556" height="1297" alt="pve_summary_page" src="https://github.com/user-attachments/assets/d78736f2-ef05-4fed-aeeb-f429ed9f647e" />


---

## Issues Encountered

**Black screen on installer boot**
AMD Ryzen APU graphics conflict with the Linux kernel during install. Fixed with `nomodeset` via Ventoy config.

**Rufus unusable for this**
DD mode locks the USB filesystem after flashing. Switched to Ventoy.
