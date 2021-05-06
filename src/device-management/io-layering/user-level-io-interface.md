# User-Level I/O Interface

At this level, the actual kinds of operations that a process may want to use are exposed. These include, for example, `open`, `close`, `read`, `write`, and `seek` for a disk. I/O libraries also configure other types of I/O paramters, e.g. blocking or non-blocking I/O, synchronous or asychronous I/O, etc.

The user-level I/O interface must provide some kind of abstraction of how to expose devices. In the case of Unix, this is done by representing them as files (as per the Unix philosophy).
