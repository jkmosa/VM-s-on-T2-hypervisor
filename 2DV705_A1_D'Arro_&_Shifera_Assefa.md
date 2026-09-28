# Building VM's with type 2 hypervisor  

## Front page 

- Author
  - Roberto D'Arro (rd222kd@student.lnu.se)
  - Mussie Shifera Assefa (ms228qx@student.lnu.se)

- Submission date 
  - 09/28/2026

- Host OS
  - window 11 version 25H2

- Type 2 hypervisor
  - virtualbox (7.2.2r170484) (where 170484 is the specific build number)

## Table of content 

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

The chosen host machine under the hybervisor is Windows 11 version 25H2 and Oracle virtualbox as a type 2 hypervisor. 

Oracle VM VirtualBox is a free, open-source virtualization software that runs on Windows 11 to let you create and run other operating systems—like Linux, macOS, or older Windows versions—as virtual machines right on your desktop.

![virtualbox](images/virtualbox.png)

Alongside installing the hypervisor recommended to install the extension package this is usefull i.e NIC (network card interface) ability fot the guest machine.   

### Tasks A2 ( Create the machine)

The standard procedure for creating a new Virtual Machine (VM) within Oracle VM VirtualBox. This process allows an administrator to isolate and run a secondary guest operating system safely inside a host computer's existing environment.

**Technical Methodology**

The setup process involves configuring virtual hardware resources to mimic a physical computer through a five-step wizard:

  - Instance Initialization: Initiating the wizard via the New button to define the machine's name, host destination folder, and the guest operating system's ISO image.

  ![machine namimg](/images/first.png)

  - Hardware Allocation: Adjusting the allocation sliders to assign host RAM and CPU cores to the virtual environment based on guest OS system requirements.

  ![username and host](/images/second.png)

  - Virtual Disk Provisioning: Setting up a virtual hard disk using the Dynamically allocated format, ensuring host storage is only consumed as data is written within the VM.

  ![vcpu and vRAM](/images/third.png)

  - Configuration Validation: Reviewing the hardware summary matrix before clicking Finish to build the virtual machine.

  ![vDISK](/images/fourth.png)

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

> 2. command free -h

- What the guest sees:

The guest sees a rigid, fixed amount of total, used, and available random-access memory (i.e Total: 1.9GiB). It acts as if it has exclusive, physical ownership of this address space.

- What it really is:

This memory is a slice of the host's physical RAM, mapped to the guest's virtual address space by the hypervisor. 

In reality, the host may use techniques like

  - memory ballooning, 

  - page sharing, or overcommitting, meaning the physical RAM backing the guest can dynamically shift or be shared with other VMs.

![RAM](/images/meme.png)

> 3. command lsblk

- What the guest sees:

The guest sees emulated SCSI/SATA drives (i.e sda and partion sda1 and sda2). It treats these as local, physical hardware drives.

- What it really is:

The sda disk is not a physical hard drive. It is a virtual disk file (most likely a .vdi or .vmdk file) stored on the host computer's actual drive. It is configured inside VirtualBox as a virtual SATA or SCSI controller storage device.

The sr0 drive is an ISO image file (such as a Linux installation disc or VirtualBox Guest Additions) mapped by the VirtualBox hypervisor to act like a physical disc inserted into a tray.

![storage](/images/listblock.png)

> 4. command ip addr

- What the guest sees:

The guest sees local network interfaces (such as eth0, ens3, or enp0s3) assigned specific IP addresses, usually within a private subnet. It perceives these as dedicated network interface cards connected to a physical switch.

- What it really is:

The guest is using a Virtual Network Interface Card (vNIC) software-emulated by the hypervisor. This vNIC connects to a virtual bridge or switch inside the host's operating system, which multiplexes the traffic alongside other VMs over the host's actual physical network interface card (pNIC).

![nat network](/images/nat_ip.png)

> 5. command systemd-detect-virt

- What the guest sees:

oracle: The guest operating system explicitly detects that it is running in an environment managed by Oracle. The system detects specific firmware parameters, DMI/SMBIOS motherboard strings, and hypervisor CPUID flags that identity the virtual hardware layer.

- What it really is:

Oracle VM VirtualBox: The hypervisor software running on the host machine is VirtualBox. When the virtual machine boots up, VirtualBox populates the guest's virtual BIOS/EFI system tables with identifying metadata (such as the system manufacturer string).

Detection Mechanism: The systemd-detect-virt command looks inside files like /sys/class/dmi/id/product_name and detect execution in a virtualized environment.

![virtualized environment](/images/virt.png)

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

