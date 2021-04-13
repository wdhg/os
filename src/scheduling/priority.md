# Priority Scheduling (Preemptive)

Under a priority scheduling algorithm, processes are assigned priorities. Priorities are a heuristic used to determine the importance of the workload of the process. The result is that more important workloads (e.g. user facing processes such as video games) can be executed before lower priority ones (e.g. background processes fetching data like the weather).

Priorities can be externally provided (e.g. from the user), or they can be based on process-specific metrics (e.g. their expected CPU burst). Priorities can also be **static** (i.e. unchanging) or **dynamic** (i.e. changing).

For example, assume there exist 3 processes that arrive at the same time:

- `A`: Priority = 4
- `B`: Priority = 7
- `C`: Priority = 1

In this example, higher priority values means higher priority (in practice it can be the other way round). Under these conditions, the processes would execute in the order of `B, A, C`.
