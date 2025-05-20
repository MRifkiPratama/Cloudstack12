# Single Node Apache Cloudstack Private Cloud Installation Guide

![image](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)

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

```
CPU: 4-core processor (Intel VT-x or AMD-V enabled)
RAM: Minimum 8 GB (16 GB recommended for smoother performance)
Storage: At least 100 GB free disk space (SSD preferred)
Network: 1 or more NICs (separate NICs recommended for management, public, and guest traffic)
Operating System: Ubuntu Server 22.04
```

### Our Network Address

> [!IMPORTANT]
> The following configuration is based on our internal setup. Please adjust according to your own network environment.

```
Network Address: 192.168.1.0/24
Host IP Address: 192.168.1.220
Gateway: 192.168.1.200
Management IP:
System IP:
Public IP:
```

## Utils Set Up
> [!IMPORTANT]
>  This tool is optional, it helps maintain formatting but is not required to complete the installation.


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

> [!WARNING]
> Utils kurang setup default editor untuk `sudo -e`  
> \- Edgrant

## Network Configuration

The netplan configuration is similar to the network/wifi setting in Ubuntu Desktop/Windows, but we edit it using a file. More information available [here](step1-CLI/00_netplan.md)

### Netplan

```bash
cd /etc/netplan
sudo -e /etc/netplan/01.
```
### Change the IP

> [!WARNING]
> Netplan revisi, ganti dengan yang terbaru, lalu code block tolong ditulis pakai bahasanya, contohnya ini harusnya yaml, cli harusnya bash (harusnya dikaish tau di markdown linter)  
> \- Edgrant

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

```bash
sudo netplan get
sudo netplan apply
```

> [!WARNING]
> 4 Heading ke bawah bukan termasuk network, harusnya install tools masuk di utils, ssh buat di heading baru  
> \- Edgrant

### Install Hardware Resource Monitoring Tools

```bash
sudo su
apt update -y
apt upgrade -y
apt install htop lynx duf -y
apt install bridge-utils
```

### Install Network Servies and Text Editor

```bash
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

## Configure Cloudstack Host with KVM Hypervisor
### Install KVM and Cloudstack Agent
```
apt-get install qemu-kvm cloudstack-agent -y
```
- Command `qemu-kvm` digunakan untuk menginstal virtualizer QEMU dan modul KVM untuk virtualisasi berbasis hardware.
- Command `cloudstack-agent` digunakan untuk menginstal agent yang digunakan untuk menghubungkan host ke CloudStack.

### Configure KVM Virtualization Management
```
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```
Command `sed` digunakan untuk memanipulasi file `/etc/default/libvirtd`, di mana akan dilakukan subtitusi baris `LIBVIRTD_ARGS=` menjadi `LIBVIRTD_ARGS="--listen"` yang akan mengaktifkan mode listening.

### Add some lines
```
echo 'listen_tls = 0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp = 1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```
Serangkaian baris command di atas digunakan untuk mengkonfigurasi `libvirtd` agar dapat menerima koneksi TCP dari host lain (remote management).
- `listen_tls = 0` digunakan untuk menonaktifkan mode TLS.
- `listen_tcp = 1` digunakan untuk mengaktifkan mode TCP.
- `tcp_port = "16509"` digunakan untuk menentukan port TCP yang akan digunakan, di mana port 16509 adalah port default untuk libvirtd.
- `mdns_adv = 0` digunakan untuk menonaktifkan mDNS advertising.
- `auth_tcp = "none"` digunakan untuk menonaktifkan autentikasi TCP (mengizinkan koneksi TCP tanpa autentikasi).

### Restart libvirtd
```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```
Pada tahap ini, digunakan command `systemctl mask` yang berfungsi untuk menonaktifkan socket-socket default yang tidak diperlukan oleh `libvirtd`. Kemudian, command `systemctl restart libvirtd` digunakan untuk merestart `libvirtd` agar konfigurasi baru dapat diterapkan dalam sistem.

### Configuration to Support Docker and Other Services
```
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
sysctl -p
```
Command `sysctl` digunakan untuk mengatur parameter kernel. Di mana serangkaian baris command di atas digunakan untuk menonaktifkan `bridge-nf-call-iptables` dan `bridge-nf-call-arptables` agar Docker dapat berjalan dengan baik.

### Generate Unique Host ID
```
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = "\"$UUID\"" >> /etc/libvirt/libvirtd.conf
```
Command `apt-get install uuid -y` digunakan untuk menginstal `uuid` yang digunakan untuk menghasilkan UUID unik. Kemudian, command `UUID=$(uuid)` digunakan untuk menghasilkan UUID unik dan menyimpannya ke dalam variabel `UUID`. Terakhir, command `echo host_uuid = "\"$UUID\"" >> /etc/libvirt/libvirtd.conf` digunakan untuk menambahkan UUID ke dalam file konfigurasi `libvirtd` sebagai identitas unik host, yang berarti setiap host akan memiliki UUID yang berbeda.

### Restart libvirtd (setelah update UUID)
```
systemctl restart libvirtd
```
Setelah mengupdate UUID, digunakan command `systemctl restart libvirtd` untuk merestart `libvirtd` agar konfigurasi baru dapat diterapkan ke dalam sistem.

## Configure Iptables Firewall
To enable proper communication between virtualization services, add the following rules for your local network (adjust `NETWORK` as needed):
```bash
NETWORK=192.168.106.0/23
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

## Disable AppArmor for libvirtd 
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
