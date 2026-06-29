# Hardware Requirements

This document describes the hardware specifications used for each node in the Data Center DR Simulation Lab.

---

## Domain Controller

| Component | Specification |
|-----------|--------------|
| Processor | Intel Xeon E5620 2.4 GHz (dual core) |
| Network | 1 × Bridged Network |
| Memory | 8 GB RAM |
| Storage | 80 GB HDD |

---

## ESXi 6.5 Hosts (Sapcore1, Sapcore2, Sapcore3)

These three hosts form **Cluster SYS A** (production cluster) and contribute to vSAN storage.

| Component | Specification |
|-----------|--------------|
| Processor | Intel Xeon E5620 2.4 GHz (8 core) |
| Network | 1 × Bridged Network |
| Memory | 16 GB RAM |
| Storage — OS | 50 GB (ESXi boot disk) |
| Storage — Flash | 1 × 10 GB (vSAN Cache Tier) |
| Storage — Capacity | 1 × 25 GB (vSAN Capacity Tier) |
| I/O | USB 3.0, CD/DVD |

> **Note:** Each ESXi 6.5 host contributes 10 GB flash + 25 GB capacity to the shared vSAN datastore.

---

## ESXi 6.7 Host — DR Target (Sapcore4)

This host forms **Cluster SYS B** and serves as the replication target.

| Component | Specification |
|-----------|--------------|
| Processor | Intel Xeon E5620 2.4 GHz (8 core) |
| Network | 1 × Bridged Network |
| Memory | 32 GB RAM |
| Storage | 1 × 250 GB HDD |
| I/O | USB 3.0, CD/DVD |

---

## ESXi 6.7 Host — vCenter HA (Sapcore5)

This host runs the vCenter Server Appliance and the vCenter HA cluster.

| Component | Specification |
|-----------|--------------|
| Processor | Intel Xeon E5620 2.4 GHz (8 core) |
| Network | 1 × Bridged Network |
| Memory | 32 GB RAM |
| Storage | 300 GB HDD |
| I/O | USB 3.0, CD/DVD |

---

## vCenter Client Host (Windows Server 2012 R2)

The machine used to run the VCSA installer and join the domain.

| Component | Specification |
|-----------|--------------|
| Processor | Intel Xeon E5620 2.4 GHz (8 core) |
| Network | 1 × Bridged Network + 1 × NAT Network |
| Memory | 32 GB RAM |
| Storage | SCSI 100 GB |

---

## Backup Server (BACKUPSRV)

Windows Server acting as the FTP backup target for vCenter backups.

| Component | Specification |
|-----------|--------------|
| Processor | Intel Xeon E5620 2.4 GHz (4 core) |
| Network | 1 × Bridged Network |
| Memory | 4 GB RAM |
| Storage | 80 GB HDD |

---

## Summary Table

| Hostname | Role | CPU Cores | RAM | Primary Storage |
|----------|------|-----------|-----|----------------|
| vsystem (DC) | Domain Controller | 2 | 8 GB | 80 GB |
| Sapcore1 | ESXi 6.5 / SYS A | 8 | 16 GB | 50 GB + 35 GB |
| Sapcore2 | ESXi 6.5 / SYS A | 8 | 16 GB | 50 GB + 35 GB |
| Sapcore3 | ESXi 6.5 / SYS A | 8 | 16 GB | 50 GB + 35 GB |
| Sapcore4 | ESXi 6.7 / SYS B | 8 | 32 GB | 250 GB |
| Sapcore5 | ESXi 6.7 / VCSA HA | 8 | 32 GB | 300 GB |
| vCenter Client | VCSA Installer Host | 8 | 32 GB | 100 GB |
| BACKUPSRV | FTP Backup Server | 4 | 4 GB | 80 GB |

---

## Home Lab Notes

All nodes in this lab run as **virtual machines inside VMware Workstation**, nested on a single physical host. This approach allows the entire data center simulation to run on consumer-grade hardware while accurately replicating enterprise configurations.

**Physical host minimum recommended specs:**
- CPU: Intel Core i9 / Xeon (with VT-x and VT-d enabled in BIOS)
- RAM: 128 GB DDR4
- Storage: 2 TB NVMe SSD
- Network: Gigabit Ethernet
