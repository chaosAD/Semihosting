`printf` Through Semihosting Under SW4STM32
===========================================
Most embedded systems does not come with a display. To print/dump useful information, like log, for debugging purpose can be a bit tricky. However GCC, GDB, and OpenOCD provide `semihosting` to enable dumping of the information onto the the debugging console instead. The feature must be manually enabled. The following steps show how to do it (I used the idea from [here][1]).

First, we need to tell the `linker` to link with `libc` library (from `nanolib C`) that provides `printf` function and also to link with `librdimon` library that does the semihosting. This is done by configuring the the linker flag. To do that under SW4STM32 IDE, click on `Project Properties`, select `C/C++ Build`, then `Settings`. In the `MCU GCC Linker` menu, select `Miscellaneous`, then update the `Linker flags` field with:
```
-specs=nosys.specs -specs=nano.specs -specs=rdimon.specs -lc -lrdimon
```
![Advanced_Semihosting1](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/Advanced_Semihosting1.png)

In `Debug Configurations` add the following option in the `Startup` tab. To get there, click on `Run` and then `Debug Configurations...`:
```
monitor arm semihosting enable
```
The initialization script will be run everytime the debugger (GDB) is fired up. The command tells the OpenOCD to enable ARM semihosting. 

![GdbInitScript](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/GDBInitScript.png)

Add the following function prototype and function call. The `initialise_monitor_handles()` call must be before any printf call. The function initializes the hardware and software to handle semihosting:

![MonInitCode](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/MonitorInitializationCode.png)

Include `stdio.h` header file in `main.c`. The header is required to use `printf()`:

![IncludeStdio](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/IncludeStdio.png)

Add `printf()` call to print any message desired. Note that `\n` is important to force the message to be flushed to the console. Otherwise the message will not appear.

![PrintStatement](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/PrintStatement.png)

Run the program under debug mode. The `Hello, world!` should be printed in the OpenOCD console like the following:

![HelloWorldPrintedInConsole](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/HelloWorldPrintedInConsole.png)

It is fine with printing integer values. But it does not play well with floating point value:

![FailToPrintFloat](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/FailToPrintFloat.png)

To allow the printing of floating point values, in `Debug Configurations` **remove** the following option, as suggested [here][2]:
```
-specs=nano.specs
```
![RemoveNanoSpecs](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/RemoveNanoSpecs.png)

Now, it should be able to get printed:

![RemoveNanoSpecs](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/AbleToPrintFloat.png)

The only catch is that the **code size** and the **data size** are increased by about 20KB and 2KB, respectively.


`sscanf` Integer, Float, and Double
===================================
With the above change to the linker flag, we can also convert integer, float, and double values in a string and store them into some variables using `sscanf()`.

![RemoveNanoSpecs](https://github.com/chaosAD/Semihosting/blob/master/Docs/images/sscanfIntFloatDouble.png)

Note that for reading a `double`, `sccanf()` requires `%lf` instead of `%f` as the type field. However, printing a `double` by `printf()` uses `%f` as the type field, which is the same for printing a `float`. Also the **code size** is increased by an additional of about 20KB. 


Debugging
=========

Resetting Program
-----------------
[**Update (27/06/2017): New version of SW4STM32 IDE has a button on the toolbar to reset the MCU. However, the following serves as informational text.**]
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

The `tb` instruction is to create a temporary breakpoint at `main()`. Note that _temporary breakpoint_ means that once the breakpoint has been triggered, it is automatically discarded. To create a permanent break point, use `b` instead. The next line asks the monitor to reset and halt the MCU. The last line, requests GDB to continue (resume running).

To view the list of breakpoints, type `info breakpoints` and you get something like the below:
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
There are a number of useful GDB commands that you might want to try out:
```
info registers
list main
x                         # Step 1 instruction    
x/5i                      # Step 5 instructions
p/x $pc                   # Print PC (Program Counter) in hex   
```

References
==========
1. http://bgamari.github.io/posts/2014-10-31-semihosting.html
2. https://community.st.com/thread/41124-floating-point-support-in-sscanf
3. http://wiki.stm32duino.com/index.php?title=Blue_Pill
4. http://wiki.stm32duino.com/index.php?title=STM32_Smart_V2.0
5. https://mcuoneclipse.com/2014/09/11/semihosting-with-gnu-arm-embedded-launchpad-and-gnu-arm-eclipse-debug-plugins/

[1]: http://bgamari.github.io/posts/2014-10-31-semihosting.html
[2]: https://community.st.com/thread/41124-floating-point-support-in-sscanf
