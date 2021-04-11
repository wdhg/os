# Scheduling

## Process States

![Process State Transition Diagram](https://i.stack.imgur.com/KqZRe.png)

[Source](https://stackoverflow.com/questions/43844228/process-states-on-a-single-processor-vs-dual-core-system)

States are:

- **New**: the process is being created.
- **Ready**: the process is runnable and awaiting for processor time.
- **Running**: the process is currently executing on the processor.
- **Waiting / Blocked**: the proccess is waiting for an event to occur.
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

## Example Scheduling Alorithms

### First-Come First-Served (FCFS) (Non-Preemptive)

![FCFS Diagram](https://www.researchgate.net/profile/Neetu_Goel4/publication/249645533/figure/download/fig1/AS:668501794643974@1536394656707/First-Come-First-Serve-Scheduling-Characteristics-The-lack-of-prioritization-does.png)

[Source](https://www.researchgate.net/figure/First-Come-First-Serve-Scheduling-Characteristics-The-lack-of-prioritization-does_fig1_249645533)

In FCFS, new processes (and waiting processes once their event arrives) get added to the end of a ready queue. The scheduler takes the first process off the queue and runs it, and either the process gets terminated or added back to the waiting processes list (non-preemptive).

Advantages of FCFS:

- No indefinite postponement as all processes will eventually be scheduled.
- Extreamly simple to implement.

Disadvantages of FCFS:

- Long processes prevent shorter processes from completing, potentially reducing the overall throughput and / or turnaround time.
