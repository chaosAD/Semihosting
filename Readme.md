`printf` Through Semihosting Under SW4STM32
===========================================
Most embedded systems does not come with a display. To print/dump useful information, like log for debugging purpose, can be a bit tricky. However GCC, GDB, and OpenOCD provide `semihosting` to enable dumping the information onto the the debugging cosole instead [1]. The feature must be manually enabled. The following steps show how to do it.

First, we need to tell the `linker` to link with `libc` library (from `nanolib C`) that provides `printf` function and also to link with `librdimon` library that does the semihosting. This is done by configuring the the linker flag. To do that under SW4STM32 IDE, click on `Project Properties`, select `C/C++ Build`, then `Settings`. In the `MCU GCC Linker` menu, select `Miscellaneous`, then update the `Linker flags` field with:
```
-specs=nosys.specs -specs=nano.specs -specs=rdimon.specs -lc -lrdimon
```
![GDB console](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/Advanced_Semihosting1.png)

In Debug Configurations add the following option in the Startup Tab. To get there, click on `Run` and then `Debug Configurations...`:
```
monitor arm semihosting enable
```
![GdbInitScript](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/GDBInitScript.png)

Add the following function prototype and function call. The `initialise_monitor_handles()` call must be before any printf call:

![MonInitCode](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/MonitorInitializationCode.png)

Include `stdio.h` header file in `main.c`:

![IncludeStdio](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/IncludeStdio.png)

Add `printf()` call to print any message desired. Note that `\n` is important to force the message to be flushed to the console. Otherwise the message will not appear.

![PrintStatement](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/PrintStatement.png)

Run the program under debug mode. The `Hello, world!` should be printed in the OpenOCD console like the following:

![HelloWorldPrintedInConsole](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/HelloWorldPrintedInConsole.png)



Debugging
=========

Resetting Program
-----------------
To reset the program under debugging mode, click on the GDB console (in `Debug` perspective):

![GDB console](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/OpenGdbConsole.png)

and type:
```
tb main
monitor reset halt
c
```
The following shows how it looks like:
![Resetting the program](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/ResettingProgram.png)

The `tb` instruction is to create a temporary breakpoint at main. Note that temporary breakpoint means once breakpoint is triggered, it is automatically discarded. To create a permanent break point, use `b` instead. The next line asks the monitor to reset and halt the MCU. The last line, request GDB to continue (run).

To view the list of brakpoints, type `info breakpoints` and you get something like the below:
```
Num     Type           Disp Enb Address    What
13      breakpoint     del  y   <PENDING>  main; monitor reset halt; c
15      breakpoint     keep y   0x08000e2a in main at ../Src/main.c:102
16      breakpoint     keep y   0x08000e1e in main at ../Src/main.c:76
17      breakpoint     keep y   0x08000d9c in SystemClock_Config at ../Src/main.c:121
```

The numbers on the leftmost are the IDs of the breakpoints. To delete a breakpoint, say breakpoint 15, type `del 15`.

Other Useful GDB Commands
-------------------------
The are a number of useful GDB commands that you might want to try out:
```
info registers
list main
x                         # Step 1 instruction    
x/5i                      # Step 5 instructions
p/x $pc                   # Print PC (Program Counter) in hex   
```

References
==========
[1] http://bgamari.github.io/posts/2014-10-31-semihosting.html
