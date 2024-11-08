# Linux Library Infrastructure Project

This project sets up a Linux-based infrastructure for a local library that lacks the funding for Windows licenses. The infrastructure includes a server and a workstation configured with essential services and applications to demonstrate the functionality, security, and usability of Linux in a library setting.

---

## Project Overview

The library manager has opted to consider Linux as a cost-effective alternative, but some users are skeptical. This project is a demonstration that showcases Linux's capabilities through the following configuration:

1. **Server** (without GUI):

   - DHCP Service
   - DNS Service
   - Web Server (Nginx)
   - Scheduled configuration backups
   - SSH for remote access

2. **Workstation**:
   - Desktop environment
   - LibreOffice, Gimp, and Mullvad browser
   - Automatic IP addressing
   - Separate partition for `/home`

---

## Setup Requirements

### Prerequisites

- A virtualized environment, such as **VirtualBox** or **VMware**, with virtual machines for the server and workstation.
- Basic understanding of Linux commands and system administration.

---

## Installation Steps

### 1. Server Configuration

#### DHCP Server

- Install and configure `isc-dhcp-server` to serve IPs to an internal network.

#### DNS Server

- Configure `bind9` for local DNS resolution with external query forwarding.

#### Web Server

- Install and set up `nginx` to host a simple local webpage.

#### Backup and SSH

- Automate weekly backups of configuration files using `rsync` and `cron`.
- Configure SSH for secure remote access.

### 2. Workstation Configuration

#### Desktop Environment and Applications

- Install a graphical environment along with **LibreOffice**, **Gimp**, and **Mullvad browser** for user applications.

#### Network and Partitioning

- Enable DHCP for automatic IP addressing.
- Configure `/home` on a separate disk partition for better data management.

---

## Usage Instructions

1. **Starting the Services**: Ensure each service (DHCP, DNS, Nginx) is active on the server.
2. **Accessing the Web Server**: Visit the IP of the server from a browser on the workstation to view the demo webpage.
3. **Remote Access**: Use SSH to connect to the server for remote management and monitoring.

---

## Security Considerations

- Firewall setup (using `ufw`) to allow only essential services (e.g., SSH, HTTP).
- SSH security hardening with customized port settings and key-based authentication.

---

## Files and Directories

- `/etc/dhcp/dhcpd.conf`: DHCP configuration
- `/etc/bind/`: DNS configuration
- `/var/www/html/`: Nginx web directory for demo content
- `/usr/local/bin/backup_configs.sh`: Backup script
