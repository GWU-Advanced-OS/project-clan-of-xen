# Hypothesis
  Now we are brainstorming..
  
## SET OF QUESTIONS

1. Summarize the project, what it is, what its goals are, and why it exists.
- Xen provide level 1 hypervisor, which means provides full virtualization. But at the same time, xen adopt paravirtualization (pv) technology, so each domain use physical resources thorouth dom0.(a)

- Xen exists to provide resource efficient and robust virtualization. Xen is for the people that want to be able to create virtual machines on demand and have them all share the same hardware. Xen aims to minimize the overhead from this and make the user experience virtually identical to the experience on a true machine. Most implementations of virtualization are for the use case where the user uses both the host OS and the guest OSes frequently. Contrary to this, Xen is rarely directly used by the user and the guest OSes are the focus. (g)

- The Xen project allows efficient virtualization by providing a management VM (dom0). In addition, Xen provides paravirtualization for legacy systems that don’t have VM hardware capabilities. (k)


2. What is the target domain of the system? Where is it valuable, and where is it not a good fit? These are all implemented in an engineering domain, thus are the product of trade-offs. No system solves all problems (despite the claims of marketing material).
- Xen are used at wide internet hosting services. Xen provides type 1 hypervisor, which can provide better security. I guess this is the reason that xen could be used in automotive industries. On the other hand, xen needs some overhead for executing.(a)
- This is valuable to large server farms, research institutions and private companies that utilize frequent virtual machines. Amazon has long relied on Xen for virtualization in EC2 instances. xen is hugely valuable when the virtualized systems are not cooperative. The original paper for Xen titled Xen and the Art of Virtualization uses the terminology "when resources are oversubscribed, or users uncooperative". However, if the user is not going to be deploying more than 100 virtual OSes on one machine and instead looks to use 1 or 2, then more traditional approaches or process level isolation would be better(g)


3. What are the "modules" of the system (see early lectures), and how do they relate? Where are isolation boundaries present? How do the modules communicate with each other? What performance implications does this structure have?
- The characteristics of xen is dom0, which is default virtual machine, and other virtual machine, domU. Xen is usually para-virtualization (pv), then, each domU use physical resources via dom0. This system can improve security but need some computation cost.(a)

- In Xen the hypervisor is the only component that has full privileges, it is designed to be small and limited as possible. The hypervisor relies on a trusted guest OS to provide hardware drivers, a kernel, and a userland. This privileged domain is uniquely distinguished as the domain that the hypervisor allows to access devices and perform control functions. By doing this, the Xen developers ensure that the hypervisor remains small and maintainable and that it occupies as little memory as possible.(b)
- It is from dom0 that the user administers Xen, from dom0 the user can create,destroy other domains. Network and storage devices can also be manipulated. dom0 also has privileged access to hardware and it uses it to expose virtualized hardware to other virtual machines. (b)
- There are two mechanisms for Communication, hypercalls and event notifications. Calls from domains to Xen are done using hypercalls and notifications from Xen to domains are done using an asynchronous event mechanism. Pending events raised by Xen are handled by each domain by maintaining a bit mask. The callback handler for those events are responsible for resetting the set of pending events. A domain can also defer event handling  by setting a Xen-readable software flag.(b)


4. What are the core abstractions that the system aims to provide. How and why do they depart from other systems?
- In my understanding, basically, xen abstract most of computer resources, including memory and cpu. Each domain on virtual machine can use these resources via dom0.(a)
- Xen aims to provide strong isolation that is performative. They use "paravirtualization" to do this. This is where virtual machine abstractions that are very similar to the actual hardware are used. The OS is required to be modified a bit still under this approach. It is important to note that this provides a large performance improvement over full virtualization yet still allows for the isolation. Xen also allows binaries to go unmodified for compatibility. Paravirtualization also allows Xen to expose real machine addresses to the guest OSes, notably supporting superpages and page coloring. (g)
- domU virtual machines can assume safety via hardware-based isolation. (k)


