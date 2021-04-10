# Introduction

## Computer Architecture Overview

### Processor

- Controls computer hardware
- Executes instructions that constitute programs

### Memory

-  Stores data and code

###  I/O components

- Read and write from

### I/O devices

- Intelligence exists in (HW) I/O controller

### System bus

- Interconnects different hardware components
- Provides communication facility between components

## Operating Systems: Top and Bottom Level View

Operating Systems provide a "clean" interface to user applications. This is to abstract the "ugly" interface of the hardware devices, allowing for applications to be more portable as they don't need to be concerned with all possible combinations of hardware.

## Kernels

Kernels are a part of an OS that:

- Is always loaded in memory.
- Executes in a privaleged mode with access to all hardware.

## Operating System Goals

### Managing Resources

An OS must efficiently use it's available resources as they are limited, and it must be capible of sharing its resources among users.

These resources are:

- **Processors** (CPU cores, multi-socket CPU, SMT/hyper-threading)
- **Memory** (caches, RAM, etc)
- **I/O Devices** (displays, GPUs, network interface cards (NICs), printers, etc.)
- **Internal Devices** (clocks, timers, interrupt controllers, etc)
- **Non-volatile Storage** (disks, SSDs, DVDs, tapes, etc)

### Providing Clean Interfaces

An OS must hide the complexity of lower level hardware from higher level systems. This is done by hiding the low level hardware details from programs, and by providing higher level abstractions that user programs can use to request to access to hardware resources.

## Operating System Characteristics

### Sharing

An OS must:

- be able to share data, programs, and hardware. This is done by using time and space multiplexing.
- offer resource allocation. It must:
  - ensure efficient and fair usage of memory, CPU time, disk space, etc.
  - offer mutual exclusion for some resources (unsafe operations must be protected).
  - protect against corruption of data, whether accidental or malicious.

### Concurrency

An OS must:

- support running multiple activities in parallel (I/O and computation, multiple user programs, etc.)
- be capible of switching activities at arbitrary moments in time. It must guarantee fairness such that every activity can / will have CPU time, and it must ensure prompt replies to important activities.
- guarantee safe concurrency. This includes:
  - offering primitive structures (such as mutexs) to allow for the synchronisation of actions.
  - protecting from user / process interference. Programs should have their own "space" that no other program can read from / write to accidentally / maliciously.

### Non-determinism

Events occur in a random and unpredictable order. An OS must be capible of handling this without failure.

### Storing Data

An OS must:

- offer easy access to files through user-defined names. This is done my managing directory structures, links, and shared disks.
- enforce access controls on data, including reading, writing, removing, executing, and copying permissions.
- protect against unforseen failures (e.g. by making backups).
- manage storage devices by organising disks into volumes, partitions, redundant arrays, etc.

## Operating System Facilities

- **Simplified I/O** (e.g. access to disks, DVDs or remote file servers)
- **Virtual Memory** (to seperate the user logical memory from physcial memory)
- **File Systems** (long term storage on disk accessed by names)
- **Program Interaction and Communication** (e.g. messages, pipes, sockets, shared memory)
- **Network Communication** (sending / receiving data on a network)
- **Security** (prevents programs from accessing resources not allocated to them)
- **Human-computer Interface** (user interaction with programs, command language, shells)
- **Administration, Management, and Accounting**

## Types of Operating Systems

- **Multiprocessor** (Windows, MacOS, Linux)
  - Many CPU cores and CPUs.
- **Server OS** (Solaris, FreeBSD, Linux, Windows Server)
  - Share hardware / software resources e.g. internet servers.
- **Mainframes, Supercomputers** (OS/390)
  - Bespoke hardware, limited workloads.
- **Smartphone** (iOS, Android).
  - Power-efficient CPUs.
- **Embedded OS** (QNX, VXWorks)
  - Home utilities.
  - Only trusted software.
- **Real-time OS**
  - Time oriented (not performance or I/O).
  - e.g. controlling a manufacturing plant.
- **Sensor Network OS** (TinyOS)
  - Resource / energy efficient.

## Operating System Structures

### Monolithic kernels

A single executable with its own address space, acting as a single black box. Their structure is implied through pushing parameters to the stack and trapping to execture system calls. These are the most popular style of kernel.

Advantages:

- Efficient calls within kernel.
- Easier to write kernel components due to shared memory.

Disadvantages:

- Complex design with lots of interactions.
- No protection between kernel components. If one part crashes the whole program crashes (less reliable).

### Microkernels

Small sized kernels that have functionality in user-level servers. They use inter-process communication (IPC) between servers, and have separate servers for device I/O, file access, process scheduling, etc.

Advantages:

- Kernel is less complex so less error-prone.
- Servers can have clean interfaces.
- If a server crashes, it can be restarted without crashing the entire kernel (more reliable).

Disadvantages:

- Large amount of overhead from IPC within kernel.

### Hybrid Kernels

Combines features from both monolithic and microkernels. They are a compromise between clean design and performance overhead.

Advantages:

- More structured design.

Disadvantages:

- Performance penalty for user-level servers.

## Example Kernels

### Linux (monolithic)

Linux system calls are implemented by pushing arguments into registers or the stack and issuing traps to switch from a user context to a kernel one.

Linux supports a rich set of programs (through GNU project). This includes:

- shells (bash, ksh, zsh, etc), compilers, editors, etc.
- desktop environments (GNOME, KDE, i3).
- Utility programs (file, fitlers, editors, compilers, text processors, sys admin, etc).

Interupt handles are the primary means to interact with devices. They stop the current process, save the current state, starts the driver, then returns to the saved state. They are typically written in assembler as they need to be as fast as possible.

The I/O scheduler orders disk operations.

Linux also supports static in-kernel components and dynamically loadable modules.

### Windows (hybrid)

The Windows NT kernel consists of two layers:

- **Executive**: most services.
- **Kernel**: thread scheduling and synchronisation, traps, interrupt handlers, and CPU management.

Programs are built on top of dynamic code libraries (DLLs), and DLLs implement the OS services in a modular fashion.

NTOS provides system calls. It is loaded from ntoskrnl.exe at boot.

Hardware Abstraction Layer (HAL) abstracts out DMA operations, BIOS config, CPU types.

Device drivers are loaded into kernel memory and are dynamically linked against NTOS and HAL layers.
