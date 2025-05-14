# Apache Cloudstack Private Cloud Installation and Configuration
![image](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)
## Contributor
- [Edgrant Henderson Suryajaya](https://github.com/EdgrantHS)
- [Miranti Anggunsari](https://www.github.com/rantiaaa)
- [Muhammad Rifki Pratama](https://github.com/MRifkiPratama)
- [Safia Amita Khoirunnisa](https://github.com/mitasafia)

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
xxx
