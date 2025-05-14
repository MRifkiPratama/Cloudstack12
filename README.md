# Apache Cloudstack Private Cloud Installation and Configuration
## Contributor
- Edgrant Henderson Suryajaya
- Miranti Anggunsari
- Muhammad Rifki Pratama
- Safia Amita Khoirunnisa

## Introduction
Apache CloudStack is an open-source cloud computing platform designed to deploy and manage large networks of virtual machines, providing IaaS (Infrastructure as a Service) through a user-friendly web interface and robust API support. The dependencies are
- KVM (Kernel-based Virtual Machine): A Linux kernel module that enables the hardware virtualization features of modern processors, allowing CloudStack to run virtual machines efficiently on host servers.
- MySQL: A popular open-source relational database management system used by CloudStack to store and manage configuration data, VM metadata, user information, and operational state.

## Environment Set Up
### Hardware Requirement
```
CPU: 4-core processor (Intel VT-x or AMD-V enabled)
RAM: Minimum 8 GB (16 GB recommended for smoother performance)
Storage: At least 100 GB free disk space (SSD preferred)
Network: 1 or more NICs (separate NICs recommended for management, public, and guest traffic)
Operating System: Ubuntu Server 22.04
```
### Network Address

```
Network Address:
Host IP Address:
Gateway:
Management IP:
System IP:
Public IP:
```
## Utils Set Up
### LazyVIm
```
sudo snap install nvim --classic
sudo apt install gcc git xclip
mkdir ~/.config/nvim
git clone https://github.com/LazyVim/starter ~/.config/nvim
```
### Other Tools
```
sudo apt-get install openntpd openssh-server sudohtop tar
sudo apt-get install intel-microcode
sudo passwd root

```
## Network Configuration
### Netplan
```
cd /etc/netplan
sudo -e /etc/netplan/01.
```
### Change the IP
```
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
      addresses: [192.168.1.200/24]  #Your host IP address
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [enp0s3]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
        forward-delay: 0
```
### Confirm Netplan
```
netplan Get
sudo netplan apply
```
### Install Hardware Resource Monitoring Tools
```
sudo su
apt update -y
apt upgrade -y
apt install htop lynx duf -y
apt install bridge-utils
```