5. In what conditions is the performance of the system "good" and in which is it "bad"? How does its performance compare to a Linux baseline (this discussion can be quantitative or qualitative)?
- In my understanding, the core technology of xen is para-virtualization and hardware virtual machines.(a)
- Linux VM (dom0) is utilized to provide management services throughout the machines execution. Hardware virtualization (when not doing paravirutalization) via the machine architecture is used. These are abstracted to the VMs in order to seem like they are running as a regular server. This is especially useful for cloud-based systems (k)

6. What are the core technologies involved, and how are they composed?
- In general, dom0 is the “refmon” where most operations are required to go through. This VM has the utmost privilege on the system, and is the first to boot up. (k)

7. What are the security properties of the system? How does it adhere to the principles for secure system design? What is the reference monitor in the system, and how does it provide complete mediation, tamperproof-ness, and how does it argue trustworthiness?
- By thorough control via dom0, xen can monitor every activity. Also, since each domain is separated, isolated well.(a)
- One recent optimization is the dom0-less static partitioning to avoid the complexity of running an extra management virtual machine after bootup. (k)
- Also, if dom0 is used, there is the option for xen PCI passthrough, which gives the VM guest more rights - instead of sending requests to the event channel via dom0, the privileged domU can bypass the access the hardware directly (much faster but less secure (k)

8. What optimizations exist in the system? What are the "key operations" that the system treats as a fast-path that deserve optimization? How does it go about optimizing them?
- Actually, I do not understand the optimization mechanism of them. But scheduling function is in virtual machine, I guess, there are some scheduling system like capability based operating systems.(a)

9. Subjective: What do you like, and what don't you like about the system? What could be done better were you to re-design it?
- I feel it is a very good architecture for cyber security. Dom0 and domU relationship looks good for security issues. As of now, I do not know the details of technology, so I do not have any opinion to rebuild the technology.(a)

 

### EMIL

The Project Hypervisor serves as the basis for various types of virtualization, including commercial, desktop, server, IaaS and embedded products, as well as cloud solutions. It is a type-1 or baremetal hypervisor, which means it operates below even the host OS itself, right on top of the bare hardware. 

Probably the most appropriate target domain is the large cloud providers (such as AWS). It is valuable where the hypervisor should be lightweight and hence cause less cold-starts for the VM instances that can be fired up simultaneously and natively when virtualization is enabled for the CPU. It is not a good fit in cases where dedication of a certain resource must be allocated for a piece of the virtualization system (Dom0) that is mandatory and adds some overhead in almost every operation. Additionally, as opposed to other VM systems like VirtualBox, it is not meant to be used for general User, where they can use both a host OS and a guest OS alongside.

The system consists of two big layers: Hypervisor at the low layer and Domains (guest VMs) on top of that. Hypervisor modules are the Scheduler, MMU, Timers and Interrupts, whereas Domains typically are the Guest OS, except the very first special Domain (Dom0) that contains drivers for all the devices (native and virtual) in the system. Isolation boundaries are present on the Domain layer, where all Domains are isolated from each other, except for the Dom0 that domains can interact with throughout their lifetime. Having Dom0 cases slower startup times and increases complexity (need for various Xen tools.

The two core abstractions are Paravirtualization (PV) and Full Hardware-assisted Virtualization (HVM). With PV guest VMs are optimized to run as VM, even if the core hardware actually doesn’t support virtualization extensions, whereas today most virtualization systems rely on the hardware support.

In PV conditions system performance is good and with HVM it is bad. 

My answer to question #3 covers this, I think.

Xen claims to be the most reliable hypervisor to use for security-first environments because of its architecture, advanced security features, and an industry-leading security disclosure process. In Xen, Dom0 would be the reference monitor, since it’s the one taking care of the all the communication between the hardware the Guest VMs, which at the same time is how Xen provides complete mediation. Tamperproof-ness is there too, but in the past, there's been security vulnerabilities especially with the guest hosts where they're able to gain root access to the main host. So far, I think of Xen as trustworthy.

Not sure about this :(

I like how Xen is separate from the OS and goes even further by separating out each component, it provides reasonable security, better performance is offered compared to some of the competitors, and lets you run any OS you want in a virtual environment. Most importantly, it is OPEN SOURCE! 
Compared to others on the market, Xen provides less management tools and you should expect the same level of management tools as most other open-source software.



### KEVIN
### Useful articles:
- https://www.serverwatch.com/server-news/hypervisor-face-off-kvm-vs-xen-vs-vmware/
- http://www-archive.xenproject.org/files/Marketing/HowDoesXenWork.pdf
- https://www.linux.com/news/kvm-or-xen-choosing-virtualization-platform/

## VEDANT

* Summarize the project, what it is, what its goals are, and why it exists.

Xen provides a type-1 hypervisor, thus providing services to allow multiple operating systems to execute concurrently on the same computer hardware. It aims to minimize the overhead in implementing robust virtualization and make the whole experience of the VM as realistic as running the OS on an actual machine. It exists so that users can simultaneously use host and guest OSes on the same hardware. 

* What is the target domain of the system? Where is it valuable, and where is it not a good fit? These are all implemented in an engineering domain, thus are the product of trade-offs. No system solves all problems (despite the claims of marketing material).

Xen’s primary users are private companies such as Internet hosting services that provide virtual private servers, research firms that run various sandboxed guest systems, and hardware appliance vendors that need to ship their appliance running several guest systems. Xen is a good fit when resources are in high demand and there are too many users that don’t usually cooperate well together. Xen might not be the best fit when it comes to running a small set of guest OSes which can simply be one by deploying one or more hosts running standard OSes that do not require much system administration.  


* What are the "modules" of the system (see early lectures), and how do they relate? Where are isolation boundaries present? How do the modules communicate with each other? What performance implications does this structure have?

* The Xen Project Hypervisor is an exceptionally lean (<65KSLOC on Arm and <300KSLOC on x86) software layer that runs directly on the hardware and is responsible for managing CPU, memory, and interrupts. It is the first program running after the bootloader exits. The hypervisor itself has no knowledge of I/O functions such as networking and storage.
* Guest Domains/Virtual Machines are virtualized environments, each running their own operating system and applications. The hypervisor supports several different virtualization modes, which are described in more detail below. Guest VMs are totally isolated from the hardware: in other words, they have no privilege to access hardware or I/O functionality. Thus, they are also called unprivileged domain (or DomU).
* The Control Domain (or Domain 0) is a specialized Virtual Machine that has special privileges like the capability to access the hardware directly, handles all access to the system’s I/O functions and interacts with the other Virtual Machines. he Xen Project hypervisor is not usable without Domain 0, which is the first VM started by the system. In a standard set-up, Dom0 contains the following functions:
    * System Services: such as XenStore/XenBus (XS) for managing settings, the Toolstack (TS) exposing a user interface to a Xen based system, Device Emulation (DE) which is based on QEMU in Xen based systems
    * Native Device Drivers: Dom0 is the source of physical device drivers and thus native hardware support for a Xen system
    * Virtual Device Drivers: Dom0 contains virtual device drivers (also called backends).
    * Toolstack: allows a user to manage virtual machine creation, destruction, and configuration. The toolstack exposes an interface that is either driven by a command line console, by a graphical interface or by a cloud orchestration stack such as OpenStack or CloudStack. Note that several different toolstacks can be used with Xen
* Xen Project-enabled operating systems: Domain 0 requires a Xen Project-enabled kernel. Paravirtualized guests require a PV-enabled guest. Linux distributions that are based on Linux kernels newer than Linux 3.0 are Xen Project-enabled and usually include packages that contain the hypervisor and Tools (the default Toolstack and Console). All but legacy Linux kernels older than Linux 2.6.24 are PV-enabled, capable of running PV guests.

