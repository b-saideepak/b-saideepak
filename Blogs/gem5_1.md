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
Let us break the command and understand it part by part.

1. `build/X86/gem5.opt` is the gem5 binary that you have built.

2. `configs/example/se.py` is the configuration script for your system. This script contains the details of your system that you are using. The two main scripts that you use are se.py and fs.py which are short forms of system emulation and full system. In system emulation mode, there is no operating system involved, all the system calls in your program are emulated. But, it is very easy to use and almost all of the beginners use this configuration while they are in their initial days. Whereas, full system simulation comes with an operating system and gives you the real stats. Running a full system is little tricky and we will look into it later.

3. Moving on, `--cmd` this option is used to provide the binary that you want to execute using the simulator. 

4. --cpu-type` denotes the type of CPU that you want to use. There are different types of CPU that you can choose from `AtomicSimpleCPU` `DerivO3CPU` etc. We will use `TimingSimpleCPU` for simplicity although you can change based on your requirement.

5. To use caches in your system you just have to mention them as an option. `--caches` and `--l2cache` include l1cache and l2cache in your system. You could see the following output if you have built gem5 correctly.


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
Most of the times I wanted to make changes only to a particular cache say L1, L2. But the code for all the caches will be the same. So if you make changes to that, those will be applied to all the caches. It is better if you distinguish between caches using a level variable.

1. All the parameters that you want your C++ file to have can be declared in the python script(in this case Cache.py). Add a parameter named `Level` in the script.
```bash
Level = Param.Int(0,"Denotes cache level")
```
0 is the initial value of the variable.

2. You have to set the level variable for all the caches using `configs/common/Caches.py`. In this file, you can find all the caches that are being declared. In each cache that you want to assign a value. Add this option
```bash
Level = 1
```
You can add level 1 for instruction and data caches. In my case I only wanted to make changes to dcache, so I gave different values to dcache and icache.

3. You have to receive this value in the constructor of the `Cache`(`base.cc` or `cache.cc`) in a variable using params. Here I am doing it in `base.cc` and as `cache.cc` inherits `base.cc` you can directly use it in `cache.cc` too.
```bash
cache_level(p->Level)
```
Note that you have to declare the `cache_level` variable in the corresponding header file (`base.hh` or `cache.hh`).

## Adding a Pseudo Instruction
Many a times, you might want to add an instruction to the ISA. Sometimes, if the functionality of the instruction is minimal it can be done using a pseudo instruction in gem5. Using a pseudo instruction is way simple compared to adding an instruction to ISA. Gem5 already has some reserved opcodes which the user can use. I took the help of this [link](http://gedare-csphd.blogspot.com/2013/02/add-pseudo-instruction-to-gem5.html) But the structure of m5 files now is different.

1. Add the instruction in `src/arch/x86/isa/decoder/two_byte_opcodes.isa` 
```bash
0x56: mynewop({{
    Rax = PseudoInst::mynewop(xc->tcBase(), Rdi, Rsi);
    }}, IsNonSpeculative);
```
You have to do this using the m5reserved opcodes. In my case I have overridden m5reserved2.

2. Add the instruction and definition in `src/sim/pseudo_inst.cc`
In switch case:
```bash
case M5_MYNEWOP:
m5_mynewop(tc, args[0], args[1]);
break;
```
definition

```bash
uint64_t
mynewop(ThreadContext *tc, uint64_t arg1, uint64_t arg2)
{
    if (!FullSystem) {
        panicFsOnlyPseudoInst("mynewop");
        return 0;
    }

    return arg1+arg2;
}
```

3. Declare the function in `src/sim/pseudo_inst.hh` 
```bash
uint64_t mynewop(ThreadContext *tc, uint64_t arg1, uint64_t arg2);
```

4. Add the function declaration also in `include/gem5/m5ops.h`
```bash
uint64_t m5_mynewop(uint64_t arg1, uint64_t arg2);
```
5. Add the instruction in `util/m5/m5op_x86.S`
```bash
TWO_BYTE_OP(m5_mynewop, mynewop_func)
```

6. Add the function in `include/gem5/asm/generic/m5ops.h`
```bash
#define mynewop_func            0x56 // Reserved for user
```
and in `#define M5OP_FOREACH`
```bash
M5OP(m5_mynewop, M5_MYNEWOP)
```



























