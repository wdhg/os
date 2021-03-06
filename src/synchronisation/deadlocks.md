# Deadlocks

A deadlock is a state where multiple processes are waiting for an event that only another waiting process can cause.

A resource deadlock is the most common type of deadlock, and it has 4 conditions that must hold for it to occur: 

- **Mutual Exclusion**: each resource is either available or assigned to exactly one process.
- **Hold and Wait**: a process can request resources whilst it holds another resource which it has previously requested.
- **No Preemption**: resources given to a process cannot be foricbly revoked.
- **Circular Wait**: two or more processes in a circular chain where each process is waiting for a resource held by the next process.

## Resource Allocation Graphs

Using a directed graph, resource allocation can be modelled:

- a directed edge from a resource to process means the process currently owns that resource.
- a directed edge from a process to a resource means the process is blocked and waiting for that resource.

If a cycle appears in the graph, then there is a deadlock.

## Strategies for Dealing with Deadlocks

- **Ignore It**:
  - Also known as "The Ostrich Algorithm".
  - If contention for resources is low, then the number of deadlocks is infrequent, so ignoring the problem may be okay given how rare it can be.
- **Detection and Recovery**:
  - After a system is deadlocked:
    1. Detect the deadlock.
    2. Recover from the deadlock.
- **Dynamic Avoidance**:
  - Dynamically consider every request and consider whether it is safe to grant it.
  - Requires information regarding potential resource use.
- **Prevention**:
  - Prevent deadlocks by ensuring at least one of the four deadlock contentions can never hold.

## Detection and Recovery

Detection and Recovery works by building a resource allocation graph and searches for cycles.

### Detection

At a high level, the algorithm simply performs depth-first search over each node (resource / process) in the graph. At each node, it checks for any cycles. The algorithm is as follows:

1. For each node (resource / process) do:
2. Initialise variable `L` to the empty list.
3. Check if the current node exists in `L` already. If yes then there is a cycle and the algorithm can exit, otherwise continue to 4.
4. From the current node, check for any unmarked outgoing edge. If there is one, goto 5, otherwise goto 6.
5. Pick the unmarked edge, mark it so it isn't inspected again, and follow it to the next node. Goto 3 with this node.
6. If this node is the initial node then no cycles are detected and the algorithm can exit. Otherwise, the search has reached a dead end, so remove it from `L`, set the previous node to the current node, and goto 3.

### Recovery

There are three potential techniques for recovering:

- **Pre-emption**: temporarily take resources from the owner process and give it to the waiting process. This only works in certain conditions as if that process is using the resource then giving it to another state might corrupt its state.
- **Rollback**: processes are temporarily checkpointed (e.g. by making images of their memory and state), and on a deadlock the process is rolled back to the previous state before the deadlock condition.
- **Killing Processes**: select a random process in cycle and kill it. The safety of this technique depends significantly on the workload of the process (e.g. may be safe for compile jobs, but not for a database system).

## Dynamic Avoidance

Dynamic Avoidance works by checking each request to see if it will cause a deadlock.

### Banker's Algorithm (Dijkstra 1965)

Banker's Algorithm is used to check if a sequence of resource allocations will lead to a **safe state** or not.

A **safe state** is any state where there exists a sequence of allocations that guarantees all customers can be satisfied.

Using the example of a bank:

- Firstly set-up with a single type of resource.
  - A bank may have N customers, where each customer has maximum credit expressed in a number of credit units (e.g. 1 credit unit = ??1000).
- Each customer may ask for its maximum credit at some point, use it, and then repay it.
- The banker knows that all customers don't need their max credit at the same time, so they can reserve less than the sum of all credit limits.

For example, take the following setup:

```
     Has Max
+---+---+---+   Four customers A, B, C, and D
| A | 0 | 6 |
+---+---+---+   1 credit unit = ??1000
| B | 0 | 5 |
+---+---+---+   Banker reserves only 10 (instead of 22) units
| C | 0 | 4 |
+---+---+---+
| D | 0 | 7 |
+---+---+---+
   Free: 10
```

A state can be determined to be safe or not by:

- checking if there are enough resources to satisfy **any** maximum request from some customer.
- by assuming that customer pays back their loan, check the next customer closest to the limit, and repeat.
- if this results in the state were all loans are paid back (each customer has 0 credit to pay back), then the state is safe.

Then consider the following states, one is safe and the other is unsafe:

```
    SAFE               UNSAFE

     Has Max             Has Max
+---+---+---+       +---+---+---+
| A | 1 | 6 |       | A | 1 | 6 |
+---+---+---+       +---+---+---+
| B | 1 | 5 |       | B | 2 | 5 |
+---+---+---+       +---+---+---+
| C | 2 | 4 |       | C | 2 | 4 |
+---+---+---+       +---+---+---+
| D | 4 | 7 |       | D | 4 | 7 |
+---+---+---+       +---+---+---+
   Free: 2             Free: 1
```

The first state is safe as the following sequence of requests can occur:

1. C requests 2 credits (Free: 0).
2. C pays back 4 credits (Free: 4).
3. D requests 3 credits (Free: 1).
4. D pays back 7 credits (Free: 8).
5. B requests 4 credits (Free: 4).
6. B pays back 5 credits (Free: 9).
7. A requests 5 credits (Free: 4).
8. A pays back 6 credits (Free: 10).

With this definition of a safe state, we can then only grant requests that lead to a safe state. This doesn't necessarily mean that an unsafe state will lead to a deadlock, but rather that it cannot be guaranteed that it won't lead to a deadlock.

This algorithm can be generalised to handle multiple resource types.

## Prevention

Prevention works by preventing any one of the four deadlock conditions: mutual exclusion, hold and wait, no preemption, or circular wait:

- **Attacking the Mutual Exclusion Condition**: for example, by sharing the resources.
- **Attacking the Hold and Wait Condition**: force all processes to request their resources before starting, and if they aren't all available then wait. This has an issue that the process must know what resources it will need in advance.
- **Attacking the No Preemption Condition**: by forcing a process to give up their resource. This can be bad in certain situations, such as a stopping a process using an external output device (e.g. a printer) would be bad.
- **Attacking the Mutual Exclusion Condition**: either by forcing a process to use a single resource (which would cause optimality issues), or by numbering all resources and forcing processes to request resources in the order they are numbered (this is hard as a large number of resources would be difficult to organise).

## Other Types of Deadlock

### Communication Deadlock

A **communication deadlock** is specific type of deadlock caused by a poor communication policy. For example:

- Process A sends a message to process B, but the message is lost in transit.
- Process B is blocked whilst it is waiting for this message.
- Process A is blocked whilst waiting for a response.
- Both processes are blocked waiting for each other, hence it is a deadlock.

Ordering of resources or careful scheduling are not useful here. Instead, we can use a communication protocol based on timeouts. In the previous example, A and B would timeout and be able to recover fully.

### Livelock

A **livelock** is when processes are not blocked, but they or the system as a whole is not making any progress.

For example, constider a system which has two processes where one receives messages and the other processes them. If the processing thread has a lower priority then under high load (i.e. when many messages are being received), the processing thread will never be able to run. This situation is also known as a **receive livelock**, and they are related to **starvation**.

### Starvation

Starvation is the event where progress is not being made as the system is unable to schedule the required process. Picking the correct scheduling policy is important to prevent certain livelocks.
