# Round-Robin (RR) (Preemptive)

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
