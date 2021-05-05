# Segmentation

Paging only gives a one-dimensional virtual address space, however what if there were seperate address spaces for code, data, and the stack?

A **segment** is an idependant address space from 0 to some maximum which:

- can grow / shrink independently.
- can support different kinds of protection such as read / write / execute.
- unlike pages, programmers can be aware of segments.

**Segmentation** is an old mechanism for managing memory. Initially it was used for shared libraries, where a shared library was represented as a segment w hich could then be accessed by different processes. Nowadays paging is much more popular, but segmentation still exists on many architectures (e.g. x86) for legacy reasons.

Memory allocation of segments can be harder due to their variable size, which may cause external fragmentation.

## Hybrid Segmentation / Paging

x86 has segmentation support because it started out with segmentation before adopting paging later. However, x86 actually has a hybrid of the segmentation and paging models.

x86 used segmentation when it was 32-bit (IA-32 / x86-32).

**TODO DRAW DIAGRAM**
