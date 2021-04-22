# Xen - Final Report

## Project Summary
Xen is a virtual machine (VM) hypervisor, that provides full hardware virtualization. But at the same time, Xen allows VMs to boot with ParaVirtualization (PV) technology, so each domain can use physical resources through the core domain (dom0). In addition, Xen's PV allows legacy systems to run VMs even if they don't have high-level VM hardware capabilities. Xen's `Project Hypervisor` serves as the basis for various types of virtualization, including commercial, desktop, server, Infrastructure as a Service (IaaS), embedded products, as well as cloud solutions. 

As a type-1 or bare-metal hypervisor, Xen operates below even the host OS itself, right on top of the machine hardware. Xen exists to provide resource-efficient and robust virtualization for customers and developers that need to create VMs on demand and have them all share the same hardware. Xen aims to minimize the overhead from this and make the VM experience mimic an OS on regular hardware.  The Xen project allows efficient virtualization by providing a management VM (dom0), which is the manager for most operations throughout a Xen VM's lifecycle.

Most implementations of virtualization are for the use case where the user uses both the host operating system (OS) and the guest OSes frequently (in the case of VMware Fusion or VirtualBox). Contrary to this, Xen is rarely directly used by the user and the guest OSes are the focus.

## Target Domain
Xen Project runs in a more privileged CPU state than any other software on the machine. Responsibilities of the hypervisor include memory management and CPU scheduling of all virtual machines ("domains"), and for launching the most privileged domain ("dom0", the only virtual machine which by default has direct access to hardware). From the dom0, the hypervisor can be managed and unprivileged domains ("domU") can be launched. The dom0 domain is typically a version of Linux or BSD. User domains may either be traditional operating systems, such as Microsoft Windows under which privileged instructions are provided by hardware virtualization instructions (if the host processor supports x86 virtualization, e.g., Intel VT-x and AMD-V), or paravirtualized operating systems whereby the operating system is aware that it is running inside a virtual machine and so makes hypercalls directly, rather than issuing privileged instructions. Xen Project boots from a bootloader such as GNU GRUB, and then usually loads a paravirtualized host operating system into the host domain (dom0).

#### Where it’s valuable and a good fit
* Xen is most valuable where multiple operating systems need to be run concurrently on the same computer hardware. As stated by Barham et al. in their research paper, “Xen and the Art of Virtualization”, Xen is a good fit when the resources are oversubscribed or the users are uncooperative. 
* Xen’s primary users are Internet hosting service companies as they use hypervisors to provide virtual private servers.
   * Amazon EC2 (since August 2006), IBM SoftLayer, Liquid Web, Fujitsu Global Cloud Platform, Linode, OrionVM and Rackspace Cloud use Xen as the primary VM hypervisor for their product offerings.
* Many research firms (especially computer-security research teams) working on development of operating systems that run various sandboxed guest operating systems to observe and study effects of viruses or worms without compromising the host system are big users of Xen.  
* Besides these, Virtual machine monitors also often operate on mainframes and large servers running IBM, HP, and other systems. Server virtualization can provide benefits such as:
   * Consolidation leading to increased utilization
   * Rapid provisioning
   * Dynamic fault tolerance against software failures (through rapid bootstrapping or rebooting)
   * Hardware fault tolerance (through migration of a virtual machine to different hardware)
   * Secure separations of virtual operating systems
   * Support for legacy software as well as new OS instances on the same server
   * Xen's support for virtual machine live migration from one host to another allows load balancing and the avoidance of downtime.
* Finally, hardware appliance vendors may decide to ship their appliance running several guest systems, so as to be able to execute various pieces of software that require different operating systems.

