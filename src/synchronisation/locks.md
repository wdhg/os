# Locks / Mutexes

A **lock** (also known as **mutex**) is a piece of data that restricts access to a certain resource. A process can:

- "acquire" a lock, preventing any other process from accessing the data associated with the lock.
- "release" it, allowing the next process to acquire it.

Locks are purely symbolic as nothing forces a process to acquire a lock before accessing the protected data.

An example of how we would use these locks would be:

```c 
int *accounts;
lock_t accounts_lock; // a lock of some type lock_t

void withdraw(int account_no, int amount) {
  acquire(accounts_lock);

  int balance = accounts[account_no];
  accounts[account_no] = balance - amount;

  release(accounts_lock);
}
```

One attempt at implementing `acquire` and `release` is:

```c
// here lock_t is replaces with *int

void acquire(*int lock) {
  while (*lock != 0) {}
  *lock = 1;
}

void release(*int lock) {
  *lock = 0;
}
```

but this has the issue that the two instructions in `acquire` (the `while` loop and the assignment) are **not** atomic. This means that two or more processes could read at the same time that a lock is free and attempt to acquire it.

How this is resolved is in the hardware of the CPU. A test-and-set instruction is provided, which can atomically test the value of lock and can set it if it is released. Most CPUs have this instruction.

Revisiting the previous example, `acquire` can be reimplemented using a new function called `TSL`, whichs returns `1` if it acquires the lock, and `0` otherwise:

```c
void acquire(*int lock) {
  while (TSL(lock) != 0) {}
}
```

This implementation is known as a **spin lock**.

## Spin Locks

Spin locks are locks that use busy waiting whilst waiting for the lock to release. They:

- waste CPU time from busy waiting, but are useful then the wait is expected to be short as they can be faster than alternative implementations (more on this later).
- may run into the priority inversion problem.

## Priority Inversion Problem

The priority inversion problem is a scenario where a high priority process is indirectly preempted by a lower priority process.

For example, assume there are two processes H and L with high and low priorities respectively. Under normal conditions, H should be scheduled if it is runnable. However, consider the following scenario:

1. H is blocked whilst waiting for I/O.
2. L, during this time, acquires lock A.
3. H is unblocked and is scheduled.
4. H attempts to acquire lock A.

If A is a spin lock, then H would still be scheduled as it is busy waiting and has a higher priority than L. L would never get scheduled, effectively preventing both H and L from continuing.

Two solutions to this problem is to either not use spin locks, or to temporarily increase L's priority whilst it holds lock A.

## Lock Granularity

**Lock granularity** is the amount of data a lock is protecting. Lock granularity is described as either course or fine:

- **Course-grained**: a large amount of data is protected by the lock.
- **Fine-grained**: a small amount of data is protected by the lock.

**Lock overhead** is a measure of cost associated with using locks (e.g. memory space, initialisation, acquire and release times).

**Lock contention** is a measure of the number of processes waiting for a lock. Typically as contention increases, parallelism decreases.

An example of course-grained locking is:

```c
// ...

int *accounts;
int accounts_lock;

void withdraw(int account_no, int amount) {
  acquire(accounts_lock);

  int balance = accounts[account_no];
  accounts[account_no] = balance - amount;

  release(accounts_lock);
}

void process_work_a() {
  withdraw(1, 40);
}

void process_work_b() {
  withdraw(2, 40);
}
```

An example of fine-grained locking is:

```c
// ...

struct {
  int balance;
  int account_lock;
} Account;

Accont *accounts;

void withdraw(int account_no, int amount) {
  acquire(accounts[account_no]->lock);

  int balance = accounts[account_no]->balance;
  accounts[account_no]->balance = balance - amount;

  release(accounts[account_no]->lock);
}

void process_work_a() {
  withdraw(1, 40);
}

void process_work_b() {
  withdraw(2, 40);
}
```

Different level granularity solutions have their uses, but typically as granularity gets finer:

- lock overhead increase due to there being more locks.
- lock contention decreases as its less likely that processes will contend for the same lock.
- complexity increases as lock acquisition order and storing the locks safely needs to be considered.

To minimise lock contention and maximise concurrency:
- choose a finer lock granularity (but understand its tradeoffs)
- release a lock as soon as possible (make critical sections as **small** as possible)

## Read / Write Locks (RW Locks)

If multiple processes are simply reading the same data, then a lock may not even be required. However, if a process begins to write to this data, then a lock is required.

The solution to this are read / write locks (also known as reader-writer locks). RW locks allow for concurrent read-only access, or exclusive read and write access.

For example:

```c
// ...

int *accounts;
rwlock_t accounts_lock; // for some rwlock_t type

void get_balance(int account_no) {
  aquire_read(accounts_lock);
  int balance = accounts[account_no];
  release(accounts_lock);
  return balance;
}

void withdraw(int account_no, int amount) {
  acquire_write(accounts_lock);
  accounts[account_no] -= amount;
  release(accounts_lock);
}

void process_work_a() {
  int balance = get_balance(1);
  // ...
}

void process_work_b() {
  int balance = get_balance(2);
  // ...
}

void process_work_c() {
  withdraw(2, 40);
}
```

Processes A and B can safely execute at the same time. However, as soon as process C acquires the lock in write mode, processes A and B will be blocked until C completes it's workload.
