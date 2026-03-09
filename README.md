# Educational-Windows-DLL-Injector

Disclaimer: A lightweight python implementation of the classic LoadLibrary DLL Injection method FOR EDUCATIONAL USE ONLY! Intended for use
           in controlled lab environments with explicit permission from the owner or administrator of any target system.
           REMEMBER Unauthorised use of this tool may violate local, national and international law — the author accepts no 
           responsibility for misuse.

Overview: DLL Injection attempts to force a running program to load and execute external code that it wasn't orginally designed to run.

# [>] How it works

  1. Capturing the PID(process identification) of the target and the Explicit Path To DLL(C:\Path\to\your\dll\example.dll).
  2. Process the Dll path into a memory compliant c charcter array.
  3. Open the host process using kernel32's OpenProcess Windows API function.
  4. Allocate memory in the host process using VirtualAllocEx, passing the Windows memory constants and buffer size
     (equivalent to DLL length) as parameters then storing the returned address for use later.
  5. Write the DLL path into allocated memory using kernel32's WriteProcessMemory function with the VirtualAllocEx memory address
     created before.
  6. Get the address of LoadLibrary A.
  7. Create a remote thread and attempt to execute the DLL code.

# Requirements

[>] Python 3.x
[>] Windows OS
[>] Target PID which can be found using Procmon.exe,Procexp.exe and Task Manager under the details page
[>] A compiled DLL file which has to match the architecture of your host program
[>] Administrative privileges recommended as OpenProcesses may fail without elevated rights when using PROCESS_ALL_ACCESS

# Usage
[>] Navigate to the tool directory and run:
    python pythonloadlibrary.py
[>] The tool will then prompt you for:
    [*] The PID of the target process
    [*] The explicit path to your compiled DLL
[!] Try running the tool in an administrative terminal as this will yield better results. However it is not necessary and may yield an
    interesting result like: [!] Failed remote thread: 5

# Detection

While modern windows systems have very capable means of prevention such as CFG(control flow guard), DEP(data execution prevention),
SafeDLLSearchMode, WDAC(Windows Defender Application Control) among so many others. Detection is relatively simple and I will provide some
examples:

[1] In use method! Look for CreateRemoteThread API with LoadLibrary functions: These are cornerstones of the attack that is not commonly
    used in windows systems meaning there will be far less false positives than say searching for usage of VirtualAllocEx.

[2] Log activity method! Presence of Issued error 0xC0000409 in event viewer logs: Often DLL Injection fails like this! The error in
    question is (STATUS_STACK_BUFFER_OVERRUN) which indicates that most likely the security mechanism CFG(Control Flow Guard) terminated
    the process before any exploitation could take place and it is worth investigating the faulting process.

[3] Behavior Based method! Calling of OpenProcess, VirtualAllocEx, WriteProcessMemory and CreateRemoteThread: This method of detection is<br>
    easy to understand however it is underrated as interpreted languages like python are significantly harder for signature based 
    anti-virus softwares to detect, so using a behavior based model like this would be far more effective. In fact in my environment 
    Windows Defender did not prevent the execution of the code on both User and Admin cmd shells, instead the binary security 
    mechanisms thwarted the attack.

[4] Process Hunting method! Look for processes loading files that have just been created on disk: If for some reason a program has not 
    been compiled with CFG(Control Flow Guard) or Windows Security Mechanisms have failed to prevent this attack. We can search for
    compromised processes like this. Tools like Sysmon(Event ID 7) can monitor for processes loading recently created DLL files from
    suspicious paths.

# References

[!] The attack in question and theory around it was explained in great detail in this Hack The Box Module by Cry0l1t3.
    https://academy.hackthebox.com/app/module/67

[!] I'd also like to state that some of this was supplemented by independent research into the Windows API documentation and particularly
    its usage in python.


  






