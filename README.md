# 🖥️ VMware ESXi-Based Secure RDP Infrastructure Lab

## 🌐 Project Overview

This project simulates a secure, enterprise-grade Remote Desktop access infrastructure using **VMware ESXi** as the hypervisor on a bare-metal Hetzner server. The architecture is carefully designed to resemble real-world corporate environments, including:

- 🔐 **RD Gateway Server** with dual NICs (WAN and LAN)
- 🧠 **Active Directory** with DNS and DHCP services
- 💻 **RDS Session Hosts** (RDS01, RDS02) for user-based RDP access
- 🧭 Internal and external networking via **vSwitches**
- ⚙️ Clean routing, NAT, and secure domain-joined authentication
- 🚧 Fully isolated internal network for added security

## 📦 Components Breakdown

| Component       | Role                                             | IP Address         |
|----------------|--------------------------------------------------|--------------------|
| `RDGW01`        | RD Gateway (Dual NIC: Public & Internal)         | 138.X.X.X / 192.168.0.6 |
| `DC01`          | Active Directory, DNS, DHCP                      | 192.168.0.5        |
| `RDS01`         | RDP Host for user sessions                       | 192.168.0.10       |
| `RDS02`         | RDP + File Server                                | 192.168.0.11       |
| `ESXi Host`     | Hypervisor (Hetzner Dedicated Server)            | Mgmt IP: 138.X.X.X |
| `vSwitch0`      | External switch connected to Hetzner uplink      | (Has uplink)       |
| `vSwitch1`      | Internal lab switch (no uplink)                  | VLAN isolated      |

## 🛡️ Key Features

- ✅ **Gateway-forwarded RDP access** only (no direct RDP to internal hosts)
- ✅ **Clean private IP subnetting** with domain join enforcement
- ✅ **Internal-only file sharing (SMB)** and service resolution via AD-integrated DNS
- ✅ **DHCP dynamically assigns IPs** for flexibility during scaling
- ✅ **No reliance on prebuilt cloud tools** – ideal for edge, air-gapped, or regulated environments

## 🚀 Use Cases

- ✅ Corporate-style RDP access simulation for MSPs, Enterprises, and IT Pros
- ✅ Perfect for **DevOps, Infra, and Security Engineers** building hybrid models
- ✅ Interview-ready demonstration of core Windows infrastructure skills
- ✅ Reliable training ground for VMware, AD DS, DHCP, DNS, NAT, and SMB

## Actively looking to colloborate and help businesses

This setup was built from scratch as part of my ongoing work in infrastructure, cloud, and enterprise IT simulation. I’m actively looking for opportunities where deep system knowledge, hands-on infra building, and hybrid architecture skills are valued.

📨 **Open to opportunities in DevOps, Infrastructure Engineering, and Secure Access Design roles.**

🧠 Let’s build the future of hybrid cloud—secure, controlled, and in your hands.
