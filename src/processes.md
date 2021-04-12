# Processes

A process is an instance of a program being executed (a running program). They allow for a single processor to run multiple programs "simulatneously". Each process runs on a virtual CPU, and the real CPU can use this to run any single process. Effectively, the single CPU has been divided into multiple virtual CPUs.

## Why have Processes?

They:

- provide an illusion of concurrency (more on this later).
- provide isolation between programs (each process has its own address space).
- simplify programming (programs don't have to worry about other programs).
- allow for better utilisation of resources (different processes require different resources at certain times).

## Time-Slicing for Concurrency

An OS can switch the current running process every few milliseconds such that every process can have CPU time. This is done using a CPU scheduler, and it can be used to provide pseudo concurrency.

## Types of Concurrency

- **Pseudo Concurrency**: a single processor switches between processes by interleaving their runtimes. Overtime this can give the illusion of concurrent execution.
- **Real Concurrency**: multiple processors or CPU cores are used to run multiple processes in parallel.

## Fairness

Fairness is the concept that the CPU scheduler switches fairly between processes. This means that every process should have equal CPU time, or more generally they should have appropriate CPU times determined by each processes priority and workload.

## Context Switches

A context switch is when a processor switches from executing one process to another.

An OS may switch either periodically or in reponse to certain events / interrupts (such as I/O completion). Context switches are not pre-determined because the events causing them are non-deterministic.

Since processes will need to be restarted safely when being switched to, all relevant information (e.g. register values) must be stored when they are switched away from. This is done by storing the data in a process descriptor or a process control block (PCB), which is kept inside a process table.

Context switches are expensive. The process state needs to be saved / restored and CPU caches (including translation lookaside buffer (TLB)) will be lost. It is important for an OS to avoid unnecessary context switches.

## Process Control Block (PCB)

Each process has it's own virtual machine: They have their own virtual CPU, address space (stack, heap, text, data, etc), open file descriptors, etc. Enough data must be stored such that all of this remains preserved after a context switch.

The information that gets stored on a context switch is:

- **Registers**: program counter (PC), page table register, stack pointer, etc.
- **Process management info**: process ID (PID), parent process, process group, priority, CPU used, etc.
- **File management info**: root directory, working directory, open file descriptors, etc.

## Process Creation

There are two types of processes:

- **Foreground**: processes that interact with users.
- **Background**: daemons (e.g. handling incoming mail, printing requests, etc).

Processes are created on:

- system initialisation.
- user request.
- system calls by a running process.

Processes terminated can be caused by:

- **Normal Completion**: when the process completes the execution of its body.
- **System Calls**: e.g. `exit()` in UNIX, `ExitProcess()` in Windows.
- **Abnormal Exit**: when the process runs into an error or unhandled exception.
- **Aborted**: when another process overrules its execution (e.g. killed from terminall).
- **Never**: some processes run in an endless loop and never terminate unless an error occurs (e.g. most web servers).

## Process Hierarchies

UNIX allows for processes to form hierarchies (e.g. parent, child, child's child, etc).

Windows has no notion of process hierarchy. Instead when a process is created its parent is given a handle token to control it. This handle can be passed around to other processes.
