# IP Configuration Apache CloudStack

This guide walks you through updating the network configuration for a new zone in Apache CloudStack. It includes steps for IP reservation, Pod setup, VLAN range configuration, Management Server setup, and Storage configuration.

## Table of Contents

- [IP Configuration Apache CloudStack](#ip-configuration-apache-cloudstack)
  - [Table of Contents](#table-of-contents)
  - [1. IP Reservation](#1-ip-reservation)
  - [2. Pod Configuration](#2-pod-configuration)
  - [3. VLAN/VNI Range](#3-vlanvni-range)
  - [4. Management Server](#4-management-server)
  - [5. Storage Configuration](#5-storage-configuration)
    - [Primary Storage](#primary-storage)
    - [Secondary Storage](#secondary-storage)

## 1. IP Reservation

Go to **Infrastructure → Zone → Network Configuration**, and update the reserved IP range:

| **Gateway**   | **Netmask**   | **VLAN/VNI**  | **Start IP**    | **End IP**      |
| ------------- | ------------- | ------------- | --------------- | --------------- |
| 192.168.160.1 | 255.255.254.0 | (leave blank) | 192.168.160.201 | 192.168.160.210 |

---

## 2. Pod Configuration

Create or modify the Pod with the following configuration:

- **Pod Name**: `Pod-New`
- **Reserved System Gateway**: `192.168.160.1`
- **Reserved System Netmask**: `255.255.254.0`
- **Start Reserved System IP**: `192.168.160.211`
- **End Reserved System IP**: `192.168.160.220`

Ensure that this reserved IP range does not conflict with the IPs allocated to virtual machines.

---

## 3. VLAN/VNI Range

Configure guest traffic VLANs:

- **VLAN/VNI Range**: `3300 - 3399`

> This range will be used to carry guest traffic across the physical network.

---

## 4. Management Server

When adding the host (management server):

- **Host Name**: `192.168.160.199`
- **Username**: `root`
- **Authentication Method**: `Password`
- **Password**: `AkiyamaMizuki1`

Ensure this host is accessible from the CloudStack management server and can communicate over the reserved IP range.

---

## 5. Storage Configuration

### Primary Storage

- **Name**: `Pristor-12`
- **Scope**: `Zone`
- **Protocol**: `NFS`
- **Server**: `192.168.160.199`
- **Path**: `/export/primary`
- **Provider**: `DefaultPrimary`

### Secondary Storage

- **Provider**: `NFS`
- **Name**: `SECSTOR-12`
- **Server**: `192.168.160.199`
- **Path**: `/export/secondary`

Ensure that both primary and secondary storage paths are properly exported and mounted on the management server and hosts.
