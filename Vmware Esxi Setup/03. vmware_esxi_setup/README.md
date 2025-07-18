# Enterprise-Grade RDS Infrastructure on ESXi with RD Gateway and Active Directory (infra.local)

## 🎯 Objective

This infrastructure simulates a secure, enterprise-grade Remote Desktop Services (RDS) access architecture using **VMware ESXi** hosted on a **Hetzner bare-metal server**. It leverages:

- 🛡️ An RD Gateway with dual NICs for secure internet tunneling  
- 🧠 A fully functional Active Directory (AD DS) and DNS  
- 🖥️ Multiple RD Session Hosts (RDS01, RDS02)  
- 🌐 A fully isolated internal network (`vSwitch1`) with no uplink for security  
- 🔐 HTTPS-based RDP access over the internet (port 443)

All components are orchestrated to replicate a real-world, enterprise-like remote access environment **without relying on any cloud platforms**.

---

## 🧱 Architecture Overview

| Component | Role | IP Address | Key Notes |
|----------|------|------------|-----------|
| **DC01** | Domain Controller, DNS | `192.168.0.3` | Root of `infra.local` |
| **RDGW01** | RD Gateway (Dual NIC) | `Public: 138.X.X.X`, `Internal: 192.168.0.1` | Entry Point over HTTPS |
| **RDCB01** | RD Connection Broker | `192.168.0.2` | Load Balancing brain |
| **RDS01** | RD Session Host 1 | `192.168.0.4` | Connected to domain |
| **RDS02** | RD Session Host 2 + File Server | `192.168.0.5` | Combined role |
| **FS01** | Optional File/Tools Server | `192.168.0.6` | Utility server |
| **Dedicated** | JumpBox (Direct RDP only) | `192.168.0.7` | Not part of the RD farm |

---

## 🏗️ Platform Stack

- **Hypervisor**: VMware ESXi 8.0 (on Hetzner dedicated server)
- **Virtual Networking**:
  - `vSwitch0 (WAN)` → Has uplink to Hetzner NIC (`vmnic0`)
  - `vSwitch1 (Internal)` → No uplink, used by all internal VMs

---

## 🌐 Networking Topology

### 🔹 vSwitch0 (WAN)
- **Uplink**: `vmnic0` → Hetzner Public Uplink  
- **Portgroups**:
  - `VM Network` → Used by RDGW01's **Public NIC**
  - `Management Network` → Used by ESXi's `vmk0` for admin access

### 🔹 vSwitch1 (Internal)
- **No uplink** (air-gapped from internet)
- Used by all VMs: DC01, RDCB01, RDS01, RDS02, FS01, JumpBox
- RDGW01 internal NIC is the gateway to the outside world

---

## 🔁 Network Flow (RDP over HTTPS)

1. 🧍 User launches an `.rdp` session using:
   - **Gateway**: `138.X.X.X` (RDGW01)
   - **Target**: `192.168.0.4` (RDS01)

2. 🔐 RDP client creates an HTTPS tunnel to RDGW01 over port 443

3. 🧠 RDGW01 validates credentials and **initiates an internal RDP session** (port 3389) to `192.168.0.4`

4. 📦 RDGW01 **bridges** the external and internal sessions like a reverse proxy:
   - Takes screen, input/output, audio, etc. from RDS01
   - Sends it securely via HTTPS tunnel back to the client

5. 🔒 The internal VM (`192.168.0.4`) is never directly exposed, ensuring complete isolation

---

## 🧠 Role-by-Role Breakdown

### 1️⃣ DC01 – Domain Controller
- **NIC**: Internal only (vSwitch1)
- **IP**: `192.168.0.3`
- **Roles**:
  - AD DS (creates and manages `infra.local`)
  - DNS Server for internal name resolution
  - DHCP (optional, not active in current setup)

### 2️⃣ RDGW01 – RD Gateway
- **NICs**:
  - Public: `vSwitch0 → 138.X.X.X`
  - Internal: `vSwitch1 → 192.168.0.1`
- **Roles**:
  - Accepts HTTPS connections from outside
  - Forwards RDP traffic to internal RDS hosts
  - NAT/Routing (RRAS optional)
  - Can be enhanced to provide outbound NAT to internet

### 3️⃣ RDCB01 – RD Connection Broker
- **IP**: `192.168.0.2`
- **Role**:
  - Maintains active RDP sessions
  - Load balancing among RDS hosts
  - Required for larger RDS deployments

### 4️⃣ RDS01 and RDS02
- **IPs**: `192.168.0.4` and `192.168.0.5`
- **DNS**: Points to `192.168.0.3`
- **Gateway**: `192.168.0.1`
- **Roles**:
  - RDS01: Standard RDP Host
  - RDS02: RDP Host + File Sharing (SMB)

---

## ✅ Test Cases

| Test | Expected Outcome |
|------|------------------|
| 🔐 RDP via RD Gateway | HTTPS tunnel → Internal session to RDS01 |
| 📛 DNS Resolution | Internal VMs resolve names via DC01 DNS |
| 🏢 Domain Join | RDS01, RDS02 successfully join `infra.local` |
| 🌍 NAT Access (optional) | Internet access via RDGW01 if RRAS configured |
| 🔒 Isolation | vSwitch1 has no uplink – full internal traffic |
| 🚫 Public Block | Direct RDP to `192.168.0.x` from outside = blocked |

---

## 🔧 Future Enhancements

- 🔁 **Enable RRAS** on RDGW01 for internet access to internal VMs  
- 🛡️ **Install pfSense** for advanced firewall + NAT  
- 📊 **Monitoring**: Add EventLog Forwarding, WMI + RDP audit logs  
- 🖥️ **Split Roles**: Decouple file server from RDS02  
- 📜 **SSL Certs**: Apply public CA certs on RDGW01  
- 🧯 **High Availability**: Use RD Gateway + RDS01 clusters (HA + Load Balancer)

---

## 📷 Screenshots

Include key screenshots for:

- ESXi host setup
- vSwitch configuration
- DC01 AD/DNS setup
- RDGW01 dual NIC + RRAS
- RDS host sessions
- DNS zones + PTR record config
- RD Gateway Manager session bridge

---

## 🧠 Why This Setup Is Valuable

✅ **Real-World Readiness**  
This architecture mirrors what real enterprises use to provide **secure RDP access** to internal apps/servers without exposing them directly.

✅ **Cloud-Free**  
Simulates enterprise infra without AWS/Azure. Perfect for **air-gapped** or **regulated** environments.

✅ **Job-Ready Learning**  
Covers topics like DNS, AD, RD Gateway, subnet isolation, and secure access—all critical for infra/cloud/DevOps roles.

---

## 🔗 GitHub Repository

**Live Here**: [https://github.com/iamhasanimam/Infrastructure-Management](https://github.com/iamhasanimam/Infrastructure-Management)

---

