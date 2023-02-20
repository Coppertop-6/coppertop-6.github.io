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

The 3 sections that we can make use of for our payload/shellcode are the .data (Data), .text (Text) and .rsrc (Resources) sections. Each section that you choose for your payload will require a different layout in our C program. We will discuss the differences, but let's create the template for our first dropper program and then look at the distinguising features of our program and it's corresponding section. For any shellcode execution, there needs to be 4 essential functions within your program (There are different ways of doing this within the offensive community, but that is a post/posts for another day). 

Firstly, we want to reserve space in memory for our code and that can be accomplished by using the ```VirtualAlloc``` API. We allocate the memory space and also tell Windows what protections to apply (Read and Write) to the memory space ```flProtect``` (very important for detection considerations).

```c++
VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect)
```
Secondly, we want to copy our payload (assigned to a variable in our template) into the newly reserved memory space with the ```RtlMoveMemory``` API.

```c++
RtlMoveMemory(VOID *Destination, VOID *Source, SIZE_T Length);
```

Thirdly, we change the protection on the assigned memory region to make it executable (Read, Write, Execute) by using the ```VirtualProtect``` API. Changing the memory protection after assignment is slightly less suspicious for AV/EDR tools.

```c++
VirtualProtect(LPVOID lpAddress, SIZE_T dwSize, DWORD flNewProtect, PDWORD lpflOldProtect);
```

Lastly we make use of the ```CreateThread``` API to execute our code in a new thread within the process.

Our template for our dropper should now look like this:

```c++
unsigned char payload[] = {SHELLCODE};
unsigned int payload_length = SHELLCODE_LENGTH;
```

```c++
int main(void) {

// Reserve space in memory for our payload
me_reserve = VirtualAlloc(0, payload_length, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

// Move the payload to the newly reserved space.
RtlMoveMemory(me_reserve, payload, payload_length);
	
// Change the memory protection to add execute permissions
mem_protect = VirtualProtect(me_reserve, payload_length, PAGE_EXECUTE_READ, &oldprotect);

// run the payload
if ( mem_protect != 0 ) {
	exec = CreateThread(0, 0, (LPTHREAD_START_ROUTINE) me_reserve, 0, 0, 0);
	WaitForSingleObject(th, -1);
}

return 0;
}

```

The differences in the code comes in w.r.t to which section you would like to use for your payload.

If you want to use the ```.text``` section, you need to assign your payload as a local variable. So within the main function of the application.
If you want to use the ```.data``` section, you need to assign your payload as a global variable. So outside of the main function of the application.

The use of the ```.rsrc``` section is slightly more complex:

<ol>
  <li>Store your payload in a resources file on disk - payload.rc</li>
  <li>You need an additional include in the application "include resources.h"</li>
  <li>The payload needs to be extracted from a file using the FindResource, LoadResource and LockResource APIs before continuing with the above template as normal</li>
  <li>You will need to use the Windows Resource Compiler to create an object file from your payload resources file</li>
</ol>

Hopefully this post has clarified a couple of nuances for you and we will continue with this thread in future posts to look at some techniques to avoid getting detected by AV/EDR tools.
