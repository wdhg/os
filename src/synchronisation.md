# Synchronisation

Key concepts:

- Critical Sections.
- Mutual Exclusion.
- Atomic Operations.
- Race Conditions.
- Deadlocks.
- Starvation.
- Synchronisation Mechanism (e.g. locks, semaphores, monitors, etc).

These concepts are relevant to **both** processes and threads, but to keep things simple they will both be refered to as processes throughout this chapter unless stated otherwise.

## Critical Sections and Mutual Exclusion

A **critical section / region** is a section of code in which processes access a shared resource.

**Mutual exclusion** ensure that if one process is executing a critical section, no other process can be executing it (processes must request permission to enter critical sections).

A **synchronisation mechanism** is used at the entry / exit of a critical section, and is what processes use to request permission to enter it.

The requirements for mutual exclusion are:

- no two processes may be simulatneously inside the same critical section.
- no process running outside the critical section may prevent other processes from entering the critical section. This means when no process is currently inside a critical section, any process that requests permission to enter must be allowed to do so immeditately.
- no process requiring access to its critical section can be delayed forever (i.e. all processes requesting entry must be granted it at some point).
- no assumptions are to be made about the relative speed of processes i.e a process cannot assume that another process will exit its critical section in a given amount of time.
