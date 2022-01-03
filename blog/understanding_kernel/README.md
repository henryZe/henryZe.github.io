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

All information needed by the filesystem to handle a file is included in a data structure called an inode. Each file has its own inode, which the filesystem uses to identify the file.







