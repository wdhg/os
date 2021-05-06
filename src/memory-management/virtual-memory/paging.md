# Paging

Paging allows for the logical address space to be noncontiguous, so a process can allocate physical memory when it is needed / available.

Paging has two important concepts, **frames** and **pages**:

- **Frames**: a fixed-size block of physical memory.
- **Pages**: a block of logical (virtual) memory, the same size as a frame.

A typical page size is 4KB.

When a program of n pages is run:

1. n free frames are found.
2. the program is loaded into this memory.
3. a **page table** is setup to translate logical addresses to physical addresses.

## Page Table

A page table is a translation mechanism from logical memory to physical memory.

```

    Virtual        Page         Physical
    Memory         Table         Memory
  *--------*       *---*       *--------*
  | Page 0 |     0 | 1 |     0 |        |
  *--------*       *---*       *--------*
  | Page 1 |     1 | 4 |     1 | Page 0 |
  *--------*       *---*       *--------*
  | Page 2 |     2 | 3 |     2 |        |
  *--------*       *---*       *--------*
  | Page 3 |     3 | 7 |     3 | Page 2 |
  *--------*    /  *---*       *--------*
               /     |       4 | Page 1 |
              /      |         *--------*
           Page      |       5 |        |
          Number     |         *--------*
                     *       6 |        |
                    /          *--------*
               Frame ------- 7 | Page 3 |
               Number          *--------*

```

## Address Translation

When translating an address, the page table can be used to find the frame that the data exists in, but the offset inside the frame is also required. This offset is equal to the offset inside the page as they are equal size.

Logical addresses can be split into two sections:

- **Page number** `p`: used as the index to the page table. The page table only stores the base address of the page in physical memory.
- **Page offset** `d`: combined with the base address of the page, the full physical address can be sent to the memory unit.

Given a logical address space `2^m` and page size `2^n`:

```

    Page number | Page offset
  *-------------*-------------*
  |      p      |      d      |
  *-------------*-------------*
     m-n bits       n bits

```

For example, if `m = 32` and `n = 12`, the address `0x12345678` has a page number of `0x12345` and an offset of `0x678`.

## Free Frames

To keep track of which frames are free, we can use a free frames list. Whenever a process needs to allocate new pages, this list can be searched for free frames, which can be removed and entered into the process' page table.

## Memory Protection

To protect from invalid accesses of logical memory (e.g. reading from an unallocated page), each entry in the page table is associated with a **valid-invalid protection bit**.