#### Where it’s not a good fit
* Primarily, Xen would not be a good fit when it comes to running a small number of OSes which can be simply done by deploying traditional methods such as deploying one or two hosts running standard OSes that wouldn’t require much time with system administration to provide support for things like performance isolation, scheduling priority, memory demand, etc.  
* Unfortunately, even though it is one of the most widely-used hypervisors, Xen is highly susceptible to attack because it employs a monolithic design (a single point of failure) and comprises a complex set of growing functionality including VM management, scheduling, instruction emulation, IPC (event channels), and memory management.
* As Xen’s functionality has increased, so too has its code base, rising from 45K lines-of-code (LoC) in v2.0 to 270K LoC in v4.0. Such a large code base inevitably leads to a large number of bugs that become security vulnerabilities. Attackers can easily exploit a known hypervisor vulnerability to “jail break” from a guest VM to the hypervisor to gain full control of the system. For example, a privilege escalation caused by non-canonical address handling (in a hypercall) can lead to an attacker gaining control of Xen, undermining all security in multi-tenant cloud environments.
* In their research titled "Deconstructing Xen", Shi et al. have systematically studied all 191 security vulnerabilities published in the Xen Security Advisories (XSA) list, of which 144 are directly related to the core hypervisor. 
* They expand on this by observing that 61.81% directly lead to host denial-of-service (DoS) attacks, 15.28% lead to privilege escalation, 13.89% lead to information leak, and 13.20% use the hypervisor to attack guest VMs.
* Furthermore, they found that more than half of the core vulnerabilities are located in the per-VM logic i.e. things like guest memory management, CPU virtualization, and instruction emulation.


