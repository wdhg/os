# Race Conditions

A **race condition** occurs when any two processes read and write shared data without the protection of any synchronisation mechanism. The result depends on the execution order of each process (e.g. their exact interleaving).

## Thread Interleavings

The **thread interleavings** of two or more threads is the total set of all possible orderings of their statement execution.

For example, two threads A and B with statements `{A1; A2}` and `{B1; B2}` respectively have thread interleavings:

- `A1 -> A2 -> B1 -> B2`
- `A1 -> B1 -> A2 -> B2`
- `A1 -> B1 -> B2 -> A2`
- `B1 -> B2 -> A1 -> A2`
- `B1 -> A1 -> B2 -> A2`
- `B1 -> A1 -> A2 -> B2`
