# CH 2 Building and Running Modules

## Setting Up Your Test System

## The Hello World Module

- This module defines two functions, one to be invoked when the module is loaded
  into the kernel (hello_init) and one for when the module is removed (hello_exit).
- module_init and module_exit lines use special kernel macros to indicate the role of
  these two functions. Another special macro (MODULE_LICENSE) is used to tell the
  kernel that this module bears a free license; without such a declaration, the kernel
  complains when the module is loaded.
- The kernel needs its own
  printing function because it runs by itself, without the help of the C library. The
  module can call printk because, after insmod has loaded it, the module is linked to
  the kernel and can access the kernel’s public symbols (functions and variables, as
  detailed in the next section).

## Kernel Modules Versus Applications

- While most small and medium-sized applications perform a single task from beginning to end, every kernel module just registers itself in order to serve future requests, and its initialization function terminates immediately.
- As a programmer, you know that an application can call functions it doesn’t define: the linking stage resolves external references using the appropriate library of functions.
- printf is one of those callable functions and is defined in libc.
- Figure 2-1 shows how function calls and function pointers are used in a module to
  add new functionality to a running kernel.
- Another important difference between kernel programming and application pro-
  gramming is in how each environment handles faults: whereas a segmentation fault
  is harmless during application development and a debugger can always be used to
  trace the error to the problem in the source code, a kernel fault kills the current pro-
  cess at least

## User Space and Kernel Space

- A module runs in kernel space, whereas applications run in user space. This concept
  is at the base of operating systems theory.
- The role of the operating system, in practice, is to provide programs with a consis-
  tent view of the computer’s hardware.
- the operating system must
  account for independent operation of programs and protection against unauthorized
  access to resources.
- Unix, the kernel executes in the highest level (also called supervisor
  mode), where everything is allowed, whereas applications execute in the lowest level
  (the so-called user mode), where the processor regulates direct access to hardware
  and unauthorized access to memory.
- We usually refer to the execution modes as kernel space and user space. These terms
  encompass not only the different privilege levels inherent in the two modes, but also
  the fact that each mode can have its own memory mapping—its own address
  space—as well.
- Unix transfers execution from user space to kernel space whenever an application
  issues a system call or is suspended by a hardware interrupt. Kernel code executing a
  system call is working in the context of a process

## Concurrency in the Kernel

- One way in which kernel programming differs greatly from conventional application
  programming is the issue of concurrency
- Linux kernel code, including driver code, must be reentrant—it must be
  capable of running in more than one context at the same time.

## The Current Process

- Kernel code can refer to the current process by accessing the global item current , defined in <asm/
  current.h>
- During the execution of a system call, such as open or read, the current process is the one that
  invoked the call. Kernel code can use process-specific information by using current ,
  if it needs to do so.
- the following statement prints the process ID and the command name of the current
  process by accessing certain fields in struct task_struct :

```
printk(KERN_INFO "The process is \"%s\" (pid %i)\n",
current->comm, current->pid);
```

## Compiling Modules

- Once you have everything set up, creating a makefile for your module is straightfor-
  ward. In fact, for the “hello world” example shown earlier in this chapter, a single
  line will suffice:
  `obj-m := hello.o`
- there is one module to be built from the
  object file hello.o. The resulting module is named hello.ko after being built from the
  object file.

## Loading and Unloading Modules

- After the module is built, the next step is loading it into the kernel. As we’ve already
  pointed out, insmod does the job for you.
- The modprobe utility is worth a quick mention. modprobe, like insmod, loads a module into the kernel. It differs in that it will look at the module to be loaded to see whether it references any symbols that are not currently defined in the kernel.

## Version Dependency

- Bear in mind that your module’s code has to be recompiled for each version of the
  kernel that it is linked to
- Modules are strongly tied to the data structures and function prototypes defined in a particular kernel version
- The kernel does not just assume that a given module has been built against the
  proper kernel version.

## Platform Dependency

- kernel code can be optimized for a specific processor in a CPU family to get the best from the target platform: unlike applications that are often distributed in binary format, a custom compilation of the kernel can be optimized for a specific computer set.
- When a module is loaded, the kernel checks the processor specific configuration options for the module and makes sure they match the running kernel. If the module was compiled with different options, it is not loaded.

## The Kernel Symbol Table

- The table contains the addresses of global kernel items—functions and variables—that are needed to implement modularized drivers. When a module is loaded, any symbol exported by the module becomes part of the kernel symbol table.
- New modules can use symbols exported by your module, and you can stack new modules on top of other modules. Module stacking is implemented in the main-
  stream kernel sources as well: the msdos filesystem relies on symbols exported by the fat module, and each input USB device module stacks on the usbcore and input modules.
- Module stacking is useful in complex projects. If a new abstraction is implemented in the form of a device driver, it might offer a plug for hardware-specific implementations.
- When using stacked modules, it is helpful to be aware of the modprobe utility. As we described earlier, modprobe functions in much the same way as insmod.

## Preliminaries

- module.h contains a great many definitions of symbols and functions needed by loadable modules. You need init.h to specify your initialization and cleanup functions, as we saw in the “hello world” example

## Initialization and Shutdown

- Initialization functions should be declared static , since they are not meant to be visible outside the specific file; there is no hard rule about this, though, as no function is exported to the rest of the kernel unless explicitly requested.
- The use of module_init is mandatory. This macro adds a special section to the module’s object code stating where the module’s initialization function is to be found. Without this definition, your initialization function is never called.

## The Cleanup Function

- Every nontrivial module also requires a cleanup function, which unregisters inter-
  faces and returns all resources to the system before the module is removed. This
  function is defined as:

```
static void __exit cleanup_function(void)
{
/* Cleanup code here */
}
module_exit(cleanup_function);
```

## Error Handling During Initialization

- the simplest action often requires memory allocation, and the required memory may not be available. So module code must
  always check return values, and be sure that the requested operations have actually succeeded.
  The following sample code (using fictitious registration and unregistration func-
  tions) behaves correctly if initialization fails at any point:

```
int __init my_init_function(void)
{
int err;
/* registration takes a pointer and a name */
err = register_this(ptr1, "skull");
if (err) goto fail_this;
err = register_that(ptr2, "skull");
if (err) goto fail_that;
err = register_those(ptr3, "skull");
if (err) goto fail_those;
return 0; /* success */
fail_those: unregister_that(ptr2, "skull");
fail_that: unregister_this(ptr1, "skull");
fail_this: return err; /* propagate the error */
}
```

## Doing It in User Space

The advantages of user-space drivers are:

- The full C library can be linked in. The driver can perform many exotic tasks without resorting to external programs (the utility programs implementing usage
  policies that are usually distributed along with the driver itself).
- The programmer can run a conventional debugger on the driver code without having to go through contortions to debug a running kernel.
- If a user-space driver hangs, you can simply kill it. Problems with the driver are unlikely to hang the entire system, unless the hardware being controlled is really
  misbehaving.
- User memory is swappable, unlike kernel memory. An infrequently used device with a huge driver won’t occupy RAM that other programs could be using,
  except when it is actually in use.
- A well-designed driver program can still, like kernel-space drivers, allow concurrent access to a device.
- If you must write a closed-source driver, the user-space option makes it easier for you to avoid ambiguous licensing situations and problems with changing kernel
  interfaces.
