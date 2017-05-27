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
