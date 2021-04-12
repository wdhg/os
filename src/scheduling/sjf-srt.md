# Shortest Job First (SJF) (Non-Preemptive) and Shortest Remaining Time (SRT) (Preemptive)

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

## Note: Knowing Run-Times in Advance

Typically, run-times aren't usually available or even calculatable in advance. However, they can be estimated by either:

- Computing CPU burst estimates on heuristics (e.g. based on previous execution history). However, this is not always applicable.
- The user provides estimates for their process. In some cases, a user may attempt to cheat the system by under estimating their run-time, and so the scheduler needs to be able to counteract this (e.g. terminating or penalising processes after they exceed their estimated run-time).
