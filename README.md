# Single Node Apache Cloudstack Private Cloud Installation Guide

![image](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)

## Table of Contents

- [Single Node Apache Cloudstack Private Cloud Installation Guide](#single-node-apache-cloudstack-private-cloud-installation-guide)
  - [Table of Contents](#table-of-contents)
  - [Contributor](#contributor)
  - [Disclaimer](#disclaimer)
  - [Introduction](#introduction)
  - [Environment Set Up](#environment-set-up)
    - [Hardware Requirement](#hardware-requirement)
    - [Our Network Address](#our-network-address)
    - [Installing Nessessary Tools](#installing-nessessary-tools)
  - [Utils Set Up](#utils-set-up)
  - [Network Configuration](#network-configuration)
    - [Netplan](#netplan)
    - [Change the IP](#change-the-ip)
    - [Confirm Netplan](#confirm-netplan)
  - [Cloudstack Installation](#cloudstack-installation)
    - [Add CloudStack Repository and GPG Key](#add-cloudstack-repository-and-gpg-key)
    - [Installing Cloudstack and Mysql Server](#installing-cloudstack-and-mysql-server)
    - [Configure Mysql Config File](#configure-mysql-config-file)
    - [Restart and check mysql service status](#restart-and-check-mysql-service-status)
    - [Deploy Database as Root and create user name and password](#deploy-database-as-root-and-create-user-name-and-password)
  - [Configure Cloudstack Host with KVM Hypervisor](#configure-cloudstack-host-with-kvm-hypervisor)
    - [Install KVM and Cloudstack Agent](#install-kvm-and-cloudstack-agent)
    - [Configure KVM Virtualization Management](#configure-kvm-virtualization-management)
    - [Libvirt TCP Configuration](#libvirt-tcp-configuration)
    - [Restart libvirtd](#restart-libvirtd)
    - [Configuration to Support Docker and Other Services](#configuration-to-support-docker-and-other-services)
    - [Generate Unique Host ID](#generate-unique-host-id)
    - [Restart libvirtd (after updating UUID)](#restart-libvirtd-after-updating-uuid)
  - [Configure Iptables Firewall](#configure-iptables-firewall)
  - [Disable AppArmor for libvirtd](#disable-apparmor-for-libvirtd)
  - [Start CloudStack Management Server](#start-cloudstack-management-server)


## Contributor

- [Edgrant Henderson Suryajaya](https://github.com/EdgrantHS)
- [Miranti Anggunsari](https://www.github.com/rantiaaa)
- [Muhammad Rifki Pratama](https://github.com/MRifkiPratama)
- [Safia Amita Khoirunnisa](https://github.com/mitasafia)

## Disclaimer

> [!IMPORTANT]
> Our project is intended for Cloudstack installation and configuration on a single node (the host, management server, hypervisor, and storage) in a single machine. Cloudstack can also be set up in a multi-node environment, for that you can refer to other documentation.

## Introduction

Apache CloudStack is an open-source cloud computing platform for deploying and managing large networks of virtual machines. It provides Infrastructure as a Service (IaaS) via a web interface and API.

## Environment Set Up

### Hardware Requirement

```text
CPU: 4-core processor (Intel VT-x or AMD-V enabled)
RAM: Minimum 8 GB (16 GB recommended for smoother performance)
Storage: At least 100 GB free disk space (SSD preferred)
Network: 1 or more NICs (separate NICs recommended for management, public, and guest traffic)
Operating System: Ubuntu Server 22.04
```

### Our Network Address

The following configuration is based on our internal setup. Please adjust according to your own network environment.

```text
Network Address: 192.168.1.0/24
Host IP Address: 192.168.1.220
Gateway: 192.168.1.1
Management IP: 192.168.1.220
System IP: 192.168.1.221 - 192.168.1.225
Public IP: 192.168.1.226 - 192.168.1.230
```

### Installing Nessessary Tools

Update the system and install necessary tools, this might take a while.

```bash
sudo su
apt update -y
apt upgrade -y
apt install bridge-utils
```

## Utils Set Up

This step is optional, it helps maintain formatting but is not required to complete the installation.

The tools that we use are:

- **SSH:** Secure Shell, for remote access to the machine
- **Tailscale:** A VPN service that allows you to connect to your machine from anywhere
- **Lazyvim:** A Text Editor, more specifically a user friendly Neovim configuration

**<center>[Click for Detailed Utils Explanation](/details/01_utils.md)</center>**

## Network Configuration

The netplan configuration is similar to the network/wifi setting in Ubuntu Desktop/Windows, but we edit it using a file. 

**<center>[Click for Detailed Network Explanation](/details/00_netplan.md)</center>**

### Netplan

```bash
cd /etc/netplan
sudo -e /etc/netplan/01-static-netcfg.yaml
```

### Change the IP

```yaml
# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: networkd

  ethernets:
    enp0s3:
      dhcp4: false
      dhcp6: false
      optional: true

  bridges:
    cloudbr0:
      interfaces: 
        - enp0s3
      addresses: 
        - 192.168.1.220/24 #Your host IP address
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: 
          - 8.8.8.8
          - 8.8.4.4

      dhcp4: false
      dhcp6: false

      parameters:
        stp: false
        forward-delay: 0
```

### Confirm Netplan

```bash
sudo netplan get
sudo netplan apply
```

## Cloudstack Installation

Installing Apache CloudStack Management Server, database, and NFS configuration to prepare your cloud environment.

**<center>[Click for Detailed Installation Explanation](/details/03_cloudstack_installation.md)</center>**

### Installing Cloudstack and Mysql Server

Update package index and install the CloudStack Management Server and MySQL.

```bash
apt-get update -y
apt-get install cloudstack-management mysql-server
```

### Configure Mysql Config File

```bash
sudo -e/etc/mysql/mysql.conf.d/mysqld.cnf
```

```bash
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

### Restart and check mysql service status

Restart MySQL to apply changes and verify it's running.

```bash
systemctl restart mysql
systemctl status mysql
```

### Deploy Database as Root and create user name and password

Deploy the CloudStack database using root credentials and set up the default user.

```bash
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:teep1 -i 192.168.1.220
```

## Configure Cloudstack Host with KVM Hypervisor

Installing and configuring a CloudStack host with KVM hypervisor, libvirt TCP access, unique host ID, and network settings to enable virtualization and agent communication.

**<center>[Click for Detailed Installation Explanation](/details/04_cloudstack_host.md)</center>**

### Install KVM and Cloudstack Agent

This command installs the QEMU-KVM hypervisor and the CloudStack agent, which enables the host to connect and communicate with the CloudStack management server.

```bash
apt-get install qemu-kvm cloudstack-agent -y
```

### Configure KVM Virtualization Management

This modifies the `libvirtd` default configuration to enable the daemon to listen for remote connections by setting `LIBVIRTD_ARGS="--listen"`.

```bash
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

### Libvirt TCP Configuration

These commands configure `libvirtd` to allow TCP connections without authentication. TLS is disabled, TCP is enabled on port 16509 (the default libvirt port), mDNS advertisement is turned off, and no authentication is required for TCP connections.

```bash
echo 'listen_tls = 0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp = 1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

### Restart libvirtd

This masks the default libvirt sockets that are not needed and restarts the `libvirtd` service to apply the changes.

```bash
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

### Configuration to Support Docker and Other Services

These kernel parameters are adjusted to prevent issues with Docker and other services by disabling bridge network calls to `iptables` and `arptables`.

```bash
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
sysctl -p
```

### Generate Unique Host ID

This installs the `uuid` package, generates a unique UUID for the host, and appends it to the `libvirtd` configuration to ensure each host has a distinct identifier.

```bash
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = "\"$UUID\"" >> /etc/libvirt/libvirtd.conf
```

### Restart libvirtd (after updating UUID)

The `libvirtd` service will be restarted again to apply the UUID configuration update.

```bash
systemctl restart libvirtd
```

## Configure Iptables Firewall

To enable proper communication between virtualization services, add the following rules for your local network (adjust `NETWORK` as needed):

```bash
NETWORK=192.168.1.0/24
```

Edit your persistent iptables rules:

```bash
sudo -e /etc/iptables/rules.v4
```

Append these rules:

```bash
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT
```

Make the rules persistent:

```bash
sudo apt-get install iptables-persistent
```

> When prompted, answer **Yes** to save current rules.
---

### Disable AppArmor for libvirtd

Some versions of libvirt may require AppArmor to be disabled to work properly:

```bash
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

---

## Start CloudStack Management Server

```bash
cloudstack-setup-management
systemctl status cloudstack-management
tail -f /var/log/cloudstack/management/management-server.log
```

![Apache Cloudstack Website](images/apache-cloudstack.png)

## Reconfigure Apache Cloudstack with new IP
To update the CloudStack environment with a new IP configuration, reconfigure the following components: IP Reserve, Pod, VLAN, the Management Server, and the Storage Configuration. Detailed instructions are provided in the accompanying [configuration guide.](details/02_ipconfig.md)
