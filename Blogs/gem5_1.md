# My first month with gem5
* * *
It had been a tough month I should say given the complexity of the simulator. I somehow managed to get started with gem5. I will summarize what all I learnt in this one month. If you are beginner, this is the right place for you.

## Installing gem5
Installing is not a tough task. Installation is already well [documented](https://www.gem5.org/documentation/learning_gem5/part1/building/)

## Executing the Hello World
I am assuming that you have successfully built gem5. Then go ahead and execute the following command.
```bash
$ ./build/X86/gem5.opt configs/example/se.py --cmd=tests/test-progs/hello/bin/x86/linux/hello --cpu-type=TimingSimpleCPU --caches --l2cache
```
Let us break the command and understand it part by part. `build/X86/gem5.opt` is the gem5 binary that you have built. `configs/example/se.py` is the configuration script for your system. This script contains the details of your system that you are using. The two main scripts that you use are se.py and fs.py which are short forms of system emulation and full system. In system emulation mode, there is no operating system involved, all the system calls in your program are emulated. But, it is very easy to use and almost all of the beginners use this configuration while they are in their initial days. Whereas, full system simulation comes with an operating system and gives you the real stats. Running a full system is little tricky and we will look into it later. Moving on, `--cmd` this option is used to provide the binary that you want to execute using the simulator. `--cpu-type` denotes the type of CPU that you want to use. There are different types of CPU that you can choose from `AtomicSimpleCPU` `DerivO3CPU` etc. We will use `TimingSimpleCPU` for simplicity although you can change based on your requirement. To use caches in your system you just have to mention them as an option. `--caches` and `--l2cache` include l1cache and l2cache in your system. You could see the following output if you have built gem5 correctly.
```bash
gem5 is copyrighted software; use the --copyright option for details.

gem5 compiled Jun 16 2020 12:59:08
gem5 started Jul 16 2020 09:20:26
gem5 executing on localhost, pid 19925
command line: ./build/X86/gem5.opt configs/example/se.py --cmd=tests/test-progs/hello/bin/x86/linux/hello --cpu-type=TimingSimpleCPU --caches --l2cache

Global frequency set at 1000000000000 ticks per second
warn: DRAM device capacity (8192 Mbytes) does not match the address range assigned (512 Mbytes)
0: system.remote_gdb: listening for remote gdb on port 7000
**** REAL SIMULATION ****
info: Entering event queue @ 0.  Starting simulation...
Hello world!
Exiting @ tick 40790500 because exiting with last active thread context
``` 
## Adding a debug flag
Most of the times you will perform changes to gem5 and you want to check whether it is working in the way you expect. At this time debug flags will help you. We use printf in our programming world sometimes to debug our code. We should use debug-flags in gem5. Suppose say you have made some changes to cache and want to check whether it is working in the way you expect. You have to register the debug-flags in your Sconsript. Include the following statements in your sconscript.
```bash
Debugflag('YourDebugFlagName')
```
This will automatically generate the header file `'debug/YourDebugFlagName.hh'` after you rebuild gem5. You have to include this header file in your code. A sample use is:
```bash
#include "debug/YourDebugFlagName.hh"
DPRINTF(YourDebugFlagName, "Debugging using my own debug flag\n");
```
## Adding a Level Option
