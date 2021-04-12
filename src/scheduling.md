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

### Round-Robin Scheduling (RR) (Preemptive)

RR is like FCFS, except the scheduler can preemptively stop the current process and put it back on the end of the ready queue when its time quantum is exceeded.

**Time quantum** is the allotted amount of processor time each process receives upon entering the running state. Quantum (also known as time slice) is usually around 10-200ms.

Advantages of RR:

- It is fair as every ready job gets an equal share of the CPU.
- Response time is fast for a small number of jobs.
- The average turnaround time is low when run-times differ.

Disadvantages of RR:

- Response time is high for large numbers of jobs as the entire ready queue will be long.
- If run-times are similar, turnaround time is high.

RR Overhead is calculated as the time taken to switch contexts divided by the quantum value. For example:

- 4ms quantum with 1ms context switch time = 1/4 = 20% of time (high overhead).
- 1s quantum with 1ms context switch time = 1/1000 = 0.1% of time (low overhead).

In general, as quantum increases:

- overhead decreases.
- response time worsens (e.g. when quantum = ∞, RR becomes FCFS).

Therefore, when choosing a quantum value it should be much larger than than the context switch cost, but also provide a decent response time (depending on the expected workload).

Some example quantum values for standard processes (although values can vary depending on process type, behaviour, priority, etc):

- Linux: 100ms.
- Windows client: 20ms.
- Windows server: 180ms.

### Shortest Job First (SJF) (Non-Preemptive) and Shortest Remaining Time (SRT) (Preemptive)

SJF performs scheduling by using the run-times that are known in advance, and simply picks the shortest job first. This can provide better turnaround time when compared with FCFS, and it is provably optimal when all jobs are available simulatneously.


SRT is the preemptive version of SJF. It also requires that runtimes are known in advance. It chooses the process whose remaining time is shortest.

For example:

- There exist 2 jobs in the ready queue, j1 = 6s and j2 = 4s.
- After 2 seconds, a new job of 1s arrives.
- In SJF, the job execution order looks like:
  1. j2 for 4s.
  2. j3 for 1s.
  3. j1 for 6s.
- In SRT, the job execution order looks like:
  1. j2 for 2s.
  2. j3 for 1s.
  3. j2 for 2s more.
  4. j1 for 6s.

### Note: Knowing Run-Times in Advance

Typically, run-times aren't usually available or even calculatable in advance. However, they can be estimated by either:

- Computing CPU burst estimates on heuristics (e.g. based on previous execution history). However, this is not always applicable.
- The user provides estimates for their process. In some cases, a user may attempt to cheat the system by under estimating their run-time, and so the scheduler needs to be able to counteract this (e.g. terminating or penalising processes after they exceed their estimated run-time).

### Fair-Share Scheduling

In fair-share, users are assigned some fraction of the CPU and the scheduler takes into account who owns a process before scheduling it. Fair-share is usually implemented along side other scheduling algorithms, most commonly round-robin.

For example, assume there are two users each with 50% share of the CPU:

- User 1 has 4 processes: `A, B, C, D`.
- User 2 has 2 processes: `E, F`.

Under a fair-share round-robin scheduler, a potential execution order of these processes could be `A, E, B, F, C, E, D, F`.

### Priority Scheduling (Preemptive)

Under a priority scheduling algorithm, processes are assigned priorities. Priorities are a heuristic used to determine the importance of the workload of the process. The result is that more important workloads (e.g. user facing processes such as video games) can be executed before lower priority ones (e.g. background processes fetching data like the weather).

Priorities can be externally provided (e.g. from the user), or they can be based on process-specific metrics (e.g. their expected CPU burst). Priorities can also be **static** (i.e. unchanging) or **dynamic** (i.e. changing).

For example, assume there exist 3 processes that arrive at the same time:

- `A`: Priority = 4
- `B`: Priority = 7
- `C`: Priority = 1

In this example, higher priority values means higher priority (in practice it can be the other way round). Under these conditions, the processes would execute in the order of `B, A, C`.

### General-Purpose Scheduling

A general-purpose scheduler for an OS should use an algorithm that:

- favours short and I/O-bound jobs to get good resource utilisation and short response times.
- can quickly determine the nature of the job and adapt to changes (processes can have periods where they are I/O-bound, and other periods where they are CPU-bound).

### Multilevel Feedback Queues (MLFQS) (Preemptive)

MLFQS is a practical way of implementing a priority based scheduling scheme. It is rather basic and fairly common, used in many OSs such as: Windows Vista, Windows 7, Mac OS X, Linux 2.6-2.6.23.

It works by having one queue for each possible priority level. The scheduler simply runs the job on the highest non-empty priority queue, and each queue can use a different scheduling algorithm (round-robin is typically used).

MLFQS needs:

- to determine the current nature of the job (e.g. is it I/O bound or CPU-bound?).
- to worry about starvation of lower-priority jobs.
- a feedback mechanism where job priorities are recomputed periodically (e.g. based on how much CPU they have recently used) and processes undergo **aging** where their priority increases as they waits.

Some issues with MLFQS are that:

- **It is not very flexible**: applications have effectively no control and priorities make no guarantees.
- **Does not react quickly to changes**: it often requires a warm-up period (i.e. running the system for a while to get better results), which is a problem for real-time systems, multimedia apps, etc.
- **Cheating is a concern**: a process can add meaningless I/O to boost its priority.
- **Cannot donate priority**: a high priority process that requrie resources held by a low priority process must wait for the algorithm to increase the low priority process's priority before it can continue.

### Lottery Scheduling (Preemptive)

In lottery scheduling, jobs receive lottery tickets for various resources (e.g. CPU time), and at each scheduling decision, one ticket is chosen at random. The job holding that ticket wins, and receives what ever resource it was waiting for.

For example, in a system with 100 tickets for CPU time if process A has 20 tickets, it's probabilty of running during the CPU quantum is 20%. Over a long period of time, this should end up resulting in A having 20% of the overall CPU time.

Advantages:

- **The number of lottery tickets is meaningful**: jobs holding p% of tickets should get p% of resources (unlike priorities).
- **It is highly responsive**: if a new job is given p% of tickets it has p% chance to get what ever resource it requires at the **next** scheduling decision.
- **There is no starvation**: every job will always have a chance of being run.
- **Jobs can exchange tickets**: allowing for priority donation and for cooperating jobs to acheive certain goals.
- **Adding / removing jobs affect the remaining jobs proportionally**.

Disadvantages:

- **Response time becomes unpredictable**: it is not impossible for a process to be unlucky for a few lotteries, which can be bad for real time applications / interactive processes.

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
