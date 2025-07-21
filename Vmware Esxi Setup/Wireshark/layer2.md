Goal: Simulate True VLANs in ESXi
🔸 VLAN 10 → e.g., 192.168.10.0/24 (Finance)
🔸 VLAN 20 → e.g., 192.168.20.0/24 (Engineering)

These are isolated at Layer 2, and routed only through a router VM that handles sub-interfaces (eth0.10, eth0.20) with .1 IPs in each subnet.

✅ Step-by-Step VLAN Simulation on ESXi
🔧 1. Create Port Groups (Tagged)
Inside vSwitch1 (no uplink):

Name	VLAN ID
PG_VLAN10	10
PG_VLAN20	20

Each of these inserts a VLAN tag (802.1Q) on traffic leaving the VM.

📍 This mimics what a physical switch with VLAN trunks would do.

🖥 2. Create 2 Test VMs + 1 Router VM
VM Name	Interface	Connected to
FINANCE1	eth0	PG_VLAN10
ENGINEER1	eth0	PG_VLAN20
ROUTER	eth0 (trunk port)	PG_VLAN-Trunk (VLAN ID 4095)

💡 VLAN ID 4095 = ESXi trunk mode

It allows the router to receive and tag packets for all VLANs

🧠 3. Configure VLAN Sub-Interfaces in Router VM (Linux)
Install Ubuntu or Alpine for the router (you can also use pfSense or VyOS).

Inside the router VM:

bash
Copy
Edit
# install VLAN package (if needed)
sudo apt install vlan
sudo modprobe 8021q

# Create VLAN 10 sub-interface
ip link add link eth0 name eth0.10 type vlan id 10
ip addr add 192.168.10.1/24 dev eth0.10
ip link set eth0.10 up

# Create VLAN 20 sub-interface
ip link add link eth0 name eth0.20 type vlan id 20
ip addr add 192.168.20.1/24 dev eth0.20
ip link set eth0.20 up

# Enable routing between VLANs
echo 1 > /proc/sys/net/ipv4/ip_forward
Now you’ve got:

eth0.10 → gateway for VLAN 10

eth0.20 → gateway for VLAN 20

🧪 4. Configure Test VMs (No routing yet)
Inside FINANCE1 (on PG_VLAN10):

bash
Copy
Edit
ip addr add 192.168.10.10/24 dev eth0
ip route add default via 192.168.10.1
Inside ENGINEER1 (on PG_VLAN20):

bash
Copy
Edit
ip addr add 192.168.20.20/24 dev eth0
ip route add default via 192.168.20.1
🔁 5. Test Inter-VLAN Communication
From FINANCE1:

bash
Copy
Edit
ping 192.168.20.20
✅ If all is correct, ping should succeed only through router VM.

🔥 Optional: Add Firewall Policies
Install iptables or ufw on the Router VM:

bash
Copy
Edit
# Block Finance from pinging Engineering
iptables -A FORWARD -s 192.168.10.0/24 -d 192.168.20.0/24 -p icmp -j DROP
🧭 What You Just Simulated
Layer	What You Did
L2	Created isolated VLANs via port groups
L2.5	Tagged packets using 802.1Q
L3	Routed between VLANs using sub-interfaces
L4	(Optional) Controlled access via firewall

📌 Bonus: Use in Your Real Infra
You could replace your internal-only vSwitch (vSwitch1) with:

Port Group Name	VLAN ID	Connected VMs
Internal_AD	10	DC01, RDS01, RDCB01
Internal_Apps	20	FS01, JumpBox, RDS02

Then put a router/firewall VM with NIC on trunk (4095) for complete VLAN interconnect control.