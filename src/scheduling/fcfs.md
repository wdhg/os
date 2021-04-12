# First-Come First-Served (FCFS) (Non-Preemptive)

![FCFS Diagram](https://www.researchgate.net/profile/Neetu_Goel4/publication/249645533/figure/download/fig1/AS:668501794643974@1536394656707/First-Come-First-Serve-Scheduling-Characteristics-The-lack-of-prioritization-does.png)

[Source](https://www.researchgate.net/figure/First-Come-First-Serve-Scheduling-Characteristics-The-lack-of-prioritization-does_fig1_249645533)

In FCFS, new processes (and waiting processes once their event arrives) get added to the end of a ready queue. The scheduler takes the first process off the queue and runs it, and either the process gets terminated or added back to the waiting processes list (non-preemptive).

Advantages of FCFS:

- No indefinite postponement as all processes will eventually be scheduled.
- Extreamly simple to implement.

Disadvantages of FCFS:

- Long processes prevent shorter processes from completing, potentially reducing the overall throughput and / or turnaround time.

