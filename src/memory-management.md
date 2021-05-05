# Memory Management

Memory management is a key component of the computer. Each instruction cycle involves a memory access.

## Overview

- Basic concepts:
  - Memory allocation.
  - Swapping.
- Virtual memory:
  - Paging
- Demand paging:
  - Page replacement algorithms.
  - Working set model.
- Linux memory management.


## Memory Hierarchy

```

  +--------------+--------------+---------------------------+
  | Type / Level | Typical Size | Latency                   |
  +==============+==============+===========================+
  | Registers    | Bytes        | 1 CPU clock cycle or less |
  +--------------+--------------+---------------------------+
  | L1 Cache     | KB           | ~1ns                      |
  +--------------+--------------+---------------------------+
  | L2/L3 Cache  | MB           | ~10ns                     |
  +--------------+--------------+---------------------------+
  | Memory       | GB           | ~100ns (many cycles)      |
  +--------------+--------------+---------------------------+
  | SSD          | GB / TB      | ~25µs read / ~250µs write |
  | Disk         | GB / TB      | ~1-10ms                   |
  +--------------+--------------+---------------------------+
  | Network      | PB           | ~10-1000ms                |
  +--------------+--------------+---------------------------+

```

Note:

- Cache sits between the main memory and the CPU registers.
- SDD and Disk are at the same level.
