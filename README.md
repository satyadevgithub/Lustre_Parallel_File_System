# Lustre_Parallel_File_System
High-Performance Computing Cluster with Parallel File System
# Lustre Parallel File System on RHEL 8

## Overview
This repository provides a step-by-step guide to setting up a **Lustre parallel file system** on **RHEL 8**, including **MGS (Management Server), MDS (Metadata Server), MDT (Metadata Target), OSS (Object Storage Server), and OST (Object Storage Target)**.

## Prerequisites
- **RHEL 8 installed** on all server nodes
- At least one node acting as **MGS, MDS, and MDT**
- Storage devices configured for **OSS and OST**

## Kernel Installation for Lustre
Lustre requires a specific kernel version. Install it using:

```bash
sudo dnf install kernel-4.18.0-553.5.1.el8_10.x86_64
sudo reboot
uname -r
sudo grubby --info=ALL | grep -i kernel
sudo grubby --set-default /boot/vmlinuz-4.18.0-553.5.1.el8_10.x86_64
grubby --default-kernel
```

## Installing Lustre Server
### Install Required Packages
Lustre is not available in default RHEL repositories. Download and install the required RPMs:

```bash
wget https://downloads.whamcloud.com/public/lustre/lustre-2.15.5/el8.10/server/RPMS/x86_64/*.rpm
wget https://downloads.whamcloud.com/public/e2fsprogs/1.47.1.wc1/el8/RPMS/x86_64/*.rpm

sudo dnf install kernel-headers-$(uname -r)
sudo dnf install -y epel-release
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y wget make gcc kernel-devel
sudo dnf install *.rpm
```

### Set Default Kernel for Boot
```bash
sudo grubby --set-default /boot/vmlinuz-4.18.0-553.5.1.el8_lustre.x86_64
grubby --default-kernel
```

## Configuring Lustre Metadata Target (MDT)
### Format MDT Storage
```bash
mkfs.lustre --mdt --fsname=lustre --mgs --index=0 /dev/sdb
```

### Mount the MDT
```bash
mkdir -p /mnt/mdt
echo "/dev/sdb /mnt/mdt lustre defaults 0 0" >> /etc/fstab
mount -t lustre /dev/sdb /mnt/mdt
```

### Enable Lustre Services
```bash
sudo systemctl daemon-reload
sudo systemctl stop firewalld
sudo systemctl disable firewalld
systemctl enable lustre
systemctl start lustre
```

### Verify Lustre Service
```bash
lctl list_nids
sudo lctl dl
```

## Configure OSS and OST
### Format OST Storage
```bash
mkfs.lustre --ost --fsname=lustre --mgsnode=<MGS_IP>@tcp --index=1 /dev/sdc
```

### Mount the OST
```bash
mkdir -p /mnt/ost
echo "/dev/sdc /mnt/ost lustre defaults 0 0" >> /etc/fstab
mount -t lustre /dev/sdc /mnt/ost
```

## Lustre Client Setup
### Install Lustre Client Packages
Download and install client RPMs:
```bash
wget https://downloads.whamcloud.com/public/lustre/lustre-2.15.5/el8.10/client/RPMS/x86_64/*.rpm
sudo dnf install *.rpm
```

### Mount Lustre File System
```bash
mkdir -p /mnt/lustre
echo "<MGS_IP>@tcp:/lustre /mnt/lustre lustre defaults,_netdev 0 0" >> /etc/fstab
mount -t lustre <MGS_IP>@tcp:/lustre /mnt/lustre
```

### Verify Lustre Mount
```bash
df -h /mnt/lustre
lctl dl
lfs df
```

## Testing Lustre File System
```bash
touch /mnt/lustre/testfile
ls -l /mnt/lustre/
```

## License
This guide is open-source. Feel free to contribute or report issues.
