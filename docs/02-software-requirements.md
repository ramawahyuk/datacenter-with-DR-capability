# Software Requirements

This document describes all software used in the Data Center DR Simulation Lab.

---

## Domain Controller

| Component | Software | Version / Notes |
|-----------|----------|----------------|
| Operating System | CentOS 7 | Community Enterprise OS, open source |
| Remote Console | PuTTY / Linux Terminal | SSH access |
| Virtualization Platform | VMware Workstation | Runs DC as a nested VM |
| File Sharing / AD | Samba Server | 4.6.0 — provides Active Directory and DNS |

---

## ESXi 6.5 Hosts (Sapcore1, Sapcore2, Sapcore3)

| Component | Software | Version / Notes |
|-----------|----------|----------------|
| Operating System | VMware ESXi | 6.5 |
| Remote Console | PuTTY | SSH access to ESXi shell |
| Virtualization Platform | VMware Workstation | Runs ESXi as nested VMs |
| Management Browser | Google Chrome | Access to DCUI / ESXi web client |

---

## ESXi 6.7 Hosts (Sapcore4, Sapcore5)

| Component | Software | Version / Notes |
|-----------|----------|----------------|
| Operating System | VMware ESXi | 6.7 |
| Remote Console | PuTTY | SSH access |
| Virtualization Platform | VMware Workstation | Runs ESXi as nested VMs |
| Management Browser | Google Chrome | Access to vSphere Web Client |

---

## vCenter Client (Windows Server 2012 R2)

| Component | Software | Version / Notes |
|-----------|----------|----------------|
| Operating System | Windows Server 2012 R2 | Hosts the VCSA installer |
| Remote Console | PuTTY, CMD | Local and remote access |
| Virtualization Platform | VMware Workstation | Nested VM |
| Server Management | Server Manager | For Roles & Features installation |
| Management Browser | Google Chrome | Run VCSA installer GUI |

---

## Backup Server (BACKUPSRV — Windows Server 2012 R2)

| Component | Software | Version / Notes |
|-----------|----------|----------------|
| Operating System | Windows Server 2012 R2 | FTP backup target |
| Remote Console | PuTTY, CMD | Administration |
| Virtualization Platform | VMware Workstation | Nested VM |
| Server Management | Server Manager | IIS + FTP role installation |
| FTP Service | IIS (Internet Information Services) | Windows built-in web server |
| FTP Validation Client | WinSCP | Verify FTP connectivity and backup files |

---

## VMware Software Versions Reference

| Software | Version | Purpose |
|----------|---------|---------|
| VMware ESXi | 6.5 | Hypervisor for production cluster |
| VMware ESXi | 6.7 | Hypervisor for DR cluster and VCSA host |
| VMware vCenter Server Appliance (VCSA) | 6.7 | Central management platform |
| VMware vSAN | Included in vCenter 6.7 | Distributed shared storage |
| vSphere Replication | 8.0.0 | VM-level asynchronous replication |
| VMware Workstation | Latest | Lab virtualization platform (outer layer) |

---

## Licensing Notes

> This lab uses **evaluation/trial licenses** for all VMware products. For production environments, proper VMware licensing must be purchased.

- VMware ESXi free license supports basic features; vCenter evaluation license is required for clusters, vSAN, and HA.
- Samba is open source (GPL) and free to use.
- CentOS 7 is open source and free to use.
- Windows Server 2012 R2 requires a valid Microsoft license; evaluation versions (180-day trial) are available from Microsoft.
