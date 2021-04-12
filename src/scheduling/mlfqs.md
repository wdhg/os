# Multilevel Feedback Queues (MLFQS) (Preemptive)

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

