# Enterprise-Grade RDP Infrastructure on ESXi with RD Gateway and AD Services

## Objective

This infrastructure simulates a secure, enterprise-grade RDP access architecture using VMware ESXi as the hypervisor. It includes an RD Gateway with dual NICs, an isolated internal network, Active Directory services, and Remote Desktop Session Hosts. All components are hosted on a single bare-metal server (Hetzner) to emulate a production environment for remote desktop access in corporate settings.


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

___

### Virtual Machines

#### 1. RDGateway (Dual NIC)
- **Public NIC**:
  - `vSwitch0` → `VM Network`
  - IP: `138.X.X.X` (Public IP)
- **Internal NIC**:
  - `vSwitch1` → `Internal lab`
  - IP: `192.168.0.6`
- **Role**: Securely tunnels RDP from the internet to internal RDS VMs

___

#### 2. DC01 (Domain Controller)
- **NIC**: `vSwitch1` → `Internal lab`
- **IP**: `192.168.0.5`
- **Roles**:
  - AD DS (Active Directory Domain Services)
  - DNS (internal name resolution)
  - DHCP (optional)

___

#### 3. RDS01 (User A VM)
- **NIC**: `vSwitch1` → `Internal lab`
- **IP**: `192.168.0.10`
- **Gateway**: `192.168.0.6` (RD Gateway)
- **DNS**: `192.168.0.5` (DC01)
- **Role**: Remote Desktop Session Host for User A

___

#### 4. RDS02 (User B + File Server)
- **NIC**: `vSwitch1` → `Internal lab`
- **IP**: `192.168.0.11`
- **Gateway**: `192.168.0.6` (RD Gateway)
- **DNS**: `192.168.0.5` (DC01)
- **Role**: Remote Desktop Host + File Server
___

## Networking Configuration (Conceptual Overview)

- A **vSwitch (Virtual Switch)** in VMware ESXi acts like a physical network switch inside your hypervisor.
- It connects:
  - Virtual Machines (VMs)
  - VMkernel NICs (e.g., for management)
  - Physical NICs (vmnicX) for external traffic

### vSwitch0 (WAN)
- Has an **uplink** to `vmnic0`, which connects to the physical network (Hetzner uplink).
- Carries:
  - `VM Network` (RD Gateway's public-facing NIC)
  - `Management Network` (vmk0 for ESXi web UI access)

### vSwitch1 (Internal)
- No uplink → fully isolated.
- Connects internal VMs:
  - RDGateway (internal NIC)
  - DC01
  - RDS01
  - RDS02
- Provides east-west VM traffic without exposure to the internet.

### Port Groups
- Logical containers inside a vSwitch that apply policies (VLANs, security, traffic shaping).
- You assigned:
  - `VM Network` and `Management Network` to vSwitch0
  - `Internal lab` to vSwitch1
___

## Network Flow

-  The RDP client is configured to connect to an internal server (e.g., `192.168.0.10`) **via an RD Gateway**.
-  The client first establishes a secure HTTPS tunnel to the **RD Gateway’s public IP** (e.g., `138.X.X.X`) on port **443**.
-  The RD Gateway **authenticates the user**, decrypts the tunnel, and **forwards the RDP session internally** to the target VM (`192.168.0.10`) over port **3389**.
-  The target RDS VM processes the session and **responds via its default gateway (`192.168.0.6`)**, which is the RD Gateway’s internal NIC.
-  The RD Gateway re-packages the response and **sends it securely back** to the RDP client through the existing HTTPS tunnel.
-  The internal RDS VM is never directly exposed to the internet, ensuring a secure, brokered RDP session.

___

## 🧪 Test Scenarios (to be checked)

- ✅ **RDP Access to RDS01 via RD Gateway**
  - Use an `.rdp` file or RDP client with:
    - Gateway: 138.X.X.X
    - Target: 192.168.0.10

- ✅ **DNS Resolution from RDS01/RDS02**
  - Confirm DNS is resolving via DC01 (192.168.0.5)

- ✅ **Domain Join Verification**
  - Ensure RDS VMs are successfully joined to the domain hosted by DC01

- ✅ **NAT & Routing Check**
  - Internal VMs should have internet access (optional) via RDGW if RRAS is configured

- ✅ **Secure Gateway Tunnel**
  - Verify that direct RDP to internal IPs from outside is blocked

- ✅ **Isolation Validation**
  - No uplink on vSwitch1 confirms backend network is isolated

___

## 🧠 Future Improvements

- 🔒 **Add pfSense or OPNsense**
  - For more granular firewall, NAT, and VPN control

- 🌐 **Enable RRAS on RDGW**
  - Allow NAT and internet access for internal VMs securely

- 📊 **Implement Monitoring**
  - Setup Event Log forwarding, RDP audit logs, and custom alerts

- 📡 **Split RDS Roles**
  - Separate File Server and RDS roles on RDS02 for performance

- 🛡️ **Use SSL Certificates**
  - Install a valid SSL cert on RDGW to eliminate warning prompts for users

- ⚙️ **Create HA/Cluster Setup**
  - Use multiple RD Gateways and RDS hosts in HA mode for production-like simulatio

___