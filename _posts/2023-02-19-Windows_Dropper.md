---
title: Creating a Dropper in C
date: 2023-02-19 12:00:00 -500
categories: [blog]
tags: [code_execution]     #   TAG should always be lowercase

---

In this post we will take a very high level approach to creating a dropper for Windows. The structure of the dropper can take the form of 3 templates, each using a different structure of the portable executable to execute your shellcode. A dropper is just an application/binary/PE that delivers and executes your payload/Shellcode on the target system.

# What is a Portable Executable (PE)

a PE is a Way of structuring a file on disk so that the Windows loader recognizes the file as executable code and loads it into memory.

# What is the difference between an EXE and a DLL

Two of the main PEs offensive operators can/will make use of is .exe(s) and .dll(s). The main difference between the two is that an EXE is a self-contained structure and completely independant in memory. The EXE will create a new process in windows and can interact with other processes, load different libraries and use all the different Win32/NT APIs. a DLL on the other hand is just a library that is called by an already in-memory process and usually needs a function inside the library that is called by the process.

#

https://github.com/corkami/pics/blob/master/binary/pe101/pe101.png

![headers](/assets/img/Headers.jpg)

![sections](/assets/img/PEBear.png)
