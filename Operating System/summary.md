# 操作系统划书

# OS Overview

* What is **operating system**?

  > A modern computer consists of one or more processors, some main memory, disks, printers, a keyboard, a mouse, a display, network interfaces, and various other input/output devices. All in all, a complex `system.oo` If every application programmer had to understand how all these things work in detail, no code would ever get written. Furthermore, managing all these components and using them optimally is an exceedingly challenging job. <u>For this reason, computers are equipped with a layer of software called the **operating system**, whose job is to provide user programs with a better, simpler, cleaner, model of the computer and to handle managing all the resources just mentioned</u>. Operating systems are the subject of this book.

* Shell, GUI

  > The program that users interact with, usually called the **shell** when it is <u>text based</u> and the **GUI** (Graphical User Interface)—which is pronounced ‘‘gooey’’—when it uses icons, is actually <u>not</u> part of the operating system, although it uses the operating system to get its work done.

* Kernel / User Mode

  > The operating system, <u>the most fundamental piece of software, runs in **kernel mode**</u> (also called supervisor mode). In this mode it has complete access to all the hardware and can execute any instruction the machine is capable of executing. <u>The rest of the software runs in **user mode**</u>, in which only a subset of the machine instructions is available. 

# Process /  Thread

* Process

  > In this model, all the runnable software on the computer, sometimes including the operating system, is organized into a number of sequential processes, or just processes for short. <u>A **process** is just an instance of an executing program, including the current values of the program counter, registers, and variables.</u> Conceptually, each process has its own virtual CPU. In reality, of course, the real CPU switches back and forth from process to process, but to understand the system, it is much easier to think about a collection of processes running in (pseudo) parallel than to try to keep track of how the CPU switches from program to program. This rapid switching back and forth is called multiprogramming, as we saw in Chap.1.

* Daemon

  > Processes that stay in the <u>background</u> to handle some activity such as email, Web pages, news, printing, and so on are called **daemons**.

* Copy-on-Write

  > In both UNIX and Windows systems, after a process is created, the <u>parent and child have their own distinct address spaces</u>. If either process changes a word in its address space, the change is not visible to the other process. In UNIX, the child’s initial address space is a copy of the parent’s, but there are definitely two distinct address spaces involved; no writable memory is shared. Some UNIX implementations share the program text between the two since that cannot be modified. Alternatively, the child may share all of the parent’s memory, but in that case the memory is shared **copy-on-write**, which means that <u>whenever either of the two wants to modify part of the memory, that chunk of memory is explicitly copied first to make sure the modification occurs in a private memory area</u>. Again, no writable memory is shared. It is, however, possible for a newly created process to share some of its creator’s other resources, such as open files. In Windows, the parent’s and child’s address spaces are different from the start.

* Threads

  > mini-processes(笔记里有老师的定义)