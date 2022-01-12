# 1 Introduction

## An Overview of the Unix Filesystem

### Hard and Soft Links

The same file may have several links included in the same directory or in different ones, so it may have several filenames.

Hard links have two limitations:
1. It is not possible to create hard links for directories.
2. Links can be created only among files included in the same filesystem.

To overcome these limitations, soft links (also called symbolic links) were introduced.

Symbolic links are short files that contain an arbitrary pathname of another file. The pathname may refer to any file or directory located in any filesystem; it may even refer to a nonexistent file.

### File Types

* Regular file
* Directory
* Symbolic link
* Block-oriented device file
* Character-oriented device file
* Pipe and named pipe (also called FIFO)
* Socket

### File Descriptor and Inode

All information needed by the filesystem to handle a file is included in a data structure called an `inode`. Each file has its own inode, which the filesystem uses to identify the file.

File systems must always provide at least the following attributes, which are specified in the POSIX standard:

* File type
* Number of hard links associated with the file
* File length in bytes
* Device ID (i.e., an identifier of the device containing the file)
* Inode number that identifies the file within the filesystem
* UID of the file owner
* User group ID of the file
* Several timestamps that specify the inode status change time, the last access time, and the last modify time
* Access rights and file mode

### Access Rights and File Mode

The potential users of a file fall into three classes:
1. The user who is the owner of the fil
2. The users who belong to the same group as the file, not including the owner
3. All remaining users (others)

There are three types of access rights — read, write, and execute — for each of these three classes.

## An Overview of Unix Kernels

### The Process/Kernel Model

Unix kernels do much more than handle system calls; in fact, kernel routines can be activated in several ways:
* A process invokes a system call.
* The CPU executing the process signals an exception, which is an unusual condition such as an invalid instruction.
* A peripheral device issues an interrupt signal to the CPU to notify it of an event such as a request for attention, a status change, or the completion of an I/O operation. Each interrupt signal is dealt by a kernel program called an interrupt handler.
* A kernel thread is executed.

### Process Implementation

When the kernel stops the execution of a process, it saves the current contents of several processor registers in the process descriptor. These include:
1. The program counter (PC) and stack pointer (SP) registers
2. The general purpose registers
3. The floating point registers
4. The processor control registers (Processor Status Word) containing information about the CPU state
5. The memory management registers used to keep track of the RAM accessed by the process

### Synchronization and Critical Regions

Any section of code that should be finished by each process that begins it before another process can enter it is called a `critical region`.

Several synchronization techniques have been adopted. The following section concentrates on how to synchronize kernel control paths.

* Kernel preemption disabling
* Interrupt disabling
* Semaphores
* Spin locks

### Signals and Interprocess Communication

Asynchronous notifications
> For instance, a user can send the interrupt signal SIGINT to a foreground process by pressing the interrupt keycode (usually Ctrl-C) at the terminal.

Synchronous notifications
> For instance, the kernel sends the signal SIGSEGV to a process when it accesses a memory location at an invalid address.

In general, a process may react to a signal delivery in two possible ways:
1. Ignore the signal.
2. Asynchronously execute a specified procedure (the signal handler).

If the process does not specify one of these alternatives, the kernel performs a default action that depends on the signal number.
1. Terminate the process.
2. Write the execution context and the contents of the address space in a file (core dump) and terminate the process.
3. Ignore the signal.
4. Suspend the process.
5. Resume the process’s execution, if it was stopped.

### Process Management

### Memory Management

**Virtual memory**

Virtual memory has many purposes and advantages:
1. Several processes can be executed concurrently.
2. It is possible to run applications whose memory needs are larger than the available physical memory.
3. Processes can execute a program whose code is only partially loaded in memory.
4. Each process is allowed to access a subset of the available physical memory.
5. Processes can share a single memory image of a library or program.
6. Programs can be relocatable — that is, they can be placed anywhere in physical memory.
7. Programmers can write machine-independent code, because they do not need to be concerned about physical memory organization.

**Random access memory usage**

A few megabytes are dedicated to storing the kernel image (i.e., the kernel code and the kernel static data structures).
The remaining portion of RAM is usually handled by the virtual memory system and is used in three possible ways:
1. To satisfy kernel requests for buffers, descriptors, and other dynamic kernel data structures
2. To satisfy process requests for generic memory areas and for memory mapping of files
3. To get better performance from disks and other buffered devices by means of caches

**Kernel Memory Allocator**

A good KMA should have the following features:
1. It must be fast. Actually, this is the most crucial attribute, because it is invoked by all kernel subsystems (including the interrupt handlers).
2. It should minimize the amount of wasted memory.
3. It should try to reduce the memory fragmentation problem.
4. It should be able to cooperate with the other memory management subsystems to borrow and release page frames from them.

Linux’s KMA uses a Slab allocator on top of a buddy system.

**Process virtual address space handling**

The kernel assigns to the process a virtual address space that comprises memory areas for:
* The executable code of the program
* The initialized data of the program
* The uninitialized data of the program
* The initial program stack (i.e., the User Mode stack)
* The executable code and data of needed shared libraries
* The heap (the memory dynamically requested by the program)

**Caching**

### Device Drivers

The kernel interacts with I/O devices by means of device drivers.
* Device-specific code can be encapsulated in a specific module.
* Vendors can add new devices without knowing the kernel source code; only the interface specifications must be known
* The kernel deals with all devices in a uniform way and accesses them through the same interface.
* It is possible to write a device driver as a module that can be dynamically loaded in the kernel without requiring the system to be rebooted.

# 2 Memory Addressing
