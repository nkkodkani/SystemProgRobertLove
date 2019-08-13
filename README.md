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

Compiler helps implement C standard and 

APIs and ABIs
