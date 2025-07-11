## Enterprise-Grade RDP Infrastructure on ESXi with Gateway and AD Services

### Objective
Provide secure remote desktop access to internal servers (RDS01, RDS02) using RD Gateway, while keeping all backend systems isolated in a private internal network via ESXi virtualization.


<img src="../screenshots/Project Architecture.png">

## Architecture Overview

- **Platform**: VMware ESXi (bare-metal, hosted on Hetzner)
- **Public Access**: Allowed only via RD Gateway over HTTPS (TCP 443)
- **Internal Access**: Isolated subnet via `vSwitch1` (no uplink)
- **Routing**: Handled via RD Gateway's dual NIC setup

## Components Breakdown

### ESXi Host (`static.your-server.de`)
- **Type**: Bare-metal (Hetzner)
- **Mgmt IP**: `138.X.X.X` (via `vmk0`)
- **Physical NIC**: `vmnic0`
  - MAC: `90:XX:XX:XX:XX:XX`
  - Connected to Hetzner uplink

#### vSwitch0 (WAN)
- **Has uplink**
- **Portgroups**:
  - `VM Network` → Used by RD Gateway (Public NIC)
  - `Management Network` → Used by `vmk0`

#### vSwitch1 (Internal)
- **No uplink** (isolated network)
- **Portgroup**: `Internal lab` → Used by all internal VMs

---

### Virtual Machines

#### 1. RDGateway (Dual NIC)
- **Public NIC**:
  - `vSwitch0` → `VM Network`
  - IP: `138.X.X.X` (Public IP)
- **Internal NIC**:
  - `vSwitch1` → `Internal lab`
  - IP: `192.168.0.6`
- **Role**: Securely tunnels RDP from the internet to internal RDS VMs

---

#### 2. DC01 (Domain Controller)
- **NIC**: `vSwitch1` → `Internal lab`
- **IP**: `192.168.0.5`
- **Roles**:
  - AD DS (Active Directory Domain Services)
  - DNS (internal name resolution)
  - DHCP (optional)

---

#### 3. RDS01 (User A VM)
- **NIC**: `vSwitch1` → `Internal lab`
- **IP**: `192.168.0.10`
- **Gateway**: `192.168.0.6` (RD Gateway)
- **DNS**: `192.168.0.5` (DC01)
- **Role**: Remote Desktop Session Host for User A

---

#### 4. RDS02 (User B + File Server)
- **NIC**: `vSwitch1` → `Internal lab`
- **IP**: `192.168.0.11`
- **Gateway**: `192.168.0.6` (RD Gateway)
- **DNS**: `192.168.0.5` (DC01)
- **Role**: Remote Desktop Host + File Server



