# Semaphores

A **semaphore** is a structure which allows a process to stop and wait for a signal, and continue once it receives this signal. 

Semaphores are accessible using the following **atomic** operations:

- `init(s, i)` initialise a semaphore `s` with value `i`.
- `down(s)` (also known as `P()`): wait to receive a signal through semaphore `s`.
- `up(s)` (also known as `V()`): transmit a signal through semaphore `s`.

## Semaphore Internals / Implementation

A semaphore consists of two private pieces of data:

- a **counter** (a non-negative integer).
- a **queue** of waiting processes.

The initial value of a semaphore counter determines how many processes can access the protected shared data in parallel.

A pseudo implementation of a semaphore is:

```
init(s, i) ::=
  counter(s) = i
  queue(s) = {}

down(s) ::=
  if counter(s) > 0
    counter(s) = counter(s) - 1
  else
    push current process P to queue(s)
    suspend current process P

up(s) ::=
  if queue(s) is empty
    counter(s) = counter(s) + 1
  else
    resume one process in queue(s)
```

## Example Usage

### Mutual Exclusion

If two thread, A and B, need have a shared critical section, mutual exclusion can be achieved by using a **binary semaphore**.

A **binary semaphore** is a semaphore with its counter initialised to 1. It works similiarly to a lock / mutex.

```c
void thread_work_a(semaphore_t *s) {
  down(s);
  critical_section();
  up(s);
}

void thread_work_b(semaphore_t *s) {
  down(s);
  critical_section();
  up(s);
}

int main(int argc, char *argv[]) {
  semaphore_t *s;
  init(s, 1);

  // start thread A and B in a random order
}
```

### Ordering of Events

Assume thread A needs to complete its work before thread B. This can be achieved by using a semaphore with counter initialised to 0:

```c
void thread_work_a(semaphore_t *s) {
  critical_section();
  up(s);
}

void thread_work_b(semaphore_t *s) {
  down(s); // blocks until thread A completes its work and calls up(s)
  critical_section();
}

int main(int argc, char *argv[]) {
  semaphore_t *s;
  init(s, 0);

  // start thread A and B in a random order
}
```

### Producer / Consumer

A **producer** adds items to a shared buffer, whereas a **consumer** fetches and removes items from a shared buffer:

```
          Deposit                 Fetch
Producer --------> Shared Buffer ------> Consumer
                     (N Items)
```

There can exist multiple producers and consumers.

A producer can only deposit items in the buffer if:

- there is enough space.
- mutual exclusion is ensured.

A consumer can only fetch items from the buffer if:

- the buffer is not empty.
- mutual exclusion is ensured.

A buffer of size N can hold between 0 and N items.

```c 
// ...

semaphore_t *item_removed; // semaphore to send signals when items are removed
semaphore_t *item_added;   // semaphore to send signals when items are added
semaphore_t *buffer_lock;  // semaphore to ensure mutual exclusion on buffer

item_t *buffer[10]; // buffer of 10 item pointers

void producer_work() {
  while(1) {
    item_t *item = produce();

    down(item_removed);
    down(buffer_lock);

    deposit(buffer, item);

    up(buffer_lock);
    up(item_added);
  }
}

void consumer_work() {
  while(1) {
    down(item_added);
    down(buffer_lock);

    item_t *item = fetch(buffer, item);

    up(buffer_lock);
    up(item_removed);

    consume(item);
  }
}

```
