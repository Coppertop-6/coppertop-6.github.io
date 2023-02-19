---
title: Creating a Dropper in C
date: 2023-02-19 12:00:00 -500
categories: [blog]
tags: [code_execution]     #   TAG should always be lowercase

---

In this post we will take a very high level approach to creating a dropper for Windows. The structure of the dropper can take the form of 3 templates, each using a different structure of the portable executable to execute your shellcode.

# What is a Portable Executable (PE)

a PE is a Way of structuring a file on disk so that the Windows loader recognizes the file as executable code and loads it into memory.

# What is the difference between an EXE and a DLL

Two of the main PEs offensive operators can/will make use of is .exe(s) and .dll(s).

#