- **Valid**: indicates a legal page (page is in the process' logical address space).
- **Invalid**: indicates the page is missing. This can be due to:
  - the page is not in the process' logical address space (page fault).
  - needing to load the page from disk (see demand paging).
  - an incorrect access (e.g. a programming error).

## Page Table Implementation

Page tables can be quite large, so as a result it must be kept in main memory. Typically, the MMU will have two registers:

- **Page-table base register (PTBR)**: points to the base of the page table.
- **Page-table length register (PTLR)**: stores the size of the page table.

However, this implementation has a large problem: it is inefficient. Every data / instruction access will require two memory access: one for the page table and one for the data / instruction.

### Associative Memory

To solve this ineffciency, a fast-lookup cache can be used. This cache is implemented in hardware as associative memory, and it can be thought of as a mapping from page number to frame numbers. 

```

    Page number   Frame number
  *-------------*--------------*
  |      0      |      4       |
  *-------------*--------------*
  |      1      |      2       |
  *-------------*--------------*
  |      2      |      6       |
  *-------------*--------------*
  |      3      |      9       |
  *-------------*--------------*

```

When translating from page number `p` to frame number `d`, firstly the cache is checked to see if `p` is in any associative register. If it is, then get `d` out of it. Otherwise, get `d` from the page table in memory.

### Translation Look-aside Buffers (TLBs)

These associative caches are refered to as Translation look-aside buffers. TLBs usually need to be flushed out after a context switch as pages are process specific. This can lead to poor performance / substantial overhead after the context switch as the cache needs to warm up first. To solve this, some TLBs store **address-space ids (ASIDs)** in entries, which are used to uniquely identify each process to provide address-space protection for that process. This allows the TLB to be shared across processes.

Previously, kernel memory was mapped into the virtual address space of each process which is more efficient as these pages didn't have to change in the TLB on a system call. However, more recently kernel memory has been moved to a seperate virtual address space for security reasons. This means that the TLB may have to be changed / flushed on a system call.

### Performance: Effective Access Time

- Associative Lookup = `ε`

- Assume memory cycle time = `1µs`

- **Hit ratio** `α`:
  - the fraction (between 0 and 1) of times that a page is found in associative registers.
  - this ratio is related to the number of associative registers.

Therefore, the **effective access time (EAT)** is given by:

```
    EAT = (ε + 1)α + (ε + 2)(1 - α)
        = 2 + ε - α
```

- `(ε + 1)` is the time of a TLB hit.
- `(ε + 2)` is the time of a TLB miss.


## Page Table Types

1. **Hierarchical** page table.
2. **Hashed** page table.
3. **Inverted** page table.

### 1. Hierarchical Page Table

In a traditional page table, the allocated array must be proportional to the size of the virtual address space. This is very waseful as the virtual address space is usually only sparsely populated. **Hierarchical page tables** instead exploit this fact.

#### Two-Level Paging

**Two-level paging** work by using an **outer page table** in conjunction with many inner page tables. The outer page table doesn't map to frame numbers, instead it maps to an address (page address) of an inner page table.

```

   Outer page           Page table              Physical
     table                                       memory
  *----------*        *-------------*          *--------* 0
  |          |\       |  *-------*  |          |  ...   |
  *----------* *------|->|   1   |\ |          *--------* 1
  |          |        |  *-------* *|--------->|########|
  |          |        |  |       |  |          *--------*
  |   ...    |        |  |  ...  |  |          |        |
  |          |        |  |       |  |          |  ...   |
  |          |        |  *-------*  |          |        |
  *----------*        |  |  500  |--|--*       *--------* 100
  |          |-*      |  *-------*  |   \  *-->|########|
  *----------*  \     *-------------*    \/    *--------*
                 \    |     ...     |    /\    |  ...   |
                  \   *-------------*   /  \   *--------* 500
                   \  |  *-------*  |  /    *->|########|
                    *-|->|  100  |--|-*        *--------*
                      |  *-------*  |          |  ...   |
                      |  |       |  |          *--------* 708
                      |  |  ...  |  |     *--->|########|   \
                      |  |       |  |    /     *--------*    \ Frame
                      |  *-------*  |   /      |  ...   |      Number
                      |  |  708  |--|--*       *--------*
                      |  *-------*  |
                      *-------------*

```

Note in this diagram, the each entry in the page table is an inner page table, mapped to by the outer page table.

This means that for any region where there are no allocated pages in virtual memory, there won't be an inner page table allocated.

Remember that a logical address is divided into a page number and page offset. Now that the page table is now paged, the page number is divided up further:

- `p1`: index into the outer page table.
- `p2`: displacement into the page of inner page table.

```

    page number | page offset
  *------*------*-------------*
  |  p1  |  p2  |      d      |
  *------*------*-------------*
     |
     |     *------* 0
     |     |      |
     |     *------* p1
     *---->|######|------>*------* 0
           *------*       |      |
           |      |       *------* p2
           *------*       |######|------>*------* 0
                          *------*       |      |
                          |      |       *------* d
                          *------*       |######|
                                         *------*
                                         |      |
                                         *------*


```

Note that now on a TLB miss, there are now *two* extra lookups instead of just one (this increases with higher levels).

#### Three Level Paging

In some cases, two levels of paging may not be enough. Therefore, there also exist schemes where the level of paging increase to 3 or even 4. The principle remains the same.

```
   2nd outer    Outer      Inner   | Page
    page        page       page    | offset
  *----------*----------*----------*--------*
  |    p1    |    p2    |    p3    |   d    |
  *----------*----------*----------*--------*
```

#### Page Table Size

The size of the page table grows with the size of the virtual address space. Assuming a 4KB page size:

- on a 32-bit machine, the page table will be at least 4MB per process (not too bad).
- on a 64-bit machine, the page table will need 2^52 entries. With 8 bytes per entry, that's 30 million GB per process.

One idea to resolve this is to instead stores entries per *frame* rather than per page. This makes more sense as the page table will grow linearly with the total number of physical frames on the machine. Hashed and inverted page tables do this.

### 2. Hashed Page Table

A **hashed page table** works by using a hash table to to map from virtual addresses to physical addresses.

If there are conflicts in a particular hash bucket, then each entry is chained so they can be searched until the matching virtual page number is found.

This implementation is much more complex. Since this needs to be implemented in hardware (e.g. to traverse the chain in a bucket), the MMU would require a lot more complexity. The effectiveness of this implementation also greatly depends on the quality of the hash function, and how many conflicts there are.

**TODO DRAW DIAGRAM**

### 3. Inverted Page Table

In an **inverted page table**, the page tables entries consist of:

- the virtual address of the page stored in the memory location.
- information about the process which owns the page.

The index of each entry in the page table is equal to the frame number.

This method decreases memory needed to store the page table, but it also increases the time required to search the table when a page reference occurs (however, this may not be too bad as long as there is a very high hit ratio for the TLB).

**TODO DRAW DIAGRAM**

## Shared Memory

A process can set up shared memory areas by simply mapping a page from each process' virtual memory to the same physical frame. This means that when either process goes to access that page, they are infact accessing the same memory. This has the benefit that once the memory is shared, there is no need for any kernel involvment, however it is less useful for unidirectional communication due to the lack of synchronisation.

**TODO DRAW DIAGRAM**

### System V API

- `shmget`: allocates a shared memory segment.
- `shmat`: attaches a shared memory segment to the address space of a process.
- `shmctl`: changes the properties associated with a shared memory segment.
- `shmdt`: detaches a shared memory segment from a process.
