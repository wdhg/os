# Scheduling

## Process States

![Process State Transition Diagram](https://i.stack.imgur.com/KqZRe.png)

[Source](https://stackoverflow.com/questions/43844228/process-states-on-a-single-processor-vs-dual-core-system)

States are:

- **New**: the process is being created.
- **Ready**: the process is runnable and awaiting for processor time.
- **Running**: the process is currently executing on the processor.
- **Waiting / Blocked**: the process is waiting for an event to occur.
- **Terminated**: process is being deleted.

When multiple processes are ready to be run, something needs to decide which one to run. This is the purpose of the **scheduler**.

## Goals of Scheduling Alorithms

- **Ensure Fairness**: comparable processes should get comparable services.
- **Avoid Indefinite Postponement**: no process should starve (i.e. every process should get CPU time).
- **Enforce Policy**: e.g. process priorities.
- **Maximize Resource Utilisation**: e.g. CPU, I/O devices, etc.
- **Minimize Overhead**: minimize context switches, scheduling decisions should be fast and lightweight.

Certain systems have even more goals depending on their workload:

- **Batch Systems**:
  - Throughput: number of jobs per unit of time.
  - Turnaround time: time between job submission and completion
- **Interactive Systems**:
  - Response Time (crucial): time between request being issued and first response.
- **Real-Time Systems**:
  - Soft Deadlines: failure to meet deadline is at most an annoyance (e.g. leading to degraded video quality).
  - Hard Deadlines: failure to meet deadline is severe (e.g. leading to a plane crash).

## Preemptive vs Non-Preemptive Scheduling

**Preemptive** scheduling only allows processes to run for a maximum amount of fixed time. This requires a clock interrupt.

**Non-preemptive** scheduling lets the process run until it blocks or voluntarily releases the CPU.

## CPU-Bound vs I/O-Bound Processes

**CPU-bound processes** spend most of their time using the CPU.

**I/O-bound processes** spend most of their time waiting for I/O events to occur, and tend to only use the CPU breifly before issuing new I/O requests.

## General-Purpose Scheduling

A general-purpose scheduler for an OS should use an algorithm that:

- favours short and I/O-bound jobs to get good resource utilisation and short response times.
- can quickly determine the nature of the job and adapt to changes (processes can have periods where they are I/O-bound, and other periods where they are CPU-bound).

## Summary

**Scheduling algorithms often need to balance conflicting goals**. This is to ensure fairness, enforce policy, maximise resource utilisation, etc.

**Different scheduling algorithms are appropriate in different contexts**. A batch system will require a very different scheduling algorithm compared to an interactive system or real-time system.

Well-studied scheduling algorithms include:

- First-Come First-Served (FCFS).
- Round Robin (RR).
- Shortest Job First (SJF).
- Shortest Remaining Time (SRT).
- Multilevel Feedback Queues (MLFQS).
- Lottery Scheduling.
