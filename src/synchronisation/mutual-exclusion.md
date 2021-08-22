# Attempts to Gain Mutual Exclusion

## Disabling Interrupts

```c
CLI() // clears interrupt flag (IF = 0), disabling interrupts
STI() // set interrupt flag (IF = 1), enabling interrupts
```

By using `CLI` and `STI` to disable / enable interrupts, mutual exclusion can be achieved as other processes won't be able to be scheduled:

```c
int valueMap[10];

// ...

void safe_set_value(int index, int value) {
  CLI();
  valueMap[index] = value;
  STI();
}
```

There are a few issues with this method:

- it only works on single-processor systems. Other processes on other processors will still be running.
- misbehaving / buggy processes may not release the CPU.

Because of this, disabling interrupts should only used by kernel code that requires the fine level of control / performance, and it should be used as little as possible in the kernel code as well.

## Strict Alternation

Strict alternation relies on a variable, say `turn`, to control which process can run at any moment in time. At the end of its critical section, a process will update the `turn` variable to indicate it is done.

For example, assume there are two processes A and B which have an overlapping critical section:

```c
// ...

char turn = 'A';

void process_work_a(void *arg) {
  while (true) {
    while (turn != 'A') {}    // empty loop to await turn
    critical_section();       // enter critical_section
    turn = 'B';               // leave critical_section
    non_critical_section_a(); // non-critical section
  }
}

void process_work_b(void *arg) {
  while (true) {
    while (turn != 'B') {}
    critical_section();
    turn = 'A';
    non_critical_section_b();
  }
}
```

This fixes the problem of having to turn off interrupts, but it also introduces new problems:

- if it is process A's turn, but it is in its non-critical section, it can take as long as it wants / never enter its critical section again. B will be stuck waiting for A, which breaks two rules:
  - that no process should be able to prevent another process from entering its critical section
  - that if no process is in its critical section, and process attempting to enter its critical section should be allowed to do so immeditately.
  - depending on the behaviour of `non_critical_section_a`, it could potentially break a third rule: that no process should be waiting to enter its critical section forever.
- process B is unable to enter its critical section twice in a row as it is forced to wait for process A to update `turn`.

### Busy Waiting

Strict alternation uses a mechanism called **busy waiting**, which means a loop that constantly tests a value until it meets a certain condition. This is bad as it wastes valuable CPU time. As a result, busy waiting should only ever be used when the wait is expected to be short.

## Peterson's Solution / Algorithm

Peterson's solution is similar to strict alternation, but it solves a few of its issues.

An example of it is:

```c
#include <stdbool.h>

int turn = 0;
bool interested[2] = {false, false};

// thead is either 0 or 1
void enter_critical(int thread) {
  int other_thread = 1 - thread;
  interested[thread] = true;
  turn = other_thread;
  while (turn == other_thread && interested[other_thread]) {}
}

void exit_critical(int thread) {
  interested[thread] = 0;
}

// ...

void process_work_0(void *arg) {
  enter_critical(0);
  critical_section();
  exit_critical(0);
}

void process_work_1(void *arg) {
  enter_critical(1);
  critical_section();
  exit_critical(1);
}
```

Using this example, here is a proof of the mutual exclusion:

- Assume processes 0 and 1 are requesting permission to enter the critical section. Then `interested[0] = true`, `interested[1] = true`, and `turn` is controlling which thread is able to enter the critical section.
- If `turn = 0`, process 0 will then be able to enter into the critical section. Process 1 is now waiting for process 0 to set `interested[0]` to `false`, which only happens when process 0 calls `exit_critical`. Once process 0 has called `exit_critical`, then process 1 is allowed to enter the critical section.
- If `turn = 1` the opposite will happen, where process 1 will enter the critical section before process 0.

One major downside to Peterson's solution is that it still uses busy waiting.

