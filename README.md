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
### Install Network Servies and Text Editor
```
apt-get install openntpd openssh-server sudo vim tar -y
apt-get install intel-microcode -y
passwd teep1
```
### Enable SSH Root Login
```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
#restart ssh service
service ssh restart
#or
systemctl restart sshd.service
```
### Check the SSH Configuration
```
nano /etc/ssh/sshd_config
```
Find the PermitRootLogin and make sure to set it to 'yes'

## Cloudstack Installation
### Add CloudStack Repository and GPG Key
```
sudo -i
mkdir -p /etc/apt/keyrings
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list/
```

### Installing Cloudstack and Mysql Server
```
apt-get update -y
apt-get install cloudstack-management mysql-server
```

### Configure Mysql Config File
```
sudo -e /etc/mysql/mysql.conf.d/mysqld.cnf
```
```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

### Restart and check mysql service status
```
systemctl restart mysql
systemctl status mysql
```

### Deploy Database as Root and create user name and password
```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:teep1 -i 192.168.107.187
```
### Setup Primary Storage
```
sudo su
apt-get install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```
### Configure NFS Server
```
sudo su
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```

- Perintah sed pertama mengubah konfigurasi RPCMOUNTDOPTS pada file /etc/default/nfs-kernel-server agar rpc.mountd menggunakan port 892 dan mengelola grup ID dengan opsi --manage-gids.

- Perintah sed kedua mengatur STATDOPTS pada file /etc/default/nfs-common agar statd menggunakan port 662 untuk menerima koneksi dan port 2020 untuk koneksi keluar.

- Perintah echo "NEED_STATD=yes" menambahkan konfigurasi agar daemon statd diaktifkan saat layanan NFS berjalan.

- Perintah sed ketiga mengatur RPCRQUOTADOPTS pada file /etc/default/quota agar rpc.rquotad, yang menangani sistem kuota, menggunakan port 875.

- perintah service nfs-kernel-server restart digunakan untuk me-restart layanan NFS agar semua perubahan konfigurasi segera diterapkan.
