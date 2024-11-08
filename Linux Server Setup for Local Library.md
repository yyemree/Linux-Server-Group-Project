# Linux Server Setup for Local Library

This document outlines the detailed steps to set up a Linux Ubuntu server and workstation using virtual machine (Vbox) to demonstrate the functionality and security of Linux infrastructure to library users.

---

## Project Overview

The local library does not have funds for Windows licenses, so the manager is considering switching to Linux. Some users are skeptical and request a demo. You will set up a server and a workstation in a virtual environment using internal virtual networking (ensuring your DHCP does not conflict with the LAN).

---

## Setup Components

1. **Server** (Without GUI):

   - DHCP Service
   - DNS Service
   - Web Server (Nginx)
   - Weekly backup of configuration files in a compressed archive
   - Remote access (SSH)

2. **Workstation**:
   - Desktop environment
   - LibreOffice, Gimp, and Mullvad browser
   - Automatic addressing
   - `/home` on a separate partition

---

### Prerequisites

- Install virtual machines.
- Ensure that VMs are set to use an **internal network** to prevent conflicts with LAN.

---

## Server Configuration

### 1. Install Base Packages

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. DHCP Server Setup

#### Install DHCP Server

```bash
sudo apt install isc-dhcp-server -y
```

#### Configure DHCP

Edit the DHCP configuration file:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Add the following configuration for an internal network:

```plaintext
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;

subnet 10.5.5.0 netmask 255.255.255.224 {
  range 10.5.5.26 10.5.5.30;
  option domain-name-servers ns1.internal.example.org;
  option domain-name "internal.example.org";
  option subnet-mask 255.255.255.224;
  option routers 10.5.5.1;
  option broadcast-address 10.5.5.31;
  default-lease-time 600;
  max-lease-time 7200;
}
INTERFACESv4="enp0s8";
```

Now it has been updated the INTERFACESv4 to specify the internal interface

(INTERFACESv4="enp0s8"):

Start and enable the DHCP service:

```bash
sudo systemctl start isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

### 3. DNS Server Setup

#### Install Bind9

```bash
sudo apt install bind9 -y
```

#### Set Up a Local DNS Zone

Edit the Bind configuration to add a local DNS zone:

```bash
sudo nano /etc/bind/named.conf.local
```

Add a configuration like this:

```plaintext
zone "library.internal" {
    type master;
    file "/etc/bind/db.library.internal";
};
```

Create the zone file:

```bash
# Copy the zone file
sudo cp /etc/bind/db.local /etc/bind/db.local-library
# Rename the copied file to db.library.internal
sudo mv /etc/bind/db.local-library /etc/bind/db.library.internal
sudo nano /etc/bind/db.library.internal
```

It has been edited the contents to match internal domain. `localhost` entries to server hostname or IP is configured.

```plaintext
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     library.internal. root.library.internal. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.library.internal.
@       IN      A       10.5.5.1
ns      IN      A       10.5.5.1
```

Restart Bind9:

```bash
sudo systemctl restart bind9
```

### 4. Web Server (Nginx) Setup

#### Install Nginx

```bash
sudo apt install nginx -y
```

#### Configure Nginx

Place a sample webpage in the Nginx root directory:

```bash
sudo su - root
cd /var/www/html
ls
```

Write the html for library's website

```
sudo nano index.nginx-debian.html
```

Configure html file for library's website

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Library Website</title>
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

### 5. Backups

#### Install `rsync` and Set Up Cron Jobs

```bash
sudo apt install rsync -y
```

Create a backup script for configurations:

```bash
sudo su -
sudo nano backup.sh
```

Add the following content:

```bash
#!/bin/bash
#Mounts the /dev/sdb1 partition to /mnt/yedek/
mount -t ext4 /dev/sdb1 /mnt/yedek/
#Copies the /root/backup.zip file to /mnt/yedek/ using rsync
rsync /root/backup.zip /mnt/yedek/
#unmount the disk
umount /mnt/yedek/
```

Make the script executable:

```bash
chmod +x backup.sh
```

Edit the `crontab` to run weekly:

```bash
sudo crontab -e
```

Add the following line to schedule the script every Sunday at 2 AM:

```plaintext
59 23 * * 1  zip /root/backup.zip /etc/dhcp/dhcpd.conf /etc/bind/named.conf.local /etc/bind/db.library.internal /etc/nginx/sites-enabled/default
59 23 * * 2  /root/backup.sh
```

### 6. SSH Setup

Install and configure SSH for remote management:

```bash
sudo apt install openssh-server -y
```

---

## Workstation Configuration

### 1. Install Desktop Environment and Applications

```bash
sudo apt update
sudo apt install ubuntu-desktop libreoffice gimp mullvad-browser -y
```

### 2. Enable Automatic IP Addressing

Ensure the workstation is set to use DHCP by editing the network configuration:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Add or update with:

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
```

Apply the netplan configuration:

```bash
sudo netplan apply
```

### 3. Create a Separate Partition for `/home`

Identify the disk and create a new partition using `fdisk` or `parted`. Then format and mount it:

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkdir /mnt/home
sudo mount /dev/sdb1 /mnt/home
```

Update `/etc/fstab` to mount the partition at `/home`:

---

## Final Steps

- **Firewall**: Enable `ufw` firewall and allow only essential services (SSH, HTTP).
- **Testing**: Verify each service functionality (DHCP IP assignment, DNS resolution, web server access).
- **Documentation**: Document configurations and security practices to present the infrastructure's reliability.

---

This setup offers a secure, Linux-based solution for the library, allowing users to experience and understand Linux’s capabilities within a virtualized network.
