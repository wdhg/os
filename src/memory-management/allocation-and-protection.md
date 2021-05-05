# Allocation and Protection

## Contiguous Memory Allocation

Contiguous memory allocation works by splitting the main memory into two partitions:

- **Kernel**: held in low memory and with an interupt vector (for the Operating System).
- **User**: held in high memory (for user processes).

User processes would be given contiguous ranges of memory. Then we could give the MMU two registers:

- **Base register**: contains the value of the smallest physical address for a particular range (same as the relocation register as before).
- **Limit register**: contains the size of the range of addresses.

Depending on which process is running, the MMU changes the value in these registers (dynamically mapping logical addresses).

Thus the base and limit registers define a logical address space:

```

       0 +------------------+
         | Operating System |
  256000 +------------------+
         | Process          |
  300040 +------------------+ <-- Base: 300040
         | Process          |
  420940 +------------------+ <-- Limit: 120900 (420940 - 300040)
         | Process          |
  880000 +------------------+
         |                  |
 1024000 +------------------+

```

## Memory Protection

This scheme allows for the protection of memory accesses such that each process can only access its own memory. This is done by checking that `base <= address from CPU < base + limit`. If this is true then the access is allowed to continue to memory. Otherwise, the process is trying to access memory that was never allocated to it so the hardware traps to the OS to abort the process (segmentation fault).

## Multiple-Partition Allocation

During the lifetime of multiple processes, **holes** will occur in memory when processes are freed:

```

  +------------------+       +------------------+      +------------------+
  | Operating System |       | Operating System |      | Operating System |
  +------------------+       +------------------+      +------------------+
  | Process  5       |       | Process  5       |      | Process 5        |
  +------------------+       +------------------+      +------------------+
  |                  |  ->   |                  |  ->  | Process 9        |
  | Process 8        |       |     *hole*       |      +------------------+
  |                  |       |                  |      |     *hole*       |
  +------------------+       +------------------+      +------------------+
  | Process 2        |       | Process 2        |      | Process 2        |
  +------------------+       +------------------+      +------------------+

                    Process 8                  Process 9
                      freed                    allocated

```

Holes of various sizes exist scattered throughout memory, so when a new process needs to be allocated the memory from a large enough hole is used. In order to do this the OS stores information about the **allocated partions** and **free partitions (holes)**.

## Dynamic Storage Allocation

There are multiple techniques for satisfying an allocation from a list of free holes:

- **First-fit**: allocate in the first hole that is large enough.
- **Best-fit**: allocate in the smallest hole that is large enough. If the list is not sorted by size, this must search the entire list for the best hole, but it produces the smallest leftover hole.
- **Worst-fit**: allocate in the largest hole. This also requires that the entire list is searched (unless it's sorted), and it produces that largest leftover hole.

Note that first-fit and best-fit are better than worst-fit in terms of **speed** and **storage utilisation**.

## Fragmentation

**Fragmentation** is caused by an inefficient use of memory, which can reduce capacity, performance, or both. There are two types of fragmentation:

- **External fragmentation**: when there is enough total free memory to satisfy an allocation request, but it is not contiguous so the rquest cannot be satisfied.
- **Internal fragmentation**: when a hole may be too small to be used for anything i.e. the allocated memory will always be larger than the capacity of the hole.

There isn't much that can be done about internal fragmentation, but external fragmentation can be reduced by **compaction**.

### Compaction

**Compaction** is when the contents of memory is shuffled around to place all the free memory into one large block. Whilst this can help make the memory more efficient, it can lead to I/O bottlenecks.

## Swapping

The number of process is limit by the physical size of memory. However, many processes may not actually be running, so they are just wasting space. To solve this, they can be **swapped** out of memory to the disk. This frees up memory for new running processes, and they can be brought back into memroy when for continued execution.

This technique requires that swap space is allocated on the disk (either as a file or as a dedicated partition).

The time taken to swap a process in to / out of memory is a major part of the swap time.
