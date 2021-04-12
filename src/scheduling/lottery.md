# Lottery Scheduling (Preemptive)

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
