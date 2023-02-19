---
title: Creating a Dropper in C
date: 2023-02-19 12:00:00 -500
categories: [blog]
tags: [code_execution]     #   TAG should always be lowercase

---

In this post we will take a very high level approach to creating a dropper for Windows. The structure of the dropper can take the form of 3 templates, each using a different structure of the portable executable to execute your shellcode. A dropper is just an application/binary/PE that delivers and executes your payload/Shellcode on the target system.

# What is a Portable Executable (PE)

a PE is a Way of structuring a file on disk so that the Windows loader recognizes the file as executable code and loads it into memory. If you want to have a detailed look at the structure of a PE, [here](https://github.com/corkami/pics/blob/master/binary/pe101/pe101.png) you go. The two main structures we will be looking at will be Headers and Sections.

Headers is just metadata about the PE. The below image shows the information contained in the headers for notepad.exe.

![headers](/assets/img/Headers.jpg)

Sections actually contain the executable code for a PE. The below image shows the sections contained in notepad.exe. These sections will become important later on when choosing where to store your payload.

![sections](/assets/img/PEBear.png)

# What is the difference between an EXE and a DLL

Two of the main PEs offensive operators can/will make use of is .exe(s) and .dll(s). The main difference between the two is that an EXE is a self-contained structure and completely independant in memory. The EXE will create a new process in windows and can interact with other processes, load different libraries and use all the different Win32/NT APIs. a DLL on the other hand is just a library that is called by an already in-memory process and usually needs a function inside the library that is called by the process.

# Sections for use in offensive tradecraft

The 3 sections that we can make use of for our payload/shellcode are the .data (Data), .text (Text) and .rsrc (Resources) sections. Each section that you choose for your payload will require a different layout in our C program. We will discuss the differences, but let's create the template for our first dropper program and then look at the distinguising features of our program and it's corresponding section. For any windows process to start and execute code, there needs to be 4 essential functions within your program (There are different ways of doing this within the offensive community, but that is a post/posts for another day). 

VirtualAlloc, RtlMovememory, VirtualProtect - execute thread



