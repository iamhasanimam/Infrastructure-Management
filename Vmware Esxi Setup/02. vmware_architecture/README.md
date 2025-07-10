# How Vmware Virtualization Works

Virtualization : Virtualization is the process of creating a virtual version of something — like a server, desktop, storage device, operating system, or network — instead of a physical one.

Use Case: 

- It allows multiple operating systems and applications to run on a single physical machine.
- This is done using a layer of software called a hypervisor, which abstracts hardware resources (CPU, RAM, Disk, etc.) and allocates them to virtual machines (VMs).

### How VMware Virtualization Works

VMware provides virtualization solutions based on **Type-1 hypervisor** architecture (bare-metal), primarily via **VMware ESXi**, allowing multiple virtual machines (VMs) to run on a single physical server.

#### Architecture diagram
```
+--------------------------------------------------+
|            Virtual Machines (VMs)                | 
| +----------------+   +----------------+          |
| |  Guest OS 1    |   |  Guest OS 2    |          |
| | + Apps         |   | + Apps         |          |
| +----------------+   +----------------+          |
+--------------------------------------------------+
                     ↑
+--------------------------------------------------+
|    Virtual Machine Monitor (VMM - per VM)        |
| - Emulates CPU/memory if needed                  |
| - Coordinates guest OS access to real hardware   |
+--------------------------------------------------+
                     ↑
+--------------------------------------------------+
|          VMkernel (Type-1 Hypervisor)            |
| - Directly runs on hardware                      |
| - Handles CPU, memory, disk, networking          |
| - Manages VM lifecycle                           |
+--------------------------------------------------+
                     ↑
+--------------------------------------------------+
|             Physical Hardware                    |
|  (CPU, RAM, Storage, NICs, etc.)                 |
+--------------------------------------------------+

```

#### Virtual Machines (VMs)
- These are the guest environments — the VMs you create and run. Each VM acts like a real computer, with its own:
- Guest OS (like Windows, Linux, etc.)
- Applications (just like any regular PC)
- But these VMs don’t run directly on the hardware — they are simulated environments created by the layers below.

---


#### Virtual Machine Monitor (VMM)
- Think of the VMM as a translator + bodyguard for each VM.
- It emulates CPU and memory when needed — for example, if the guest OS tries to do something that isn’t allowed directly on hardware.
- It ensures isolation between VMs.
- It coordinates access to hardware by deciding what a guest VM can or cannot do.
- Each VM has its own VMM, managed by the layer below.

---

#### VMkernel (Type-1 Hypervisor)
- It's a bare-metal hypervisor, meaning:
- It runs directly on the physical hardware (not on top of another OS).
- It handles hardware resource management — CPU scheduling, memory, storage, network.
- It creates, destroys, and manages VMs.
- The VMkernel is what makes multiple VMs on a single server possible.

---

#### Physical Hardware
- This is your real server:
- CPU, RAM, SSD/HDD, NICs (network cards), etc.
- The VMkernel directly communicates with this layer to allocate hardware resources to each VM — either fully or partially virtualized depending on the setup.

---