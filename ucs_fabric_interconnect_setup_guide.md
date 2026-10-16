# Cisco UCS Fabric Interconnect 6200 Series - Initial Cluster Setup Guide

This technical guide provides step-by-step procedures for initializing and configuring a high-availability (HA) cluster pair of **Cisco UCS Fabric Interconnects 6248UP** using serial console access.

---

## 1. Prerequisites & Access Requirements

### Hardware Requirements
* **Fabric Interconnects:** 2x Cisco UCS 6248UP Fabric Interconnects (FI-A and FI-B)
* **Cluster Cabling:** 2x RJ45 Ethernet cables for cross-connecting L1 and L2 high-availability ports
* **Management Uplinks:** 2x Ethernet patch cables connecting `mgmt0` ports to the OOB Management Switch
* **Console Access:** 1x Serial-to-RJ45 console cable with USB-to-Serial adapter (FTDI recommended)

### Serial Console Settings
Use a terminal emulator (e.g., PuTTY) with the following serial connection parameters:
* **Baud Rate:** 9600
* **Data Bits:** 8
* **Parity:** None
* **Stop Bits:** 1
* **Flow Control:** None

---

## 2. Physical Interconnect & Cluster Wiring

Before powering on the Fabric Interconnects, verify the physical connectivity between FI-A and FI-B:

| Local Device / Port | Remote Device / Port | Function | Cable Type |
| :--- | :--- | :--- | :--- |
| **FI-A Port L1** | **FI-B Port L2** | Cluster Synchronization Link 1 | Ethernet Cat5e/Cat6 |
| **FI-A Port L2** | **FI-B Port L1** | Cluster Synchronization Link 2 | Ethernet Cat5e/Cat6 |
| **FI-A Port mgmt0** | **SW-MGMT-01 Port 38** | Primary Management Interface | Ethernet Cat5e/Cat6 |
| **FI-B Port mgmt0** | **SW-MGMT-02 Port 38** | Secondary Management Interface | Ethernet Cat5e/Cat6 |

---

## 3. Initializing Primary Fabric Interconnect (FI-A)

1. Connect the serial console cable to the console port of **FI-A** and power on the unit.
2. At the boot prompt, enter `console` when asked for the configuration method:

```text
Enter the configuration method. (console/gui) [console]: console
Enter the setup mode; setup newly or restore from backup. (setup/restore) ? setup

You have chosen to setup a new Fabric interconnect. Continue? (y/n) [y]: y
Enforce strong password? (y/n) [n]: n
Enter the password for "admin": ********
Confirm the password for "admin": ********

Is this Fabric interconnect part of a cluster(select 'no' for standalone)? (yes/no) [y]: yes
Enter the switch fabric (A/B) []: A
Enter the system name: FI-CLUSTER-01

Physical Switch Mgmt0 IP address: 10.0.10.11
Physical Switch Mgmt0 IPv4 netmask: 255.255.255.0
IPv4 address of the default gateway: 10.0.10.1
Cluster IPv4 address: 10.0.10.10

Configure the DNS Server IP address? (yes/no) [n]: y
DNS IP address: 10.0.1.53
Configure the default domain name? (yes/no) [n]: y
Default domain name: enterprise.local
Join centralized management environment (UCS Central)? (yes/no) [n]: n
```

3. Review the applied configuration summary:

```text
Following configurations will be applied:
  Switch Fabric                 = A
  System Name                   = FI-CLUSTER-01
  Physical Switch Mgmt0 IP      = 10.0.10.11
  Physical Switch Mgmt0 Netmask = 255.255.255.0
  Default Gateway               = 10.0.10.1
  DNS Server                    = 10.0.1.53
  Domain Name                   = enterprise.local
  Cluster Enabled               = yes
  Cluster IP Address            = 10.0.10.10

Apply and save the configuration (select 'no' if you want to re-enter)? (yes/no): yes
Applying configuration. Please wait.
```

---

## 4. Initializing Secondary Fabric Interconnect (FI-B)

1. Move the serial console cable to the console port of **FI-B** and power on the unit.
2. Enter `console` when prompted. The installer will automatically detect the active peer (FI-A):

```text
Enter the configuration method. (console/gui) [console]: console

Installer has detected the presence of a peer Fabric interconnect. This Fabric interconnect will be added to the cluster. Continue (y/n) ? y
Enter the admin password of the peer Fabric interconnect: ********

Connecting to peer Fabric interconnect... done
Retrieving config from peer Fabric interconnect... done

Peer Fabric interconnect Mgmt0 IPv4 Address: 10.0.10.11
Peer Fabric interconnect Mgmt0 IPv4 Netmask: 255.255.255.0
Cluster IPv4 address: 10.0.10.10

Peer FI is IPv4 Cluster enabled. Please Provide Local Fabric Interconnect Mgmt0 IPv4 Address
Physical Switch Mgmt0 IP address: 10.0.10.12

Apply and save the configuration (select 'no' if you want to re-enter)? (yes/no): yes
Applying configuration. Please wait.
Configuration file - Ok
```

---

## 5. Cluster Verification & Health Check

Log into FI-A via SSH or Console and run the following CLI commands to verify cluster status:

```text
FI-CLUSTER-01-A# show cluster status
Cluster Id: 0x3a8f2190b41111e3-0x9212000573d122f0

A: UP, PRIMARY
B: UP, SUBORDINATE

HA state: READY
```

Verify extended cluster link states:

```text
FI-CLUSTER-01-A# show cluster extended-state
Start time: Sat Oct 17 06:00:00 2026
Management Client status: HA READY
Subordinate HA state: Subordinate SW version good
L1 Status: UP
L2 Status: UP
```

### Verification Checklist
- [x] Both Fabric Interconnects are **UP**.
- [x] High Availability (HA) state reports **READY**.
- [x] Interconnect Links L1 and L2 report **UP**.
- [x] Cluster Virtual IP (`10.0.10.10`) is reachable and launches Cisco UCS Manager (UCSM).