# Linux Virtual Memory

## Linux Virtual Memory Layout

On 32-bit architecture, processes in linux have a virtual address space of 4GB.

```
         0                     896MB
         *-----*-*---------------*---------------------*
  Main   | DMA | |    Normal    |         High         |
  Memory |     | |    Memory    |        Memory        |
         *-----*-*---------------*---------------------*
          \     \ \               \
           \     \ \ The nth page  \
            \     \ \ of the kernel \
             \     \ \ address space \
              \     \ \ maps to the   \
               \     \ \ nth page      \
                \     \ \ frame of main \
                 \     \ \ memory.       \
                  \     \ \               \
                   \     \ \               \
          *---------*-----*-*---------------*-----------*
  Virtual |  User   |     | |               | On-demand |
  memory  | Process |     | |               | mapping   |
          *---------*-----*-*---------------*-----------*
          0        3GB                                 4GB
                    |---------896MB---------|
                    |-------Kernel address space--------|

```

In this diagram:

- The bottom diagram is the "view" that an executing user process would have of its virtual address space.
- The first 3 gigabytes in the virtual memory is where the code, stack, and heap are allocated normally.
- Processes map the kernel to the 3-4GB virtual address range, which allows user processes to make system calls without a TLB flush (efficient).
- The kernel maps the lower 896MB of physical memory to its virtual address space.


## Paging

- On x86-32 (IA-32) linux uses:
  - a 4KB page size
  - a 4GB virtual address space
  - typically a two-level page table.
- On x86-64 linux uses
  - a larger page sizes (e.g. 4MB)
  - up to a four-level page table.

The two-level page table (which can be replaced with a three-level page table with Physical Address Extension (PAE)) uses the offset bits in the addresses to store the page status (dirty, read-only, etc).

x86 has a particular terminology for the levels of the page table

- **Page global directory**: the outer page table.
- **Page table**: the inner page tables.

**TODO DRAW DIAGRAM**

## Meltdown Attack

Meltdown is an attack that allows a user program to access the kernel address space mapped in its virtual memory, bypassing the page protection that only allows access to this memory in privileged mode.

```c
s = a + b;
t = s + c;
u = t + d;
v = u + e;

if (v == 0) {
  w = kernel_mem[addr]; // illegal access, will page fault
  // the following is not run
  x = w & 0x01;
  y = x * 4096;
  z = user_mem[y];
}
```

The code above should not run after the page fault, however due to **speculative execution** this is not the case.

### Speculative Execution

**Speculative execution** is an optimization technique where a computer will execute some code concurrently before knowing if it is needed or not. The relevent use case of speculative execution for the meltdown attack is **branch prediction**.

The CPU figures out that it can run the code outside the if block and inside the if block concurrently, as there is no data dependency between them. However, the effects of speculative execution cannot be made visible (e.g. through updating memory or register contents directly) as it is unknown whether or not the branch will be taken yet. Therefore, the results are hidden in intermediate variables `w_`, `x_`, `y_`, and `z_`.

```c
/* the branch predictor thinks the v == 0 branch will,
 * so it speculatively executes it
 */
w_ = kernel_mem[addr];
x_ = w & 0x01;
y_ = x * 4096;
z_ = user_mem[y_];
```

```c
s = a + b;
t = s + c;
u = t + d;
v = u + e;

if (v == 0) {
  // assign hidden values
  w = w_; x = x_; y = y_; z = z_;
}
```

When the speculative execution runs the code, it will be an illegal access but it cannot fault yet as the effects cannot be made visible yet (the protection level could change by the time the branch is reached). Therefore it continues executing. If `v == 0`, then the fault will occur and `w = w_; x = x_; y = y_; z = z_;` won't run.

At first, CPU manufacturers (e.g. Intel) thought this was safe as the protected kernel memory was never revealed.

Going over the speculatively executed code:

```c
w_ = kernel_mem[addr]; // retrieve data from kernel memory
x_ = w & 0x01;         // mask out 1 bit from this data. x_ = 0 or x_ = 1
y_ = x * 4096;         // multiply. y_ = 0 or y_ = 4096
z_ = user_mem[y_];     // access user memory at either index 0 or 4096
```

By initially arranging that `user_mem[0..4096]` is not cached, the access `user_mem[y_]` will cause either `user_mem[0]` or `user_mem[4096]` to be in cache after being speculatively executed. A program then can time how long it takes to access `user_mem[0]` and `user_mem[4096]`. If `user_mem[0]` is significantly faster then it must be in cache so `x_` was a `0`, otherwise if `user_mem[4096]` is significantly faster then `x_` was `1`. Thus the program has learned the value of a single bit in kernel memory.

Generalising this attack to read many bits could allow a program to leak larger amounts of memory from the kernel. In practice this attack is quite effective, allowing for a program to leak arbitrary amounts of kernel memory, which could include the root password.

### The Fix

Recent versions of Linux fix this by moving kernel memory to a separate virtual address space. This makes system calls more expensive, but it prevents any program from executing this attack.
