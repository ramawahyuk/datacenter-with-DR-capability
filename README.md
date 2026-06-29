# 🏢 VMware Data Center Simulation with Disaster Recovery Capability

> A complete, production-equivalent simulation of an enterprise data center with full Disaster Recovery (DR) capabilities, built using VMware technologies within a virtualized home lab environment.


## 📌 Project Overview

This project demonstrates the end-to-end design, implementation, and validation of a **simulated enterprise data center** with **Disaster Recovery capabilities** using VMware's virtualization ecosystem.

The lab simulates real-world scenarios such as:
- **Hardware failure** → Automatic VM failover via [Fault Tolerance](https://github.com/ramawahyuk/datacenter-with-DR-capability/blob/main/docs/10-fault-tolerance-setup.md)
- **Software failure** → VM recovery via [vSphere Replication](https://github.com/ramawahyuk/datacenter-with-DR-capability/blob/main/docs/08-replication-setup.md)
- **vCenter Server failure** → [vCenter High Availability (HA) failover](https://github.com/ramawahyuk/datacenter-with-DR-capability/blob/main/docs/09-vcenter-ha-setup.md)
- **Scheduled data protection** → Automated backup via VMware Appliance Management

**Business Context:**  
Modern enterprises cannot afford unplanned downtime. This project simulates how a company can protect its IT infrastructure using VMware's built-in DR features providing a reference architecture for real-world DR implementation.

---

## 🏗️ Architecture Overview

> More information about the system archictecture can refer to this [documentation](https://github.com/ramawahyuk/datacenter-with-DR-capability/tree/main/architecture)

| Component | Hostname | IP Address | OS / Version | Role |
|-----------|----------|------------|--------------|------|
| Domain Controller | vsystem | 192.168.1.10 | CentOS 7 + Samba 4.6 | AD, DNS |
| vCenter Server Appliance | VCSA | 192.168.1.100 | VCSA 6.7 | vSphere Management |
| ESXi Host 1 | Sapcore1 | 192.168.1.34 | ESXi 6.5 | Production Cluster |
| ESXi Host 2 | Sapcore2 | 192.168.1.35 | ESXi 6.5 | Production Cluster |
| ESXi Host 3 | Sapcore3 | 192.168.1.36 | ESXi 6.5 | Production Cluster |
| ESXi Host 4 | Sapcore4 | 192.168.1.37 | ESXi 6.7 | DR Cluster / Replication Target |
| ESXi Host 5 | Sapcore5 | 192.168.1.28 | ESXi 6.7 | vCenter HA Host |
| Backup Server | BACKUPSRV | 192.168.1.99 | Windows Server 2012 R2 | FTP Backup Target |
| Replication Appliance | Photon-machine | 192.168.1.180 | vSphere Replication 8.0 | VM Replication |

---

## 🔁 Disaster Recovery Features Implemented

| DR Feature | Technology Used | Description |
|------------|----------------|-------------|
| **Backup** | VMware Appliance Management | Scheduled & manual vCenter backups to FTP server |
| **Restore** | VCSA Installer (Restore Mode) | Full vCenter restore from FTP backup |
| **Replication** | vSphere Replication 8.0 | VM replication with 10-minute RPO to Cluster SYS B |
| **Failover (vCenter)** | vCenter HA | Active/Passive/Witness 3-node vCenter failover |
| **Failover (VM)** | Fault Tolerance (FT) | Zero-downtime VM failover across ESXi hosts |
| **High Availability** | vSphere HA | Automatic VM restart on host failure |

---

## 📁 Repository Structure

```
vmware-datacenter-dr-lab/
│
├── README.md                        ← You are here
├── LICENSE
│
├── docs/
│   ├── 01-hardware-requirements.md  ← Lab hardware specs for all nodes
│   ├── 02-software-requirements.md  ← Software versions and licensing notes
│   ├── 03-domain-controller-setup.md← CentOS 7 + Samba AD configuration
│   ├── 04-esxi-installation.md      ← ESXi 6.5 / 6.7 installation guide
│   ├── 05-vcenter-installation.md   ← VCSA 6.7 two-stage deployment guide
│   ├── 06-vsan-configuration.md     ← vSAN storage setup with Distributed Switch
│   ├── 07-backup-configuration.md   ← FTP backup server + VMware Appliance Mgmt
│   ├── 08-replication-setup.md      ← vSphere Replication 8.0 deployment
│   ├── 09-vcenter-ha-setup.md       ← vCenter HA Active/Passive/Witness setup
│   ├── 10-fault-tolerance-setup.md  ← NFS storage + VM Fault Tolerance config
│   └── troubleshooting.md           ← Common issues and fixes
│
├── architecture/
│   ├── lab-topology.md              ← Full topology description
│   └── network-diagram.md           ← Network addressing reference
│
├── config/
│   ├── hosts.example                ← /etc/hosts template for all nodes
│   └── samba-smb.conf               ← Samba domain controller config template
│
├── runbooks/
│   ├── DR-backup-restore.md         ← Step-by-step restore runbook
│   ├── DR-replication-failover.md   ← vSphere Replication failover procedure
│   ├── DR-vcenter-ha-failover.md    ← vCenter HA failover runbook
│   └── DR-fault-tolerance.md        ← Fault Tolerance failover procedure
│
└── testing/
    └── alpha-test-results.md        ← Blackbox test results for all DR
```

---

## 🚀 Quick Start — Lab Build Order

Follow this sequence to replicate the lab:

```
Step 1  →  Provision physical/virtual host hardware (see docs/01-hardware-requirements.md)
Step 2  →  Install CentOS 7 Domain Controller + Samba AD (docs/03-domain-controller-setup.md)
Step 3  →  Install ESXi 6.5 on Sapcore1, Sapcore2, Sapcore3 (docs/04-esxi-installation.md)
Step 4  →  Install ESXi 6.7 on Sapcore4, Sapcore5 (docs/04-esxi-installation.md)
Step 5  →  Join vCenter client (Windows Server 2012 R2) to domain
Step 6  →  Deploy vCenter Server Appliance 6.7 (docs/05-vcenter-installation.md)
Step 7  →  Configure Distributed Switch + vSAN (docs/06-vsan-configuration.md)
Step 8  →  Configure Backup Server + FTP (docs/07-backup-configuration.md)
Step 9  →  Deploy vSphere Replication 8.0 (docs/08-replication-setup.md)
Step 10 →  Configure vCenter HA (docs/09-vcenter-ha-setup.md)
Step 11 →  Configure Fault Tolerance (docs/10-fault-tolerance-setup.md)
Step 12 →  Run DR tests (testing/alpha-test-results.md)
```

---

## ⚙️ Technologies Used

| Category | Technology | Version |
|----------|-----------|---------|
| Hypervisor | VMware ESXi | 6.5 / 6.7 |
| vCenter | VMware vCenter Server Appliance | 6.7 |
| Storage | VMware vSAN | Included in vCenter 6.7 |
| Replication | vSphere Replication | 8.0.0 |
| HA | vCenter High Availability | Built-in |
| Fault Tolerance | VMware Fault Tolerance | Built-in |
| Domain Controller | CentOS 7 + Samba | 4.6.0 |
| Backup Target OS | Windows Server | 2012 R2 |
| Backup Protocol | FTP via IIS | — |
| Virtualization Platform | VMware Workstation | — |
| Shared Storage (FT) | NFS (Windows Server NFS role) | — |

---

## 📊 Testing Results Summary

### Alpha Testing (Blackbox — Functional)

> More information regarding the system testing can refer to this [documentation](https://github.com/ramawahyuk/datacenter-with-DR-capability/blob/main/testing/alpha-test-results.md)

| Feature | Correct Input Result | Incorrect Input Result |
|---------|---------------------|----------------------|
| Backup Schedule | ✅ Backup executed on schedule | ❌ Connection to FTP failed |
| Manual Backup | ✅ Backup files created on server | ❌ Authentication failed |
| Restore | ✅ VM restored from backup | ❌ Restore aborted |
| Replication | ✅ VM replicated in 6m 30s | ❌ Replication failed (DNS down) |
| vCenter HA Failover | ✅ vCenter back in ~29m 34s | ❌ Failover blocked |
| Fault Tolerance | ✅ VM auto-migrated (zero downtime) | ❌ FT not protected |

### Failover RTO Timeline (vCenter HA)

| Phase | Duration |
|-------|----------|
| vCenter goes unreachable | ~27 seconds |
| vCenter services restored (ping) | ~1 minute 46 seconds |
| vSphere Web Client initializing | ~25 minutes |
| Full access to vCenter Web Client | **~29 minutes 34 seconds** |

---

## 📚 Lessons Learned

- **DNS is critical** — vSphere Replication fails completely if the Domain Controller DNS is unavailable. Always ensure DNS redundancy.
- **vSAN requires a minimum of 3 hosts** — each with a dedicated flash (cache) disk and a capacity disk.
- **vCenter HA uses a separate subnet** — the HA network (192.168.0.x) must be on a different subnet from the management network.
- **NFS shared storage is required for Fault Tolerance** — VMs using FT cannot reside on local datastores.
- **FTP backup paths are strict** — the exact format `ftp://IP:port/folder` must be used in VMware Appliance Management.
- **Thin Provisioning** saves lab storage significantly — always select it for vSphere Replication and VCSA deployments.

---

## 🔮 Future Improvements

- [ ] Migrate to VMware vSphere 8.x (current generation)
- [ ] Implement VMware Site Recovery Manager (SRM) for automated failover orchestration
- [ ] Add vRealize Operations Manager for monitoring and capacity planning
- [ ] Extend replication to a cloud-based DR site (VMware Cloud on AWS)
- [ ] Automate ESXi host configuration using Ansible
- [ ] Add Terraform vSphere provider for infrastructure-as-code provisioning
- [ ] Implement vSphere Distributed Resource Scheduler (DRS) for workload balancing
- [ ] Add NSX-T for micro-segmentation and network virtualization

---

## 🛠️ Skills Demonstrated

`VMware ESXi` `vCenter Server` `vSphere Replication` `vSAN` `vCenter HA`  
`VMware Fault Tolerance` `Disaster Recovery` `NFS Storage` `Active Directory`  
`CentOS Linux` `Windows Server` `FTP Server (IIS)` `Networking` `System Administration`  
`Blackbox Testing` `Technical Documentation` `Home Lab to Enterprise Architecture`

---

## 📄 License

MIT License — see [LICENSE](LICENSE)

---

## 📬 Author

Built as part of a professional portfolio demonstrating enterprise-level VMware infrastructure and Disaster Recovery engineering capabilities.

> *"This lab was designed to mirror real-world data center operations, providing hands-on experience with VMware's DR ecosystem without requiring access to physical enterprise hardware."*
