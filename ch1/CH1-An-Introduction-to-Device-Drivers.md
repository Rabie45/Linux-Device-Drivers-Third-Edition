# CH 1 An Introduction to Device Drivers
- One of the many advantages of free operating systems, as typified by Linux, is that
their internals are open for all to view.
- Device drivers take on a special role in the Linux kernel. They are distinct “black
boxes” that make a particular piece of hardware respond to a well-defined internal
programming interface; they hide completely the details of how the device works.
- This book teaches you how to write your own drivers and how to hack around in
related parts of the kernel.

## The Role of the Device Driver
- Most programming problems can indeed be split into two parts: “what capabilities are to be provided” (the mechanism) and “how those capabilities can be
used” (the policy).
- When writing drivers, a programmer should pay particular attention to this fundamental concept: write kernel code to access the hardware, but don’t force particular policies on the user, since different users have different needs.
- The driver should deal with making the hardware available, leaving all the issues about how to use the hardware to the applications.
- You can also look at your driver from a different perspective: it is a software layer
that lies between the applications and the actual device.
- Policy-free drivers have a number of typical characteristics. These include support for both synchronous and asynchronous operation, the ability to be opened multiple times, the ability to exploit the full capabilities of the hardware, and the lack of software layers to “simplify things” or provide policy-related operations.

## Splitting the Kernel
- In a Unix system, several concurrent processes attend to different tasks. Each process asks for system resources, be it computing power, memory, network connectivity, or some other resource.
- The kernel is the big chunk of executable code in charge of handling all such requests.
- the kernel’s role can be split into the following parts:
  - Process management The kernel is in charge of creating and destroying processes and handling their connection to the outside world (input and output).Communication among different processes (through signals, pipes, or interprocess communication primitives) is basic to the overall system functionality and is also handled by the kernel. In addition, the scheduler, which controls how processes share the CPU, is part of process management. More generally, the kernel’s process management activity implements the abstraction of several processes on top of a single CPU or a few of them.
  - Memory management The computer’s memory is a major resource, and the policy used to deal with it is a critical one for system performance. The kernel builds up a virtual addressing space for any and all processes on top of the limited available resources. The different parts of the kernel interact with the memory-management subsystem through a set of function calls, ranging from the simple malloc/free pair to much more complex functionalities.
  - Filesystems Unix is heavily based on the filesystem concept; almost everything in Unix can be treated as a file. The kernel builds a structured filesystem on top of unstructured hardware, and the resulting file abstraction is heavily used throughout the whole system.
  - Device control Almost every system operation eventually maps to a physical device. With the exception of the processor, memory, and a very few other entities, any and all device control operations are performed by code that is specific to the device being addressed.
  - Networking must be managed by the operating system, because most network operations are not specific to a process: incoming packets are asynchronous events. The packets must be collected, identified, and dispatched before a process takes care of them.

## Loadable Modules
- One of the good features of Linux is the ability to extend at runtime the set of fea-
tures offered by the kernel.
- This means that you can add functionality to the kernel (and remove functionality as well) while the system is up and running.
- Each piece of code that can be added to the kernel at runtime is called a module. The Linux kernel offers support for quite a few different types
- that can be dynamically linked to the run-
ning kernel by the insmod program and can be unlinked by the rmmod program.

## PICOUT

## Classes of Devices and Modules
- The Linux way of looking at devices distinguishes between three fundamental device types.
- Each module usually implements one of these types (char module, a block module, or a network module)
- the programmer can choose to build huge modules implementing different drivers in a single chunk of code. Good programmers, nonetheless, usually create a different module for each new functionality they implement, because decomposition is a key element of scalability and extendability.

### Character devices: 
- A character (char) device is one that can be accessed as a stream of bytes (like a file); a char driver is in charge of implementing this behavior. Such a driver usually implements at least the open, close, read, and write system calls.

### Block devices:
- Like char devices, block devices are accessed by filesystem nodes in the /dev directory.
- A block device is a device (e.g., a disk) that can host a filesystem. In most Unix systems, a block device can only handle I/O operations that transfer one or more whole blocks, which are usually 512 bytes
- Linux, instead, allows the application to read and write a block device like a char device—it permits the transfer of any number of bytes at
a time.
- As a result, block and char devices differ only in the way data is managed internally by the kernel, and thus in the kernel/driver software interface.
- Block drivers have a completely different interface to the kernel than char drivers.

### Network interfaces
- Any network transaction is made through an interface, that is, a device that is able to exchange data with other hosts.
- an interface is a hardware device, but it might also be a pure software device, like the loopback interface.
- A network interface is in charge of sending and receiving data packets, driven by the network subsystem of the kernel

## Security Issues
- Any security check in the system is enforced by kernel code. If the kernel has security holes, then the system as a whole has holes.
- the system call init_module checks if the invoking process is authorized to load a module into the kernel.

## Version Numbering
- every software package used in a Linux system has its own release number.

## License Terms
- Linux is licensed under Version 2 of the GNU General Public License (GPL), a document devised for the GNU project by the Free Software Foundation.
- The GPL allows anybody to redistribute, and even sell, a product covered by the GPL, as long as the recipient has access to the source and is able to exercise the same rights.