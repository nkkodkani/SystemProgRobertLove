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
- Working: When usser space application requests to open a file, kernel opens a dirctory containing filename and searches for given name. From filename kernel obtains inode number from which inode structure is found. Inode contains metadata of ondisk location of file data
- How does kernel know which directory to look in to find a given filename? 
- Initially theire is only one directory - "root" - path "/"
- Directories similar to regular file have inode
- Links inside directories can point to inode of other directories. It is kind of nests/hierarchy of directories  which allows for use of pathname
- Working: a) Absolute Path(starts at ROOT): When kernel is asked to open a pathname "/home/blackboard/concode.png" it walks each directory entry (called dentry inside of kernel) in pathname to find inode of next entry. In example, it first gets inode / (root) then gets the inode of further directories. This operation is called as name path resolution.
- Linux kernel employs cache - "Dentry Cache" - stores results of directory resolution providing speedier lookup in future
- Relative Path (path at current directory) -> In relative path kernel starts name resolution from current directory
- Although directories are treated like normal files, kernel  does not allow them to be opened and manipulated like regular files instead they must be manipulated using special set of system calls. These system calls allow for the adding and removing of links. If user space were allowed to manipulate directories without kernel's mediation it would be too easy for a single simple error to correup he filesystem

Hard Links
- When multiple links map different names to the same inode we call them hard links
- Hardlink allow complex filesystem structure with multiple pathname point to the same data
- hardlink can be in same directory or 2 or more different directory
- deleting a file involves unlinking from directory structure which is done by simply deleting its name and inode pair from directory
- Filesystem cannot destroy inode and associated data on very unlink operation. WHat if it had link existed elsewhere in filesystem?
- So only when link count is zero in the inode, then the associated data is actually removed from the filesystem

Symbolic links

- Hard links cannot span filesystems because an inode no is meaningless outside of inode's own fs - so to resolve this UNIX has symbolic link
- Symbolic linlinked-to fileek looks like regular files
- Symlink has its own inode and data chunck which contains complete pathname of linked-to-file
- That means sym link can point anywhere even when files or directories that do not exists (broken link)
- symlink has overhead - resolve inode of symlink & file linked to

Special Files

- Special files are kernel object that are represented as files
- Linux supports four: block device files, character device files, named pipes and Unix domain sockes
- Special files purpose => let certain abstraction fit into the file
- Device accessed via device files and normal operations sunch as open/read/write/allow user spaces access and manipulate device
- Character Device:
  1. It is accessed by linear queue of bytes
  2. Devide driver places bytes onto queue and user reads the bytes in order they are placed on queue
  When new queue character is left read then device returns EOF
  
- Block device:
  1. Accessed by array of bytes
  2. The device driver maps the bytes over seekable device and user is free to access any valid bytes in the array, it might read byte 12, then byte 7, and then byte 12
  
- Named pipes (FIFO)
 1. IPC mechanism that provides a communiction channel over a file descriptor accessed via a special file
 2. Regular pipes pipe output of one program to imput of another
 3. Named pipes are similar like  regular pipes but are accessed via a file called FIFO special file
 
 Filesystem and namespaces:
 
 - Filesystem is a collection of files and directories in a formal and valid hierarchy, they can be added/removed from global namespace of files and directories and these operations are called mounting and unmounting
 - Filesystem usually exits physically but linux also supports virtual filesystem that exists only in memory and network filesystem that exists on machine across the network
 - Smallest addressable unit on bloack device is sector. Sector is physical attribute of the device which comes in powers of tow with 512 being common. Block devive cannot transfer or access a unit of data smaller tha a sector and all I/O must occur in terms of one or more sectors
 - Smallest logically addressable unit on the filesystem is the block. Block is the abstraction of the filesystem not on physical media.
 - Linux, blocks are larger than sector but must be smaller than page size. Page size is smallest unit addressable by memory management unit a hardware component. Common block size are 512 bytes, 1 kb and 4 kb
 
 Processes:
 
 - Process are object code in execution: active, running programs
 - More than object code - process consists of data, resources, state and virtualized computer
 - Processes begin life as executable object code which is maachine-runnable code in executable format that the kernel understands.
 - Most common format in Linux - "Executable and Linkable Format (ELF)". Executable format contains metadata and multiple sections of code and data.
 
 
 
 
 
 
 



