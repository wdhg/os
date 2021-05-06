# Linux I/O API

## Linux I/O Classes

Linux distinguishes operations in terms of the class of device:

- **Character** (unstructured): files and devices.
- **Block** (structured): devices.
- **Pipes** (message): interprocess communication.
- **Socket** (message): network interface.

### Sockets

Sockets are an I/O abstaction that allows for **bidirectional** communication. They can be used between two processes on a single machine, or between two devices over a network.

Sockets are their own I/O class as network communication can be quite quite different in comparison to the other classes. For example, sockets can send variable amounts of data (packets) whereas block devices cannot. Sockets also may suffer from packet loss, which needs to be handled in their protocols (TCP).

There are two types of sockets:

- **TCP** (stream sockets).
- **UDP** (datagram sockets).

## Linux I/O API

```c
int create(const char *path, )
fd = create(filename, permission);
```

`create` opens a file for reading / writing. `fd` is the index to the file descriptor, `permission` is used for access control.

A file descriptor is like a handle that can be used in subsequent system calls by the user process to refer to that particular file or device that the process wants to operate on.

```c
fd = open(filename, mode);
```

`open` opens a file. `mode` is:

- `0` for read.
- `1` for write.
- `2` for read / write.

```c
close(fd);
```

`close` closes the file or device referenced by `fd`.

```c
numbytesread = read(fd, buffer, numbytes);
```

`read` reads `numbytes` from a file or device referenced by `fd` into `buffer`. It returns the number of bytes actually read as `numbytesread`.

```c
numbyteswritten = write(fd, buffer, numbytes);
```

`write` writes `numbytes` from `buffer` to the file or device referenced by `fd`. It returns the number of bytes actually written as `numbyteswritten`

```c
pipe(&fd[0]);
```

`pipe` create a pipe. `fd` is an array of two intergers: `fd[0]` is for reading, `fd[1]` is for writing.

```c
newfd = dup(oldfd);
dup2(oldfd, newfd);
```

`dup` and `dup2` duplicate a file descriptor.

```c
ioctl(fd, operation, &termios);
```

`ioctl` is used to control devices e.g. `&termios` is an array of control characters.

```
fd = mknod(filename, permission, dev);
```

`mknod` creates a new special file e.g. a character or block device.

### Linux: File Descriptors

Each process has its own **file descriptor table**. When a process is created, it has 3 file descriptors:

```

  *-----------------*----------------*
  | File descriptor | input / output |
  *-----------------*----------------*
  |        0        |     stdin      |
  *-----------------*----------------*
  |        1        |     stdout     |
  *-----------------*----------------*
  |        2        |     stderr     |
  *-----------------*----------------*

```

By default, all three file descriptors refer to the terminal from which the program was started.

Additional file descriptors are added when files or special files (devices) are opened.

### Linux: I/O Example

```c
#include <stdlib.h>

#define BUFSIZE 512

int main(int argc, char **argv) {
  char buffer[BUFSIZE];

  int stdin = 0;  // standard input always corrosponds to fd = 0
  int stdout = 1; // standard output always corrosponds to fd = 1
  int stderr = 2; // standard error always corrosponds to fd = 2

  // open a file passed as an argument to the program
  int fd = open(argv[1], O_RDONLY);

  // validate that file exists
  if (fd < 0) {
    write(stderr, "Can't open file", 15);
    return 1;
  }

  int n; // number of bytes read from stdin

  do {
    // read in some text from the user
    n = read(fd, buffer, BUFSIZE);

    // validate user input was read okay
    if (n < 0) {
      write(stderr, "Error while reading", 19);
    } else {
      write(stdout, buffer, n);
    }
  } while (n > 0);

  // close the file at the end
  close(fd);
}
```

### Linux: Asynchronous I/O Example

```c
#include <aio.h>

// ...

  // aiocb stands for "asychronous I/O control block"
  struct aiocb my_aiocb;

  int fd = open("myfile", O_RDONLY);

  // allocate buffer for aio request
  my_aiocb.aio_buf = malloc(BUFSIZE + 1);

  // initialise aio control structure
  my_aiocb.aio_fildes = fd;
  my_aiocb.aio_nbytes = BUFSIZE;
  my_aiocb.aio_offset = 0;

  // intialise read request
  int ret = aio_read(&my_aiocb);

  // wait for read to finish (more useful to do something else)
  // also possible to register a signal notification of thread callback
  while (aio_error(&my_aiocb) == EINPROGRESS) {}

  int ret = aio_return(&my_iocb);

  // check results from read
  if (ret > 0) {
    // successfully read ret bytes
  } else {
    // read failed, check errno
  }

// ...

```