## System Modules
![Xen Control Plane](https://www.researchgate.net/profile/Kamyab-Khajehei/publication/299480036/figure/fig2/AS:388513037078530@1469640134610/The-structure-of-machine-running-the-Xen-hypervisor-8.png)
- The main characteristics of Xen is a dom0 (host/manager), which is default virtual machine, and other virtual machines (domU).
- Xen is usually para-virtualization (pv), then, each domU use physical resources via dom0.
This system could improve security by trading off and potentially increasing computation cost.
- In Xen, the hypervisor is the only component that has full privileges, it is designed to be small and limited as possible. The hypervisor relies on a trusted guest OS to provide hardware drivers, a kernel, and a userland. This privileged domain is uniquely distinguished as the domain that the hypervisor allows to access devices and perform control functions. By doing this, the Xen developers ensure that the hypervisor remains small and maintainable and that it occupies as little memory as possible.
- The system consists of two big layers: Hypervisor at the low layer and Domains (guest VMs) on top of that. Hypervisor modules are the Scheduler, MMU, Timers and Interrupts, whereas Domains typically are the Guest OS, except the very first special Domain (Dom0) that contains drivers for all the devices (native and virtual) in the system. Isolation boundaries are present on the Domain layer, where all Domains are isolated from each other, except for the Dom0 that domains can interact with throughout their lifetime. Having Dom0 cases slower startup times and increases complexity (need for various Xen tools.
- It is from dom0 that the user administers Xen
   - From dom0 the user can create and destroy other domains. 
   - Network and storage devices can also be manipulated.
- There are two mechanisms for communication, hypercalls and event notifications. 
   - Calls from domains to Xen are done using hypercalls and notifications from Xen to domains are done using an asynchronous event mechanism. 
   - Pending events raised by Xen are handled by each domain by maintaining a bit mask. The callback handler for those events are responsible for resetting the set of pending events.
   - A domain can also defer event handling by setting a Xen-readable software flag.


## Core Abstractions
**Paravirtualization (PV)**
 - Xen aims to provide strong isolation that is performative. They use PV to do this. This is where VM abstractions that are very similar to the actual hardware are used. The OS is required to be modified a bit still under this approach.
 - It is important to note that this provides a large performance improvement over full virtualization yet still allows for the isolation. Xen also allows binaries to go unmodified for compatibility.
 - PV also allows Xen to expose real machine addresses to the guest OSes, notably supporting superpages and page coloring. domU virtual machines can assume safety via hardware-based isolation
 - Para Virtualization (PV) - a major component of Xen which is very lightweight and does not require special hardware virtualization extensions (which is great for legacy HW)  
    - The PV API is referenced with the `_pv` suffix, especially in the `tools/libs/guest` directory.  
 - In the main dom0 code, the code is abstracted to check between HVM and PV code segments. This is done similar to how Linux polymorphism is implemented
```
// xg_sr_restore.c     
ctx.restore.ops = ctx.dominfo.hvm
    ? restore_ops_x86_hvm : restore_ops_x86_pv;

// xg_sr_restore_x86_pv.c  
struct xc_sr_restore_ops restore_ops_x86_pv =  
{  
      .pfn_is_valid    = x86_pv_pfn_is_valid,  
      .pfn_to_gfn      = pfn_to_mfn,  
…  
}  
```
- As seen in the picture below, PV VMs are able to connect (through dom0) directly to hardware devices. PV sees a performance boost by not requiring persistent driver emulation.
![Xen PV](https://wiki.xenproject.org/images/7/73/XenPV.png)
 - The Xen team (University of Cambridge) invented Paravirtualization as an efficient alternative to emulated hardware. Unfortunately, not many operating systems and their drivers are built to support paravirtualization. However, Xen and Linux are uniquely coupled for this goal. 
 
  **Full Hardware-assisted Virtualization (HVM)**
 - Resources are abstracted to the VMs in order to seem like they are running as a regular server.
 - Unlike the PV picture above, using HVMs, each call hits the virtualized CPU, memory, etc at the hypervisor layer.
- Especially useful for cloud-based systems that have a large range of clientele.
- See the image below for the dramatic performance difference between HVM and full-PV
![Xen PV vs HVM](https://wiki.xenproject.org/images/9/98/Xen-colors.png)

**Live VM Migration**
- Move one domU to another Xen-monitored host with no noticeable downtime
- Call the Xenlight Library with `xl migrate <domain> <host>`
- `main_migrate`, defined in `tools/xl/xl_migrate.c`   
    - The documentation says a persistent shared storage that both dom0’s can access (like network storage) - Alternatively, Network Block Device (NBD) in the Xen Storage Management system can be used to share the VM harddrive  
    - Migrate - The system forks, creates the required send and recv file descriptors to communicate with the destination,  suspends and renames the source domain, and finishes once the success message from the other host is heard  
    - Migrate Receive (source dom0) - Initialize a domU object and allocate memory, create the domain


## Performance (Good and Bad) versus Linux Baseline
 - Performance graphs and images
 - A lot of this comes from the paper [Xen and the Art of Virtualization](https://xenproject.org/2011/11/29/baremetal-vs-xen-vs-kvm-redux/)
 **Compairision to other VM solutions**
![Baremetal vs KVM vs Xen vs Redux](https://xenproject.org/wp-content/uploads/sites/79/2011/11/Results1.png)  ![Performance Evaluation of Xen, KVM, and Proxmox Hypervisors](https://github.com/GWU-Advanced-OS/project-clan-of-xen/raw/main/images/performance_xen-kvm-proxmox_fig2.jpg)
 - The first table in this section, the PTS testing, shows what we would that Xen and KVM trade blows. Xen tends to win with workloads that are more typical to a server like database hosting. This is not always the case, as  KVM wins with the SQLite test. For CPU benchmarks like John the Ripper, KVM tends to have a margianal advantage.
 - The second table, from the paper *Performance Evaluation of Xen, KVM, and Proxmox Hypervisors*, adds some clarity to this. It is important to note that the testing mehtodology here utilized significantly fewer VMs that Xen can handle. The most ever tested are 10 simultaneous. 
    - As we can see, KVM has the best performance for most CPU throughput, Memory, and cache performance.
        - This may seem intuitive for many
    - Xen beats KVM when it comes to filesystem and application performance.
        - There is a good reason for this. Xen is designed for many concurrent virtual machines that are responsible for traditional server tasks. Databases, Network Attached Storage, and other server applications are the target use-case of Xen. 

![Relative Performance](http://docplayer.net/docs-images/52/12655369/images/page_10.jpg)
![Apache DomainU](https://image.slidesharecdn.com/xen1-131113004608-phpapp01/95/xen-and-the-art-of-virtualization-42-638.jpg?cb=1384303592)

Here, we can see that the xen matches performance well after adding more than one instance. This makes some intuitive sense, since Xen is designed for use cases where  more than a single vm is running. This has been mentioned before, however should be again in order to further explain this gap.


## Core Technologies
 - The system consists of two big layers: Hypervisor at the low layer and Domains (guest VMs) on top of that. Hypervisor modules are the Scheduler, MMU, Timers and Interrupts, whereas Domains typically are the Guest OS, except the very first special Domain (Dom0) that contains drivers for all the devices (native and virtual) in the system. Isolation boundaries are present on the Domain layer, where all Domains are isolated from each other, except for the Dom0 that domains can interact with throughout their lifetime. Having Dom0 cases slower startup times and increases complexity (need for various Xen tools.
 - Linux VM (Dom0) is utilized to provide management services throughout the machine's execution. Hardware virtualization (when not doing paravirtualization) via the machine architecture is used.
 - In Dom0 there are two drivers for supporting networking requests and local dist requests from DomU PV and HVM Guests: 
   - The Network Backend Driver (NBD) communicates directly with the local networking hardware to process all VMs requests coming from the DomU guests.
   - The Block Backend Driver (BBD) communicates with the local storage disk to read and write data from the drive based upon DomU requests.
- In DomU the PV Guest VM is aware that it does not have direct access to the hardware and hence it can recognize that other VMs are probably running on the same machine. Similar to Dom0, a DomU PV Guest also contains two drivers for network and disk access: PV Network Driver and PV Block Driver.
- However, the DomU HVM Guest VM is not aware that it is sharing processing time on the hardware and that other VMs are present. A DomU HVM Guest does not have the PV drivers located within the VM (unlike PV guests). Instead a special daemon is started for each HVM Guest in Dom0, Qemu-DM. Qemu-DM supports the Dom U HVM Guest for networking and disk access requests.

#### Other Technologies 
- Xend daemon is a python application that is considered the system manager for the Xen environment. It leverages the libxenctrl library (see below) to make requests of the Xen hypervisor. All requests processed by the Xend are delivered to it via an XML RPC interface by the Xm (see below) tool:
- Xm is the command line tool that takes user input and passes to Xend via XML RPC.
Libxenctrl is a C library that provides Xend the ability to talk with the Xen hypervisor via Dom0. A special driver within Dom0 delivers the request to the hypervisor.
![Xend](https://github.com/GWU-Advanced-OS/project-clan-of-xen/blob/main/research/xend.jpg?raw=true)
- Qemu-DM - Every HVM Guest running on a Xen environment requires its own Qemu daemon. This tool handles all networking and disk requests from the DomU HVM Guest to allow for a fully virtualized machine in the Xen environment. Qemu must exist outside the Xen hypervisor due to its need for access to networking and I/O and is therefore found in Dom0.
- Xen Virtual Firmware is a virtual BIOS that is inserted into every DomU HVM Guest to ensure that the operating system receives all the standard start-up instructions it expects during normal boot-up providing a standard PC-compatible software environment.
- Xen PCI Passthrough is a new feature in Xen designed to improve overall performance and reduce the load on the Dom0 Guest. This technique allows the DomU Guest to have direct access to local hardware without using the Dom0 for hardware access. The DomU Guest is given rights to talk directly to a specific hardware device instead of the previous method of using Frontend and Backend drivers. The diagram below shows how this feature works
![pcipassthru](https://github.com/GWU-Advanced-OS/project-clan-of-xen/blob/main/research/xen-pci-passthru.jpg?raw=true)

## Security Principles
What are the security properties of the system? How does it adhere to the principles for secure system design? What is the reference monitor in the system, and how does it provide complete mediation, tamperproof-ness, and how does it argue trustworthiness?

### Hypothesis and questions
- dom0 is key components of the xen. domU have to communicate with dom0 to use syscall(hypercall in VM). Thanks to this architecture, xen may check the critical activities of domUs by dom0.
- But, one recent optimization is the dom0-less static partitioning to avoid the complexity of running an extra management virtual machine after bootup. Also, even though dom0 is used, there is the option for xen PCI passthrough, which gives the VM guest more rights - instead of sending requests to the event channel via dom0, the privileged domU can bypass the access the hardware directly (much faster but less secure). How to keep secure in these cases?

### Xen's security feature
**The point of Over whole architecture**
1. Domain0 (dom0) as a reference monitor
   - Dom0 is the initial domain which is starts at boot time of hyperxen hyper visor, and it is a key component for security. Dom0 has privilege to manages domU (domain user). Basically, only dom0 can access the hardware directly. DomU have to communicate with dom0 to use syscall(hypercall in VM). Thanks to this architecture, xen may check the critical activities of domUs by dom0.
   - But sometimes, domU want to access drivers directly. In these cases, there are mechanism of DriverDomain, and also hardware can be passed thourh to the domU.
2. XSM (Xen Security Module) and FLASK (Flux Advanced Security Kernel)
   - Provides **Mandatory Access control** like Security ENriched LInux (SELinux).
3. TCB
    - For tamper proof, Xen has a much larger TCB, and more flexible
4. Verification
    - Xode - 200k + LOC
    - Policy - SELinux style

### Code analysis of each feature
(1) The relationship dom0 and domU
- Since domU can not use hypercall, there is a mechanism which can enable domU to communicate with dom0. Eeach domain can communicate via the `event_channel`. This channel managed by event_fifo.c.

    **example code from event_channel.c**
    ```
    static struct evtchn *alloc_evtchn_bucket(struct domain *d, unsigned int port)
    { .....
        chn = xzalloc_array(struct evtchn, EVTCHNS_PER_BUCKET);
        ...
        for ( i = 0; i < EVTCHNS_PER_BUCKET; i++ )
        {
            chn[i].port = port + i;
            rwlock_init(&chn[i].lock);
        }
        return chn;
    }
    ```
(2) Mandatory Access control provided by XSM
- there are three type of policies.
 a. Type  - Define which hypercall can be executed.
 b. Role  - Each role has set of types which belong to the role.
 c. User  - Each user has set of roles which belong to the user.
- Policies are written in TE(type enforcement) file, like **dom0.te**. General format is below.
    ```
    *allows<ource type> <target type>:<security class> <hypercall>; 
    <example code from "dom0.te">
    allow dom0_t xen_t:xen {
    	settime tbufcontrol readconsole clearconsole perfcontrol mtrr_add
    	mtrr_del mtrr_read microcode physinfo quirk writeconsole readapic
    	writeapic privprofile nonprivprofile kexec firmware sleep frequency
    	getidle debug getcpuinfo heap pm_op mca_op lockprof cpupool_op
    	getscheduler setscheduler hypfs_op
    };
    //dom0_t type can execute hypercalls in the xen class targeting a xen_t type.
    ```
- By using these files, type can be implemented each module, and each type are allocated to role, and users. These set of security attributes associated are called **security context**. These contexts are labeled by security id **sid**. Example of sids are "xen" contains "system_u:system_r:xen_t,s0, "dom0" contains "system_u:system_r:dom0_t,s0", and so on. These sids are put on **sidtab**, and those policies are contained by **policydb**.

### Advanced Feature (dom0-less architecture)
![overview](https://github.com/GWU-Advanced-OS/project-clan-of-xen/blob/main/images/overview.png?raw=true)
(1) device model stub domain
```
xen/xen/xsm/flask/ss/policydb.h
xen/xen/xsm/flask/ss/policydb.c
xen/tools/flask/policy/modules/xen.te
```
![image of stub domain](https://raw.githubusercontent.com/GWU-Advanced-OS/project-clan-of-xen/main/images/overview_withstub.png)
## Optimizations
We can breakdown the optimizations in the system by components.

- Event Channels
Events are how Xen notifies the VMs and event channel is a primitive provided by Xen for event notifications. Events are stored as bitmap which is shared between VMs and Xen. Optimizations such as N-Level search path and per-cpu mask are used to speed up the search process. So far Xen supports 2-level event channel and 3-level is planned for their latest 4.3 version.
2-level event channel is where the first level is pending event bits which basically means what kind of event is pending and the second level is a bitset of pending events themselves.

<img align="right" src="images/Screenshot%20(25).png">
- Buffered IO Ring  
Ring data structure is an implementation of queue data structure and uses pointers for Enqueue and Dequeue. In Xen, guest VMs attach a unique Id for each request that goes into the ring and Xen after processing the request reproduces the id when placing it on the ring for consumption by guest VMs. This optimization allows Xen to reorder requests in order of priority or scheduling
considerations which is useful when dealing with disk requests.
Another optimization that exists where Xen decouples the production of requests or responses from the notification of the other party. It is very similar to batching where a domain can defer delivery of a notification by specifying a threshold of number of responses or in the case of requests, a domain may enqueue multiple entries before invoking a hypercall to alert
Xen. This allows each domain to trade-off latency and throughput requirements.

- Network   
Xen implements zero copy networking where it requires Guest OS to exchange unused page frame for each packet it receives to avoid copying the packet between Xen and guest OS.

- Scheduler  
Xen allows for schedular optimizations to work best with the workload. By default the time slice preemption is set to 30 ms but that can be modified and set to much lower value by setting ```tslice_ms ``` but in case if there is a situation where there is heavy workload and context switches slows down the workflow then context switch can be rate limited as well.  Xen uses Credit scheduling algorithm. It's a weighted proportional fair share algorithm. Each physical cpu has a run queue for virtual cpu's that are placed on it based on their priority.
The paper discussing the XenTune monitoring tool did make it obvious that choosing the right parameters can be difficult and for most users lead to worse performance just like the authors of the XenTune paper had to face.


- Virtual address translation
All hypervisors introduce an abstraction which acts as pseudo memory. This abstraction is P2M which translates guest page tables to machine page tables. In Full virtualization this P2M is managed inside hypervisor by something called shadow page tables. The problem with shadow page tables is that the hypervisor has to synchronize any changes that happen in the guest page table because the guest VM does not know about the existence of shadow page tables. This is slow.
Xen provides a novel idea called Direct Paging which allows it to map virtual address directly to underlying machine address space and the virtual to physical mapping is directly managed by the guest VM. In order to make sure that guest VM
does not make any rogue changes, all page table updates are passed to Xen for validation via hypercall. 

The update hypercall code
```
 struct mmu_update {
     uint64_t ptr;       /* Machine address of PTE. */
     uint64_t val;       /* New contents of PTE.    */
 };
 
 long HYPERVISOR_mmu_update(const struct mmu_update reqs[],
                            unsigned count, unsigned *done_out,
                            unsigned foreigndom)

```

An additional optimization here exists as well where guest OS can queue multiple updates locally before making a hypercall. The code below describes how batching works. The hypercall is defined by the function HYPERVISOR_multicall which takes an array hypercalls.  
```
struct multicall_entry {
     unsigned long op, result;
     unsigned long args[6];
 };
 
 long HYPERVISOR_multicall(multicall_entry_t call_list[],
                           unsigned int nr_calls);
```


- Disk  
Reordering of requests happen at two places. Once inside the domain by the disk scheduling algorithm inside it before queueing them on the ring and the second time inside Xen as it has more knowledge about the actual disk layout. So responses from Xen can also be out of order as we have already mentioned above when discussing the ring data structure. Domains can also explicitly pass down reorder barriers to prevent reordering if order is necessary to maintain higher level semantics.
Xen uses round robin algorithm to process competing disk requests and then it's passed to a elevator scheduler before reaching the disk hardware.


## Subjective Discussion
### Positives
- Our team appreciates how Xen is separate from the OS and goes even further by separating out each component, it provides reasonable security, better performance is offered compared to some of the competitors, and lets you run any OS you want in a virtual environment
- Being ahead in the Paravirtualization space is quite impressive as it is an extremely cool technology that helps a lot of systems boost performance in many scenarios
- Xen is open source, which has many benefits, as most developers would agree

### Negatives
- Xen's monolithic architecture is quite worrisome as it is vulnerable to security hacks, given the size of the Trusted Computing Base (TCB)
- Xen is not always suited to modern Hypervisor tasks. With faster hardware, developers are beginning to take advantage of incredibly performant memory and clock speeds. Unless the end goal is to run many different VMs, admins can shoot themselves in the foot using Xen. Like all solutions to a problem, it is important to recognize that Xen has a time and a place. 

## Resources and Papers
- [KVM vs Xen vs VMWare](https://www.serverwatch.com/server-news/hypervisor-face-off-kvm-vs-xen-vs-vmware/)
- [KVM vs Xen](https://www.linux.com/news/kvm-or-xen-choosing-virtualization-platform/)
- [Baremetal vs Xen vs KVM](https://xenproject.org/2011/11/29/baremetal-vs-xen-vs-kvm-redux/)
- [How Does Xen Work](http://www-archive.xenproject.org/files/Marketing/HowDoesXenWork.pdf)
- [Event Channel Internals](https://wiki.xenproject.org/wiki/Event_Channel_Internals)
- [Xen and the Art of Virtualization](https://www.cl.cam.ac.uk/research/srg/netos/papers/2003-xensosp.pdf)
- [Virtualization in Xen 3.0](https://www.linuxjournal.com/article/8909)
- [Unikernels: Library Operating Systems for the Cloud](https://dl.acm.org/doi/10.1145/2490301.2451167)
- [Advanced Systems Security:Virtual Machine System](http://www.cse.psu.edu/~trj1/cse544-f15/slides/cse544-vmm.pdf)
- [Xen Security Modules:XSM-FLASK](https://wiki.xenproject.org/wiki/Xen_Security_Modules_:_XSM-FLASK)
- [Stub Domain A Step Towards Dom0 Disaggregation](http://www-archive.xenproject.org/files/xensummitboston08/SamThibault_XenSummit.pdf)
- [Xen Project 4.4: Features and Futures](https://events.static.linuxfound.org/sites/events/files/slides/Xen%20Project%204-4%20Features%20and%20Futures_Rev6_0.pdf)
- [Breaking Up is Hard to Do: Security and Functionality in a Commodity Hypervisor](https://www.cs.ubc.ca/~andy/papers/xoar-sosp-final.pdf)
- [Verifying the Safety of Xen Security Modules](https://ieeexplore.ieee.org/document/6004499)
- [Deconstructing Xen](https://www.trustkernel.com/uploads/pubs/nx.pdf)
- [Virtual Cluster Management with Xen](https://link.springer.com/content/pdf/10.1007/978-3-540-78474-6_23.pdf)
- [Inter-domain socket communications supporting high performance and full binary compatibility on Xen](https://dl.acm.org/doi/pdf/10.1145/1346256.1346259?casa_token=EEidqpPOJ3QAAAAA:S4NWWNXZ8Lee6ZSrqWU6TPF9PFOJXHpI6Bgt3YoACALvtG_zo0on5A-XPGKEUJKNd1G99LUmPRPe2Q)
- [Selective Hardware/Software Memory Virtualization](research/Wang2011.pdf)

## Team Contributions
- Aki
    - Xen security architecture and system modules research
    - Security part of the final report
- Biyas
    - System optimizations and modules research
    - optimization part of the final report
- Emil
    - System modules hypotheses, and core technologies
- Gus
    - Xen performance, images, graphs, and comparison with other systems
- Kevin
    - Core abstractions and system modules research
    - Final report based on group research
- Vedant
    - Target domain (where it is and is not a good fit), use cases, and system modules research
    - Monitoring and finalizing presentation slides for group report
    - Monitoring and managing team communication and progress
