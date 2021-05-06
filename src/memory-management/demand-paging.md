# Demand Paging

**Demand paging** is a technique where pages are only brought into memory when they are required. This means that when a program is launched, the executable pages are only loaded into memory when they are accessed instead of loading all of them eagerly at the start.

This has the following benefits:

- it reduces the I/O load because when an application is launched, the loading period is much shorter.
- less memory will be used if there are parts of the executable that are never run.
- there is a faster response time for larger binaries as there is no need to wait for it to be fully loaded into memory.

Overall, memory is used much more efficiently.

It works by using the valid-invalid bit:

1. Initially, set all the valid-invalid bits to `0` (out of memory).
2. During address translation, if this bit is `0` then we trap to the kernel and generate a page fault.
3. The OS then checks another table to decide if the reference is valid or invalid. If the reference is invalid then the program is aborted. Otherwise, the reference is valid but the page is not loaded into memory.
4. The OS then handles the request by:
    1. getting an empty frame.
    2. swapping the page into the frame.
    3. updating the page table by setting the valid-invalid bit to `1`.
    4. restarting the last instruction.

Once the page has been reloaded and the valid-invalid bit is set, then the re-run access will be valid.

```

    Virtual          Page             Physical
    Memory           Table             Memory
  *--------*       *---*---*         *--------*
  | Page 0 |     0 | 1 | v |       0 |        |
  *--------*       *---*---*         *--------*
  | Page 1 |     1 |   | i |       1 | Page 0 |
  *--------*       *---*---*         *--------*
  | Page 2 |     2 |   | i |       2 |        |
  *--------*       *---*---*         *--------*
  | Page 3 |     3 | 7 | v |       3 |        |
  *--------*       *---*---*         *--------*
                     |   |         4 |        |
                     *   |           *--------*
                    /    *         5 |        |
               frame    /            *--------*
                       /           6 |        |
               Valid-invalid         *--------*
                   bit             7 | Page 3 |
                                     *--------*

```

## Performance of Demand Paging

Let the **page fault rate** `p` be a fraction between `0` and `1`:

- `p == 0` means there are no page faults.
- `p == 1` means every access causes a page fault.

The **effective access time** (`EAT`) is then defined as:

```
EAT = (1 - p) * memory access time
      + p ( page fault overhead time
          + swapping page out time [only if a page needs to be swapped out]
          + swapping page in time
          + restarting the instruction overhead
          )
```

For example:

- Memory access time = `1µs`
- Pages need to be swapped out `50%` of the time.
- Page swap time = `10ms` = `10,000µs`

Therefore (assuming no overhead):

```
EAT = (1 - p) * (1µs)
      + p ( 0.5 * 10,000µs [half of cases we need to swap out a page]
          + 10,000µs
          )
    = (1 - p) * 1µs + p * 15,000µs
    ≈ 1µs + p * 15,000µs
```

So it is obvious that if `p` is large, then the effective access time will be very large.

There are some cases where the page fault rate `p` increases, such as if the system is out of physical memory then pages will have to constantly be swapped in and out of memory (hence why a machine that is out of memory performs poorly). This situation is known as **thrashing**.
