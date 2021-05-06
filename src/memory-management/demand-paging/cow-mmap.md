# Copy-on-Write (COW) and Memory Mapped Files

Here are a couple of virtual memory tricks that can help improve performance / simplicity.

## Copy-on-Write (COW)

**COW** is a technique which allows a parent process to share its frames with a child processes initially as read-only. Only when either the child or parent process writes to these frames will a copy be made. This saves the cost of having to copy all of the pages initially. As a result, forked processes can be made almost instantaneously, but their initial performance may end up being poor if they begin to write lots of data to memory.

COW works by giving the child process its own page table pointing to the parent's frames that are marked as read-only. When either process attempts to write to one of these frames, the page protection causes a trap to the kernel. Then the kernel allocates each process its own private copy of the frame, and it marks them as read-write.

For example:

```

                       Physical
                        memory
                    *------------*
    Parent          |            |          Child
  *--------*        *------------*        *--------*
  | Page 0 |------->| Read only  |<-------| Page 0 |
  *--------*        *------------*        *--------*
  | Page 1 |--* *-->| Read only  |<--* *--| Page 1 |
  *--------*   X    *------------*    X   *--------*
  | Page 2 |--* *-->| Read only  |<--* *--| Page 2 |
  *--------*        *------------*        *--------*
                    |            |
                    *------------*

```

After the parent / child processes write to pages 0 and 1:

```

                       Physical
                        memory
                    *------------*
    Parent          | Read-write |<--*      Child
  *--------*        *------------*    \   *--------*
  | Page 0 |------->| Read-write |     *--| Page 0 |
  *--------*        *------------*        *--------*
  | Page 1 |--* *-->| Read only  |<---* *-| Page 1 |
  *--------*   X    *------------*     X  *--------*
  | Page 2 |--* *-->| Read-write |    / *-| Page 2 |
  *--------*        *------------*   /    *--------*
                    | Read-write |<-*
                    *------------*

```

## Memory Mapped Files

A **memory mapped file** is a file whose contents on disk have been mapped in the virtual address space of a process. This means when a process goes to read from / write to the file, instead of having to issue system calls through the kernel API it can simply refer to the addresses in memory directly. Through demand paging, these actions will get translated into actual disk accesses. Memory mapped files can provide an elegant programming model for I/O operations 
