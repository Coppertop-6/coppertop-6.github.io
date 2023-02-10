---
title: Executing Unmanaged Code in Memory
date: 2023-02-08 12:00:00 -500
categories: [blog]
tags: [code_execution]     #   TAG should always be lowercase

---

# Difference between P/Invoke & D/Invoke

With the ever increasing effectiveness of detection tools on the Blue Team side, more sophisticated ways of executing unmanaged code is needed on the Red Team side. Powershell has long been a go-to for Red Teamers, but has also been heavily scrutinized by detective capabilities and, as such, much of the Offensive tradecraft has moved over to the flexibility that .NET  provides. Two such mechanisms have gained popularity over the years and, in this post, I will attempt to explain the difference between the two mechanisms.

# P/Invoke - Platform Invoke

P/Invoke allows .NET applications to access structs, callbacks, and functions in unmanaged libraries (DLLs). One of the main reasons that .NET has become popular with Offensive tool developers is that .NET assemblies are easy to load and execute from memory which means that as a Red Team Operator with initial access, there is no need to touch disk while executing advanced post-exploitation activities.

Something to consider, i.t.o detection, when using P/Invoke:

When a .NET assembly is loaded, the Import Address Table will be updated to include the memory addresses of the API calls and functions that are called by the assembly (Static Reference). For example, if you make use of P/Invoke to call CreateRemoteThread, then your executable’s import address table will include a static reference to that function. This is easy to view from a defensive perspective when analyzing the executable and will clearly show that you are making a malicious call to a remote process. Any endpoint security product monitoring API calls, through API Hooking, on the endpoint will catch any call made through P/Invoke. More advanced Endpoint Detection and Response tools may also easily block these malicious looking calls.

# D/Invoke - Dynamic Invoke

D/Invoke contrasts the P/Invoke method by dynamically referencing the function addresses. The application has to manually look for the memory address of each function. This is accomplished by loading DLLs into memory manually through whatever mechanism the red team operator prefers. Once the DLL is loaded into memory, all that is left is to get a pointer to a function in the DLL and calling that function using the pointer.

The actual .NET feature that allows this is called delegates. 'Delegates are used to pass methods as arguments to other methods'. The Delegate API allows the wrapping of a method/function in a class. The distinctive feature of delegates is that they can be instantiated from a pointer to a function and dynamically invoking said function while passing in parameters.

As an example, from the SharpSploit toolkit:

```c#
public static object DynamicAPIInvoke(string DLLName, string FunctionName, Type FunctionDelegateType, ref object[] Parameters)
{
    IntPtr pFunction = GetLibraryAddress(DLLName, FunctionName);
    return DynamicFunctionInvoke(pFunction, FunctionDelegateType, ref Parameters);
}

public static object DynamicFunctionInvoke(IntPtr FunctionPointer, Type FunctionDelegateType, ref object[] Parameters)
{
    Delegate funcDelegate = Marshal.GetDelegateForFunctionPointer(FunctionPointer, FunctionDelegateType);
    return funcDelegate.DynamicInvoke(Parameters);
}
```

The second method is the core of the D/Invoke API. It creates a Delegate from a function pointer (funcDelegate) and invokes the function wrapped by the delegate (funcDelegate.DynamicInvoke), passing in your parameter (Parameters). The parameters are passed as an array of Objects (ref object[] Parameters) which provides the flexibility of passing data in any form needed. The only consideration is that the data needs to be structured in the way the unmanaged DLL expects it to be.

The most difficult part to understand is the Type FunctionDelegateType parameter. This is where you pass in the function prototype of the unmanaged code that you want to call. The prototype is the instructions for the delegate on how to structure the the CPU registers and the memory stack when the function is invoked. Defining a delegate works in a similar manner. You can define a delegate similar to how you would define a variable. Optionally, you can specify what calling convention to use when calling the function wrapped by the delegate.

Here is an example of a FunctionDelegateType for the NtCreateThreadEx API:

```c#
[UnmanagedFunctionPointer(CallingConvention.StdCall)]
public delegate Execute.Native.NTSTATUS NtCreateThreadEx(
    out IntPtr threadHandle,
    Execute.Win32.WinNT.ACCESS_MASK desiredAccess,
    IntPtr objectAttributes,
    IntPtr processHandle,
    IntPtr startAddress,
    IntPtr parameter,
    bool createSuspended,
    int stackZeroBits,
    int sizeOfStack,
    int maximumStackSize,
    IntPtr attributeList);
    
```

There is already a library of delegates and function wrappers for commonly used NT and Win32 APIs.

Using this dynamic loading technique removes the detection considerations present with P/Invoke as there are no suspicious API calls in the IAT of your Assembly.

# In Practise

SharpSploit now has a DInvoke library in the SharpSploit.Execution.DynamicInvoke namespace. The DInvoke library provides a managed wrapper function for each unmanaged function. The wrapper helps the user by ensuring that parameters are passed in correctly and the correct type of object is returned.

The code below demonstrates how DInvoke is used for the NtCreateThreadEx function in ntdll.dll. The delegate (that sets up the function prototype) is stored in the SharpSploit.Execution.DynamicInvoke.Native.DELEGATES struct. The wrapper method is SharpSploit.Execution.DynamicInvoke.Native.NtCreateThreadEx that takes all of the same parameters that you would expect to use in a normal PInvoke.

```c#
namespace SharpSploit.Execution.DynamicInvoke
{
    /// <summary>
    /// Contains function prototypes and wrapper functions for dynamically invoking NT API Calls.
    /// </summary>
    public class Native
    {
        public static Execute.Native.NTSTATUS NtCreateThreadEx(
            ref IntPtr threadHandle,
            Execute.Win32.WinNT.ACCESS_MASK desiredAccess,
            IntPtr objectAttributes, IntPtr processHandle,
            IntPtr startAddress,
            IntPtr parameter,
            bool createSuspended,
            int stackZeroBits,
            int sizeOfStack,
            int maximumStackSize,
            IntPtr attributeList)
        {
            // Craft an array for the arguments
            object[] funcargs =
            {
                threadHandle, desiredAccess, objectAttributes, processHandle, startAddress, parameter, createSuspended, stackZeroBits,
                sizeOfStack, maximumStackSize, attributeList
            };

            Execute.Native.NTSTATUS retValue = (Execute.Native.NTSTATUS)Generic.DynamicAPIInvoke(@"ntdll.dll", @"NtCreateThreadEx",
                typeof(DELEGATES.NtCreateThreadEx), ref funcargs);

            // Update the modified variables
            threadHandle = (IntPtr)funcargs[0];

            return retValue;
        }

        public struct DELEGATES
        {
            [UnmanagedFunctionPointer(CallingConvention.StdCall)]
            public delegate Execute.Native.NTSTATUS NtCreateThreadEx(
                out IntPtr threadHandle,
                Execute.Win32.WinNT.ACCESS_MASK desiredAccess,
                IntPtr objectAttributes,
                IntPtr processHandle,
                IntPtr startAddress,
                IntPtr parameter,
                bool createSuspended,
                int stackZeroBits,
                int sizeOfStack,
                int maximumStackSize,
                IntPtr attributeList);
        }
```
Hopefully this clarified the difference between these popular techniques and what to keep in mind when using them.
   
https://thewover.github.io/Dynamic-Invoke/
https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke
https://github.com/cobbr/SharpSploit

Post written by: *J33r4ff3*
