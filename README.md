# SystemProgRobertLove
Notes from System programming book, Robert love

compile a source code:
gcc -Wall -Wextra -O2 -g -o snippet snippet.c

# Chapter 1 (Introduction and Essential Concepts)

System programming -> Systems software directly talks with kernel and core system libs
Example: Shell, text editor, compiler and debugger, core utilities, system daemons, network and web server and database

Majority of Unix/Linux Code is written at system level in C/C++

Traditional system programming applications: Apache, Bash, cp, emacs, gcc, gdb, glibc, ls, mv, vim 

Major connectors of system programming:
- System calls
- C Lib
- C Compiler

1. System calls(syscalls)

- Functional invocation made from user space to kernel in order to request some service/resource from OS
- Linux implements fewer system calls approx (300) compared to (1000s) on Windows
- Not possible to directly link user space application with kernel space due to security and reliabilty
- Indeed the kernel must provide mechanism by which user application can signal kernel that it wish to invoke a system call
- Application can then trap into the kernel through kernel well defined mechanism & execute only code that kernel allows it to execute
- Exact mechanism varies from architecture to architecture
Example: i386 => user space application execute software interrupt instruction "int" with the value of 0x80

- Application tells kernel which syscalls to execute & with what parameters via maachine registers
- syscalls are denoted by number starting at zero. On x386 architecture, if we request syscall 5 (open ()) user space application puts 5 in register eax before issuing int instruction
- Parameter passing on i386 register is used for each possible parameter register in order of (ebx, eex, edx, esi and edi)
- If syscall has more thab 5 parameters then single register is used to point a buffer in user space where all parameters are stored

2. C Library (libc/ GNU -> glibc)

- Heart of unix application
- Even while programming in another language C lib is wrapped by higher level lib providing core services & facilate system call invocaton 

glibc -> also provides wrapper for syscalls/thread support

3. C Compiler (gcc)

Compiler helps implement C standard and system ABIs 

APIs and ABIs

API (Application Programming Interface)
- Defines interface by which one piece of software communicates with another source level
- Example: API is the interface defined by C standard & implemented by standard C Lib
- Ensures source compatibility

ABI (Application Binary Interface)
- Ensures Binary compatibility guranteeing that piece of object code will function on any system with same ABI without requiring recompilation
- ABI tied to architecture majority of API speaks machine specific concept - registers or assembly instructions
- Each machine architecture has its own ABI on linux
- Thus ABI is function of both OS (linux) and architecture (x86)
- ABIs are concerned with issues such as calling conventions, byte ordering, register use, system call invocation, linking, library behavior, and the binary object format. The call‐ ing convention, for example, defines how functions are invoked, how arguments are passed to functions, which registers are preserved and which are mangled, and how the caller retrieves the return value   

Standard -> Linux aims towards compliance of two most important standards POSIX (Portable Operating System Interface) and SUS (Single Unix Specific)

Files/Filesystem:

- Linux: everything is file
- In order to be accessed file must be first opened -> operations (read/write/both)
- Open file -> referenced by unique desciptor mapping from metadat associated with open file back to specific file itself
- Kernel has descriptor (handled by integer) -> known as file descriptor
- Fds are shared with user space and used directly by user program to access files

Regular File:
- Regular file contains byts of data organized into linear array called a byte stream
- In Linux no further organization or formatting such as records is specified for a file
- When file is first opened, file position is zero it is increased as bytes in file are read/written subsequently
- File position can be set beyond EOF but it will cause padding of zeros after EOF & before file position
- Maximum file length as with the max file position is bounded by limites of sizes of C type that linux kernel uses to manage file
- A single file can be opened by different or same process multiple times. Open(open()) instance of file gives unique file descriptor
- Kernel doesn't impose restriction on concurrent file access. Ordering rely on individual file operation
- Although files are accesssed via filename they are not directly associated with such name instead referenced by inode -> integer value unique to file system and not whole system
- Inode -> had inode number, modification timestamp, owner, type, length, location of file (BUT NO FILENAME is stored in inode)
- Inode - Physical object located on disk + conceptual entity represented by data structure in Linux Kernel

Directories & Links
- Files are opened by name and node inode number in user space
- Conceptually directories are viewed like a normal file with difference that it contains only mapping of name to inode 
- Working: When usser space application requests to open a file, kernel opens a dirctory containing filename and searches for given name. From filename kernel obtains inode number from which inode structure is found. Inode contains metadata which includes ondisk location of file data
- How does kernel know 





