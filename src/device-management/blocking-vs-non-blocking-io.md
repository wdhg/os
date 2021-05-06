# Blocking vs. Non-Blocking I/O

### Blocking I/O

- I/O calls only return once the operation has completed.
- Processes get suspended as it can only continue once the operation has returned. However from the perspective of the process, the operation will appear instantaneous.
- Easy to understand but leads to multi-threaded code.

### Non-blocking I/O

- I/O calls return as much as is available at that moment (e.g. a `read` with 0 bytes).
- Can be turned on for a file descriptor using the `fcntl` system call.
- Provides application-level polling for I/O.

## Asynchronous I/O

With **asychronous I/O**, a process can execute in parallel with the I/O application. Upon the I/O operation's completion, the process will be notified e.g. through a signal or a callback function. Asynchronous I/O supports checking if / waiting for the I/O operation has completed, and it is very flexible and efficient.

However, asychronous I/O can be harder to use as the process needs to specify which operations should be carried out, which user space buffer results need to be copied, etc (richer API). It can also be potentially less secure as it can cause.
