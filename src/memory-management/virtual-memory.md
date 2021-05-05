# Virtual Memory with Paging

## Virtual Memory

**Virtual memory** (logical memory) is used to seperate a user process' logical memory from physical memory. The logical address space of a process can be *much* larger than the physical address space of the machine (on 64 bit architecture, the size of the logical address space is 64 bits). Despite this, only a small amount of this virtual memory needs to be in physical memory for execution.

Virtual memory isn't a contiguous block of memory, rather it is subdivided into fixed sized chunks called **pages**. This makes memory management more flexible, as we no longer need to manage different sized blocks of memory. To keep track of where each virtual page is allocated in physical memory, some kind of translation mechanism (memory map) is required.

This implementation of virtual memory is called **paging**, but there is an alternative implementation called **segmentation** (more on this later).

```

   Virtual            Memory           Physical
   Memory              Map              Memory

  +--------+        +--------+        +--------+
  | Page 0 |------->|        |-*      |        |
  +--------+        +--------+  \     +--------+
  | Page 1 |        |        |   \    |        |
  +--------+        +--------+    \   +--------+
  | Page 2 |        |        |     *->|        |
  +--------+        +--------+        +--------+
  |        |        |        |        |        |
  +--------+        +--------+        +--------+
  |        |                          |        |
  |        |                          +--------+
  |        |                          |        |
  |   .    |                          +--------+
  |   .    |                          |        |
  |   .    |                          +--------+
  |        |                          |        |
  |        |                          +--------+
  |        |
  |        |
  +--------+
  | Page v |
  +--------+

```

Some benefits of virtual memory are:

- Multiple processes can share the same address space e.g. multiple processes can use the address `0x00001234`, but have it point to completely different parts of physical memory.
- It allows for more efficient process creation.

### Virtual Address Space

```

    Max *-------*
        | Stack |
        *-------*
        |   |   |
        |   v   |
        |       |
        |       |
        |   ^   |
        |   |   |
        *-------*
        | Heap  |
        *-------*
        | Data  |
        *-------*
        | Code  |
      0 *-------*

```
