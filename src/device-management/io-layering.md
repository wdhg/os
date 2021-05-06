# I/O Layering


```

           *----------------------------------------------*
           |                 *---------*                  |
     User  |                 | User    |                  |
     Space |                 | Program |                  |
           |                 *---------*                  |
           *----------------------|-----------------------*
           |                      V                       |
           | *------------------------------------------* |
           | |       Rest of the operating system       | |
    Kernel | *------------------------------------------* |
    Space  |       |              |              |        |
           |       V              V              V        |
           | *------------* *------------* *------------* |
           | |  Printer   | | Camcorder  | | CD-ROM     | |
           | |  driver    | | driver     | | driver     | |
           | *------------* *------------* *------------* |
           *-------|--------------|--------------|--------*
                   V              V              V
             *------------* *------------* *------------*
  Hardware   | Printer    | | Camcorder  | | CD-ROM     |
             | controller | | controller | | controller |
             *------------* *------------* *------------*
                   |              |              |
                   V              V              V
   Devices     [Printer]     [Camcorder]      [CD-ROM]

```

- Controllers are implemented in hardware.
- Interfacing with controllers is done by device drivers in kernel space.
- The rest of the OS provides different layers of abstraction on top of the interface provided by the device drivers.
- User programs (processes) then use the OS interfaces.

## I/O Layering Overview

In order, from high level to low level:

1. User-level I/O software (typically libraries that translate the lower level system call interface that the OS provides into a higher level one, typically with more convenient abstractions).
2. Device-independent OS software.
3. Device drivers.
4. Interrupt handlers (so the device can notify the CPU when it has finished an operation).
5. Hardware.

