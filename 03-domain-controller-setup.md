# Domain Controller Setup — CentOS 7 + Samba Active Directory

This guide covers the installation and configuration of an Active Directory domain controller using **CentOS 7** and **Samba 4.6.0**.

---

## Overview

| Setting | Value |
|---------|-------|
| OS | CentOS 7 |
| IP Address | 192.168.1.10 |
| Hostname (FQDN) | vsystem.helpy.com |
| Domain | helpy.com |
| AD Software | Samba 4.6.0 |
| Network Type | Bridged |

---

## Prerequisites

- CentOS 7 ISO downloaded and ready
- VMware Workstation with 8 GB RAM, 80 GB disk configured for the VM
- Internet access for package downloads
- Static IP address assigned

---

## Step 1 — Install CentOS 7

1. Boot the VM from the CentOS 7 ISO.
2. Select **Install CentOS 7** from the boot menu.
3. Configure the static IP address during installation:
   - IP: `192.168.1.10`
   - Netmask: `255.255.255.0`
   - Gateway: `192.168.1.1`
   - DNS: `127.0.0.1` (self-hosted after Samba is configured)
4. Complete the installation and reboot.

---

## Step 2 — Disable SELinux

SELinux must be disabled for Samba to function correctly as a domain controller.

```bash
# Check current status
sestatus

# Edit SELinux configuration
vi /etc/selinux/config
```

Change the following line:
```
SELINUX=enforcing
```
To:
```
SELINUX=disabled
```

Reboot the system for the change to take effect:
```bash
reboot
```

---

## Step 3 — Configure Hostname

```bash
# Set the hostname
hostnamectl set-hostname vsystem.helpy.com

# Edit /etc/hosts
vi /etc/hosts
```

Add the following entry:
```
192.168.1.10    vsystem.helpy.com    vsystem
```

---

## Step 4 — Install Required Repositories

```bash
# Install EPEL repository
yum install epel-release -y

# Install build dependencies for Samba
yum install -y \
  gcc \
  make \
  python-devel \
  gnutls-devel \
  libacl-devel \
  openldap-devel \
  pam-devel \
  cups-devel \
  libgpg-error-devel \
  krb5-workstation \
  krb5-libs
```

---

## Step 5 — Download and Install Samba 4.6.0

```bash
# Create working directory
mkdir samba_os
cd samba_os

# Download Samba source
wget https://download.samba.org/pub/samba/stable/samba-4.6.0.tar.gz

# Extract the archive
tar zxvf samba-4.6.0.tar.gz

# Enter the Samba directory
cd samba-4.6.0

# Configure the build
./configure \
  --prefix=/usr \
  --localstatedir=/var \
  --with-configdir=/etc/samba \
  --libdir=/usr/lib64 \
  --with-modulesdir=/usr/lib64/samba \
  --with-pammodulesdir=/lib64/security \
  --with-lockdir=/var/lib/samba \
  --with-logfilebase=/var/log/samba \
  --with-piddir=/run/samba \
  --with-privatedir=/etc/samba \
  --enable-cups \
  --with-acl-support \
  --with-ads \
  --with-automount \
  --enable-fhs \
  --with-pam \
  --with-quotas \
  --with-shared-modules=idmap_rid,idmap_ad,idmap_hash,idmap_adex \
  --with-syslog \
  --with-utmp \
  --with-dnsupdate

# Compile (this takes several minutes)
make

# Install
make install
```

---

## Step 6 — Create Samba systemd Service

Create the systemd service file:
```bash
vi /usr/lib/systemd/system/samba.service
```

Add the following content:
```ini
[Unit]
Description=Samba AD Daemon
Wants=network-online.target
After=network.target network-online.target rsyslog.service

[Service]
Type=forking
PIDFile=/run/samba/samba.pid
LimitNOFILE=16384
ExecStart=/usr/sbin/samba --daemon
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

Create the tmpfiles config:
```bash
vi /etc/tmpfiles.d/samba.conf
```

Add:
```
d /var/run/samba 0755 root root -
```

---

## Step 7 — Provision the Domain Controller

```bash
# Back up the existing krb5.conf
mv /etc/krb5.conf /etc/krb5.conf.org

# Provision the Samba AD domain
samba-tool domain provision \
  --use-rfc2307 \
  --interactive
```

When prompted:
- **Realm:** `HELPY.COM`
- **Domain:** `HELPY`
- **Server Role:** `dc`
- **DNS backend:** `SAMBA_INTERNAL`
- **Administrator password:** *(set a strong password)*

---

## Step 8 — Configure Firewall

Open all required ports for Active Directory services:

```bash
# Add AD-related services
firewall-cmd --add-service=dns --permanent
firewall-cmd --add-service=kerberos --permanent
firewall-cmd --add-service=kpasswd --permanent
firewall-cmd --add-service=ldap --permanent
firewall-cmd --add-service=ldaps --permanent
firewall-cmd --add-service=samba --permanent

# Add required TCP/UDP ports
firewall-cmd --add-port=135/tcp --permanent
firewall-cmd --add-port=137-138/udp --permanent
firewall-cmd --add-port=139/tcp --permanent
firewall-cmd --add-port=3268-3269/tcp --permanent
firewall-cmd --add-port=49152-65535/tcp --permanent

# Reload firewall
firewall-cmd --reload
```

---

## Step 9 — Enable and Start Samba

```bash
# Enable the Samba service
systemctl enable samba

# Start the service
systemctl start samba

# Verify status
systemctl status samba
```

---

## Verification

```bash
# Verify AD domain is working
samba-tool domain info 192.168.1.10

# Test DNS resolution
host -t SRV _ldap._tcp.helpy.com

# List domain users
samba-tool user list
```

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Samba fails to start | SELinux still enabled | Verify `sestatus` shows `disabled`, reboot |
| DNS resolution fails | Samba DNS not started | Check `systemctl status samba`, look for DNS-related errors in `/var/log/samba/` |
| Cannot join domain | Firewall blocking | Verify all ports are open with `firewall-cmd --list-all` |
| krb5.conf errors | Old config not backed up | Ensure `/etc/krb5.conf` is the one generated by Samba |
