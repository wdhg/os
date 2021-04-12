# Fair-Share Scheduling

In fair-share, users are assigned some fraction of the CPU and the scheduler takes into account who owns a process before scheduling it. Fair-share is usually implemented along side other scheduling algorithms, most commonly round-robin.

For example, assume there are two users each with 50% share of the CPU:

- User 1 has 4 processes: `A, B, C, D`.
- User 2 has 2 processes: `E, F`.

Under a fair-share round-robin scheduler, a potential execution order of these processes could be `A, E, B, F, C, E, D, F`.

