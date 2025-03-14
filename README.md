# NachosOS
Build a simple emulated OS

Nachos internal structure

![alt text](assets/image.png)

An emulated OS is just as any other process running on our system
The main advantage of developping an emulated OS is that, when we have 
an error in our kernel, the host machine doesn't shut down immediately,
like in real os developpement. In our case, we can use gdb to debug our code.


The classes present in Nachos are grouped into different categories, which also appear in the directory structure:

- The kernel, which includes the vital system functionalities: lightweight processes (threads), address spaces, synchronization between threads, scheduling;
- The drivers for the emulated peripherals (disk, console);
- Virtual memory management;
- File management.

The different C++ classes of the system are distributed according to the following directory structure:

- **machine**: classes for hardware emulation (should not be modified);
- **kernel**: main kernel classes;
- **vm**: virtual memory management classes;
- **drivers**: peripheral driver classes;
- **filesys**: file system management classes;
- **utility**: utility classes (lists, statistics, etc.);
- **userlib**: function library for user programs (mini libc);
- **test**: user programs;
- **doc**: documentation.

