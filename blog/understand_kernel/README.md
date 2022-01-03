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

Filesystems must always provide at least the following attributes, which are specified in the POSIX standard:

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

