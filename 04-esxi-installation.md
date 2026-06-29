# ESXi Host Installation Guide

This guide covers the installation of VMware ESXi 6.5 and ESXi 6.7 on the lab hosts.

---

## Host Assignment

| Hostname | IP Address | ESXi Version | Cluster | Role |
|----------|------------|-------------|---------|------|
| Sapcore1 | 192.168.1.34 | 6.5 | SYS A | Production |
| Sapcore2 | 192.168.1.35 | 6.5 | SYS A | Production |
| Sapcore3 | 192.168.1.36 | 6.5 | SYS A | Production |
| Sapcore4 | 192.168.1.37 | 6.7 | SYS B | DR Target |
| Sapcore5 | 192.168.1.28 | 6.7 | VCSA | vCenter HA |

---

## Network Configuration Reference

| Hostname | VMkernel Port | Service | IP Address | Switch |
|----------|--------------|---------|------------|--------|
| Sapcore1 | Vmk0 | Management | 192.168.1.34 | DstrbtdSwitch |
| Sapcore1 | Vmk2 | vMotion | 192.168.1.80 | DstrbtdSwitch |
| Sapcore1 | Vmk3 | vSAN | 192.168.1.70 | DstrbtdSwitch |
| Sapcore2 | Vmk0 | Management | 192.168.1.35 | DstrbtdSwitch |
| Sapcore2 | Vmk2 | vMotion | 192.168.1.81 | DstrbtdSwitch |
| Sapcore2 | Vmk3 | vSAN | 192.168.1.71 | DstrbtdSwitch |
| Sapcore3 | Vmk0 | Management | 192.168.1.36 | DstrbtdSwitch |
| Sapcore3 | Vmk2 | vMotion | 192.168.1.83 | DstrbtdSwitch |
| Sapcore3 | Vmk3 | vSAN | 192.168.1.72 | DstrbtdSwitch |
| Sapcore4 | Vmk0 | Management | 192.168.1.37 | vSwitch0 |
| Sapcore5 | Vmk0 | Management | 192.168.1.28 | vSwitch0 |

---

## Prerequisites

- VMware ESXi 6.5 ISO (for Sapcore1–3)
- VMware ESXi 6.7 ISO (for Sapcore4–5)
- Each VM configured in VMware Workstation with the specs in `docs/01-hardware-requirements.md`
- Nested virtualization enabled in VMware Workstation VM settings

---

## Installation Steps (Same for ESXi 6.5 and 6.7)

### Step 1 — Boot from ISO

1. Power on the VM in VMware Workstation.
2. The system boots from the ESXi installer ISO.
3. The VMware ESXi boot menu appears — press **Enter** to start the installer.

### Step 2 — Accept the License Agreement

Press **F11** to accept the VMware End User License Agreement (EULA).

### Step 3 — Select Installation Disk

The installer scans for available storage devices.

- Select the appropriate disk for the ESXi boot installation:
  - Sapcore1/2/3: Select the **50 GB** disk (not the flash or vSAN disks)
  - Sapcore4: Select the **250 GB** disk
  - Sapcore5: Select the **300 GB** disk
- Press **Enter** to confirm.

### Step 4 — Select Keyboard Layout

Select your preferred keyboard layout (default: US) and press **Enter**.

### Step 5 — Set Root Password

Enter and confirm a strong root password.

> **Lab password used:** `Sapcore00!` (change this in production environments)

### Step 6 — Confirm Installation

The installer displays a warning that the selected disk will be partitioned.

Press **F11** to begin the installation.

### Step 7 — Reboot

When installation completes, press **Enter** to reboot the host.

---

## Post-Installation Network Configuration

After reboot, configure the management network:

1. At the ESXi DCUI (Direct Console User Interface), press **F2** to customize the system.
2. Log in as `root`.
3. Select **Configure Management Network**.
4. Select **IPv4 Configuration**.
5. Set **Static** IP and enter the values from the network table above.
6. Select **DNS Configuration** and set the DNS server to `192.168.1.10` (domain controller).
7. Press **Escape** to exit and apply changes.

---

## Verification

After configuration, verify connectivity:

```bash
# From another host, ping the ESXi management IP
ping 192.168.1.34

# Access the ESXi web interface
# Open browser: https://192.168.1.34
```

SSH access (after enabling SSH from DCUI → Troubleshooting Options → Enable SSH):
```bash
ssh root@192.168.1.34
```

---

## Notes for Nested ESXi in VMware Workstation

When running ESXi as a nested VM inside VMware Workstation, you must enable the following in the VM settings:

- **Virtualize Intel VT-x/EPT or AMD-V/RVI** — required for nested hypervisor
- **Virtualize IOMMU (IO memory management unit)** — recommended
- Set the **CPU compatibility** level appropriately for your physical host CPU

Additionally, ensure the VM's network adapter is set to **Bridged** mode so ESXi hosts can communicate with each other on the same physical network segment.
