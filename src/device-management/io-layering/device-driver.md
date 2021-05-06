# Device Driver

A device driver / handler handles one device type, but it may control multiple devices of the same type. They must:

- implement block reading / writing.
- access device registers.
- initiate operations.
- schedule requests (e.g. managing a queue of outstanding requests).
- handle errors (e.g. reporting corrupted blocks on a disk).

