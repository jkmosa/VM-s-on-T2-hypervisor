# Building VM's with type 2 hypervisor  

## Front page 

- Author
  - Roberto D'Arro (rd222kd@student.lnu.se)
  - Mussie Shifera Assefa (ms228qx@student.lnu.se)


- Submission date 
  - 09/28/2026


<!-- TOC -->

- [Part A — the machine](#part-a--the-machine)
  - [Tasks A1 (Install a type 2 hypervisor)](#tasks-a1-install-a-type-2-hypervisor)
  - [Tasks A2 ( Create the machine)](#tasks-a2--create-the-machine)
  - [Tasks A3 (inspect the four resources)](#tasks-a3-inspect-the-four-resources)
- [Part B — allocation](#part-b--allocation)
  - [Tasks B1 (Measure the vCPU count)](#tasks-b1-measure-the-vcpu-count)
  - [Tasks B2 (Produce contention)](#tasks-b2-produce-contention)
  - [Tasks B3 (Your consolidation ratio)](#tasks-b3-your-consolidation-ratio)
- [Reflection](#reflection)
- [Appendix](#appendix)

<!-- /TOC -->


## Part A — the machine 

### Tasks A1 (Install a type 2 hypervisor)

**Physical (host machine)**

The system has strong physical resources, including a 24-core CPU with 32 logical processors, approximately 16 GB of RAM, and a 954 GB NVMe SSD. At the time of measurement, CPU utilization was low at 2%, while memory usage was relatively high at 15.2 GB. The Ethernet interface was not connected but showed no active network traffic with specifically link local ip address of 169.254.32.42.

The chosen host machine under the type 2 hybervisor is Windows 11. 

- Host OS
  * window 11 version 25H2

![Physical CPU](/images/P_cpu.png)

Figure A.1.1: CPU 24 cores and 32 logical processer.

<br>

![Physical RAM](/images/P_ram.png)

Figure A.1.2: RAM 16 GB.

<br>

![Physical DISK free and used](/images/free-and-used-space.png)

Figure A.1.3: DISK 1 TB.

<br>

![Physical Ethernet2](/images/P_eth.png)

Figure A.1.4: Port Ethernet interface with link local IP address.


Oracle VirtualBox is a premier choice as a Type 2 hypervisor for Windows 11 due to its zero-cost licensing, broad accessibility across Windows editions, and cross-platform portability. While Windows 11 includes a native virtualization platform (Hyper-V), VirtualBox serves as a highly flexible, application-level alternative. It is ideal for developers, students, and IT professionals who require rapid deployment of diverse operating systems without modifying the host’s core architecture. It create and run other operating systems—like Linux, macOS, or older Windows versions—as virtual machines right on your desktop.

- Type 2 hypervisor
  * virtualbox (7.2.2r170484) (where 170484 is the specific build number)

Alongside installing the hypervisor recommended to install the extension package this is usefull i.e NIC (network card interface) ability fot the guest machine.

### Tasks A2 ( Create the machine)

**Technical methodology for guest machine**

The standard procedure for creating a new Virtual Machine (VM) within Oracle VM VirtualBox. This process allows an administrator to isolate and run a guest operating system safely inside a host computer's existing environment.

The creation setup process for VMs involves configuring virtual hardware resources to mimic a physical computer through a five-step wizard:

![virtualbox](/images/vm_first.png)

Screenshot A.2.1: VM1, Initiating the wizard via the New button.

<br>

This is where the machine name, operating system distribution (ubuntu) and the CPU architecture of 64-bit is setup.

![machine naming](/images/vm_second.png)

Screenshot A.2.2: VM1, Assigning machine name and location of ubuntu 24-04 ISO image.

<br>

![username and host](/images/vm_third.png)

Screenshot A.2.3: VM1, Setting up the username, hostname, domain name and password.

<br>

Hardware Allocation: Adjusting the allocation sliders to assign the host 2 GB RAM and 2 CPU cores to the virtual environment based on guest OS system requirements.

Virtual Disk Provisioning: Setting up a virtual 20 GB hard disk using the Dynamically allocated format, ensuring host storage is only consumed as data is written within the VM.

![vcpu and vRAM](/images/vm_fourth.png)

Screenshot A.2.4: VM1, Setting up 2 GB RAM and 2 CPU core.

<br>

Configuration Validation: Reviewing the hardware summary matrix before clicking Finish to build the virtual machine.

![vDISK](/images/vm_fivth.png)

Screenshot A.2.5: VM1, Setting up 20 GB storage disk.

<br>

Setting up the virtual network is rather done through the setting button after completing the wizard.

![NAT topology](/images/vm_NAT.png)

Screenshot A.2.6: VM1, Configuring adaptor as NAT.


**Operational Benefits**

  - Resource Efficiency: Dynamic disk provisioning prevents immediate storage depletion on the host machine.

  - Environment Isolation: The guest OS operates inside a sandboxed layer, protecting the host system from potential malware or software instability.


### Tasks A3 (inspect the four resources)


> 1. command lscpu | grep -i hypervisor

- What the guest sees:

The guest sees a hypervisor vendor: KVM. It recognizes that its CPU execution is being managed by a virtualization layer.

- What it really is:

The guest is interacting with a virtualized CPU topology. The hypervisor intercepts specific CPU instructions (using hardware extensions Intel VT-x) and presents a simulated or pass-through subset of the physical host’s CPU capabilities to the guest.

![cpu](/images/hypervisor.png)

Figure A.3.1: VM1, Kernel-based virtualization module technology.

<br>

> 2. command free -h

- What the guest sees:

The guest sees a rigid, fixed amount of total, used, and available random-access memory (i.e Total: 1.9GiB). It acts as if it has exclusive, physical ownership of this address space.

- What it really is:

This memory is a slice of the host's physical RAM, mapped to the guest's virtual address space by the hypervisor. 

In reality, the host may use techniques like

  - memory ballooning (a virtualization technique that allows a physical host to reclaim unused RAM from virtual machines (VMs) and reassign it where it is needed.), 

  - page sharing, or overcommitting, meaning the physical RAM backing the guest can dynamically shift or be shared with other VMs.

![RAM](/images/meme.png)

Figure A.3.2: VM1, total primary memory size 2 GB. 

<br>

> 3. command lsblk

- What the guest sees:

The guest sees emulated SCSI/SATA drives (i.e sda and partion sda1 and sda2). It treats these as local, physical hardware drives.

- What it really is:

The sda disk is not a physical hard drive. It is a virtual disk file (most likely a .vdi or .vmdk file) stored on the host computer's actual drive. It is configured inside VirtualBox as a virtual SATA or SCSI controller storage device.

The sr0 drive is an ISO image file (such as a Linux installation disc or VirtualBox Guest Additions) mapped by the VirtualBox hypervisor to act like a physical disc inserted into a tray.

![storage](/images/listblock.png)

Figure A.3.3: VM1, secondary memory size sda 20 GB and no virtual memory.

<br>

> 4. command ip addr

- What the guest sees:

The guest sees local network interfaces (such as eth0, ens3, or enp0s3) assigned specific IP addresses, usually within a private subnet. It perceives these as dedicated network interface cards connected to a physical switch.

- What it really is:

The guest is using a Virtual Network Interface Card (vNIC) software-emulated by the hypervisor. This vNIC connects to a virtual bridge or switch inside the host's operating system, which multiplexes the traffic alongside other VMs over the host's actual physical network interface card (pNIC).

![nat network](/images/nat_ip.png)

Figure A.3.4: VM1, NAT network topology with IP 10.0.2.15/24.

<br>

> 5. command systemd-detect-virt

- What the guest sees:

oracle: The guest operating system explicitly detects that it is running in an environment managed by Oracle. The system detects specific firmware parameters, DMI/SMBIOS motherboard strings, and hypervisor CPUID flags that identity the virtual hardware layer.

- What it really is:

Oracle VM VirtualBox: The hypervisor software running on the host machine is VirtualBox. When the virtual machine boots up, VirtualBox populates the guest's virtual BIOS/EFI system tables with identifying metadata (such as the system manufacturer string).

Detection Mechanism: The systemd-detect-virt command looks inside files like /sys/class/dmi/id/product_name and detect execution in a virtualized environment.

![virtualized environment](/images/virt.png)

Figure A.3.5: VM1, detected it's running over oracle virtualbox.

<br>

## Part B — allocation 


### Tasks B1 (Measure the vCPU count)

**Experimental Methodology**

The objective of this task was to evaluate how virtual machine processing performance scales when modifying the allocated virtual CPU (vCPU) count.

1. System Provisioning: The virtual machine was shut down sequentially to modify the vCPU configuration in the hypervisor settings to 1 vCPU, 2 vCPUs, and 4 vCPUs.

2. Execution: For each hardware configuration, the matrixprod CPU stress method was executed using stress-ng. This specific method stresses the CPU by performing intensive matrix multiplications, isolating raw computational throughput.

3. Consistency Constraints: Each configuration was tested 3 separate times for a duration of 30 seconds to minimize background noise and ensure statistical reliability.

**Collected Data and Metrics**

The table below outlines the computational throughput measured in bogo-ops/s (bogus operations per second) across all nine iterations.

| vCPU Configuration | Run 1 (bogo-ops/s) | Run 2 (bogo-ops/s) | Run 3 (bogo-ops/s) | Average Throughput (bogo-ops/s) |
|:---                |        :---:       |        :---:       |        :---:       |                             ---:|
| 1 vCPU	           | 2,171.02	          | 2,213.90	         | 2,134.29	          | 2,173.07                        |
| 2 vCPUs	           | 2,255.63	          | 2,282.07	         | 2,352.98	          | 2,296.89                        |
| 4 vCPUs	           | 2,307.03	          | 2,056.16	         | 2,228.32	          | 2,197.17                        |

The spike of the cpu usage in the host machine:

![task manager cpu usage spike](/images/4cpu.png)

### Tasks B2 (Produce contention)

**Experimental Setup & Methodology**

The objective of this task was to analyze the performance impact of CPU resource contention by simulating two virtual machines competing for the same underlying physical hardware.

1. System Provisioning: Two identical virtual machines (Guest 1 and Guest 2) were configured, each allocated 2 vCPUs and 4 GB RAM.

2. Baseline Test (Isolated): Guest 1 was booted alone while Guest 2 was entirely powered off. The benchmark tool stress-ng was executed to measure peak isolated performance.

3. Contention Test (Concurrent): Both Guest 1 and Guest 2 were powered on simultaneously. The benchmark command was initiated on both machines at the exact same time to force resource competition.

**Collected Data & Benchmark Metrics**

The throughput metrics below reflect the computational output measured in bogo-ops/s (bogus operations per second) for both scenarios.

| Test Scenario           |	Guest 1 Throughput (bogo-ops/s)	| Guest 2 Throughput (bogo-ops/s) |	Total Aggregate Throughput |
|:---                     |                :---:            |              :---:              |                        ---:|
| Test 1: Guest 1 Alone	  |          2,318.80	              |          Powered Off	          |           2,318.80         |
| Test 2: Concurrent Load |	         2,150.54	              |          1,928.52	              |           4,079.06         |

The spike of the cpu usage in the host machine:
 
![task manager cpu usage spike](/images/2cpu_2nodes.png)

**Observations**

- Individual Performance Drop: When running concurrently, Guest 1’s performance dropped from 2,318.80 to 2,150.54 bogo-ops/s. This represents a 7.25% loss in processing efficiency for the individual VM due to resource contention.

- Aggregate Gain: While individual performance degraded, the total workload processed by the host CPU hardware jumped from 2,318.80 to 4,079.06 bogo-ops/s (43.15% higher system utilization).


### Tasks B3 (Your consolidation ratio)

![ubuntu 24.04.5 LTS](images/two_machine.png)

**Consolidation Ratios & Resource Allocation**

This analysis evaluates the consolidation metrics for a multi-VM deployment on the host hardware. The deployment scenario assumes two active virtual machines, each provisioned with 2 vCPUs, 4 GB RAM, and a 50 GB virtual hard drive.

**Host vs. Promised Allocation Metrics**

| Resource Type  | Host Hardware (Physical) | Total Promised (All VMs) | Consolidation Ratio	         | Overcommitted  |
|:---            |          :---:           |           :---:          |             :---:             |            ---:|
| CPU (Cores)	   | 24 Cores / 32 Threads	  | 4 vCPUs	                 | 6 : 1 (vCPU to Core)	         |    No          |
| Memory (RAM)	 | 16 GB RAM	              | 4 GB RAM                 | 4 : 1 (Promised to Physical)  |    No          |
| Storage (Disk) | 1 TB NVMe SSD	          | 40 GB Virtual Disk       | 25 : 1 (Allocated to Total)   |    No          |


<br>

## Reflection 

**Analytical Evaluation & Deployment Arguments**

Benefits of Resource Overcommitment (What it Buys)

- Capital Efficiency: Maximizes hardware utilization by capitalizing on the fact that idle VMs do not fully exhaust their allocated footprints simultaneously.

- Density & Scalability: Allows organizations to host more workloads on fewer physical servers, directly lowering power, cooling, and hardware procurement costs.

Risks of Resource Overcommitment (What it Risks)

- The "Noisy Neighbor" Effect: If multiple VMs spike in resource utilization simultaneously (as simulated in Task B2), the hypervisor struggles to allocate physical cycles, degrading overall performance.

- System Instability: Extreme memory overcommitment can force the host to swap to disk, drastically slowing down operations or triggering the Out-Of-Memory (OOM) killer, which terminates active processes.

<br>

## Appendix

Part A task 3:

```bash

sudo cat /proc/meminfo | grep MemTotal

sudo cat /sys/class/dmi/id/product_name

```

Part B task 1:

```bash
stress-ng --cpu N --cpu-method matrixprod --timeout 30s --metrics-brief
```
Use code with caution.
(Where N represents the respective vCPU allocation: 1, 2, or 4)

![stress bogo-ops/s](images/stress_cpu1.png)

![stress bogo-ops/s](images/stress_cpu2.png)

![stress bogo-ops/s](images/stress_cpu4.png)

Part B task 2:

```bash
stress-ng --cpu 2 --cpu-method matrixprod --timeout 30s --metrics-brief
```
![stress bogo-ops/s](images/1node.png)

![stress bogo-ops/s](images/2node.png)

