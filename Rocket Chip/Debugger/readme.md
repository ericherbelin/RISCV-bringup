# How to work with the debugger

All the information is from:

https://github.com/freechipsproject/rocket-chip#-debugging-with-gdb

## Step 1: Generating the Remote Bit-Bang (RBB) Emulator

The objective of this section is to use GNU debugger to debug RISC-V programs running on the emulator in the same fashion as in Spike (https://github.com/riscv/riscv-isa-sim#debugging-with-gdb)
For that we need to add a Remote Bit-Bang client to the emulator. 

We can do so by extending our Config with JtagDTMSystem, which will add a DebugTransportModuleJTAG to the DUT and connect a SimJTAG module in the Test Harness. This will allow OpenOCD to interface with the emulator, and GDB can interface with OpenOCD. 
The config file is locates at src/main/scala/system/Configs.scala. 
In the following example, we added this Config extension to the DefaultConfig: (add it at the end of the file)

        $ class DefaultConfigRBB extends Config(new WithJtagDTMSystem ++ new WithNBigCores(1) ++ new BaseConfig)  
        $ class QuadCoreConfigRBB extends Config( 
        $ new WithJtagDTMSystem ++ new WithNBigCores(4) ++ new BaseConfig)
    
To build the emulator with DefaultConfigRBB configuration we use the command:

        $ rocket-chip$ cd emulator
        $ emulator$ CONFIG=DefaultConfigRBB make 
        
 We can also build a debug version capable of generating VCD waveforms using the command:
 
        $ emulator$ CONFIG=DefaultConfigRBB make debug 
        
By default the emulator is generated under the name emulator-
freechips.rocketchip.system-DefaultConfigRBB in the first case and 
emulator-freechips.rocketchip.system-DefaultConfigRBB-debug in the second.
 
## Step 2: Compiling and executing a custom program using the emulator:

We suppose that helloworld is our program, you can use crt.S, syscalls.c and the linker script test.ld to construct your own program, check examples stated in riscv-tests (https://github.com/riscv/riscv-tests).
In our case we will use the following example:

        $ export PATH=$RISCV/bin:$PATH

You can install the git riscv-tests (with the link that is in this section). If, while installing the riscv-tests, you get an error: "riscv64-unknown-elf-gcc command not found", just type: "export PATH=$RISCV/bin:$PATH" and then continue.

In the riscv-tests there are some files you can use to test the debugger, they are at riscv-tests/benchmarks and the c file is in the folder with the same name. 
I use rsort which is a quicksort example. The .riscv is the assembly file, so where they use hellowold, I used rsort.riscv.

First we can test if your program executes well in the simple version of emulator before moving to debugging in step 3:

        $ ./emulator-freechips.rocketchip.system-DefaultConfig helloworld
        
Additional verbose information (clock cycle, pc, instruction being executed) can be printed using the following command:

        $ ./emulator-freechips.rocketchip.system-DefaultConfig +verbose helloworld 2>&1 | spike-dasm

VCD output files can be obtained using the -debug version of the emulator and are specified using -v or --vcd=FILEarguments. A detailed log file of all executed instructions can also be obtained from the emulator, this is an example:

        $ ./emulator-freechips.rocketchip.system-DefaultConfig-debug +verbose -v output.vcd  helloworld 2>&1 | spike-dasm > output.log
        
Please note that generated VCD waveforms and execution log files can be very voluminous depending on the size of the .elf file (i.e. code size + debugging symbols).

Please note also that the time it takes the emulator to load your program depends on executable size. Stripping the .elf executable will unsurprisingly make it run faster. For this you can use $RISCV/bin/riscv64-unknown-elf-strip tool to reduce the size. 
This is good for accelerating your simulation but not for debugging. Keep in mind that the HTIF communication interface between our system and the emulator relies on tohost and fromhost symbols to communicate. This is why you may get the following error when you try to run a totally stripped executable on the emulator:

        $ ./emulator-freechips.rocketchip.system-DefaultConfig totally-stripped-helloworld  
        
This text will appear on screen:

    This emulator compiled with JTAG Remote Bitbang client. To enable, use +jtag_rbb_enable=1.
    Listening on port 46529 
    warning: tohost and fromhost symbols not in ELF; can't communicate with target
    
To resolve this, we need to strip all the .elf executable but keep tohost and fromhost symbols using the following command:

        $ riscv64-unknown-elf-strip -s -Kfromhost -Ktohost helloworld 

More details on the GNU strip tool can be found here: https://www.thegeekstuff.com/2012/09/strip-command-examples/
The interest of this step is to make sure your program executes well. To perform debugging you need the original unstripped version, as explained in step 3.

## Step 3: Launch the emulator

First, do not forget to compile your program with -g -Og flags to provide debugging support as explained here.
We can then launch the Remote Bit-Bang enabled emulator with:

        $ ./emulator-freechips.rocketchip.system-DefaultConfigRBB +jtag_rbb_enable=1 --rbb-port=9823 helloworld 
        
This text will appear on screen:
    This emulator compiled with JTAG Remote Bitbang client. To enable, use +jtag_rbb_enable=1. 
    Listening on port 9823 
    Attempting to accept client socket  
You can also use the emulator-freechips.rocketchip.system-DefaultConfigRBB-debug version instead if you would like to generate VCD waveforms.

Please note that if the argument --rbb-port is not passed, a default free TCP port on your computer will be chosen randomly.
Please note also that when debugging with GDB, the .elf file is not actually loaded by the FESVR. 
In contrast with Spike, it must be loaded from GDB as explained in step 5. So the helloworld argument may be replaced by any dummy name.

## Step 4: Launch OpenOCD

You will need a RISC-V Enabled OpenOCD binary. This is installed with riscv-tools in $(RISCV)/bin/openocd, or can be compiled manually from riscv-openocd. OpenOCD requires a configuration file, in which we define the RBB port we will use, which is in our case 9823.
        
        $ cat cemulator.cfg

Go to $RISCV directory and type (to create a new file)

        $ nedit cemulator.cfg
        
Then copy and paste the text down here and save 

        interface remote_bitbang 
        remote_bitbang_host localhost 
        remote_bitbang_port 9823 
        set _CHIPNAME riscv 
        jtag newtap $_CHIPNAME cpu -irlen 5 
        set _TARGETNAME $_CHIPNAME.cpu 
        target create $_TARGETNAME riscv -chain-position $_TARGETNAME 
        gdb_report_data_abort enable 
        init 
        halt 
You may have to install OpenOCD first, go to the $RISCV/riscv-openocd directory (rocket-chip/riscv-tools/riscv-openocd) and type in this order: 

        ./onfigure 
        make 
        make install 
        
You may need sudo before the make install (sudo make install) 
Then we launch OpenOCD in another terminal using the command 

        $(RISCV)/bin/openocd -f ./cemulator.cfg 
        
This text will appear on screen (some of it will appear during the work with the GDB at the next step):

        Open On-Chip Debugger 0.10.0+dev-00112-g3c1c6e0 (2018-04-12-10:40) 
        Licensed under GNU GPL v2 
        For bug reports, read 
        http://openocd.org/doc/doxygen/bugs.html 
        Warn : Adapter driver 'remote_bitbang' did not declare which transports it allows; assuming legacy JTAG-only 
        Info : only one transport option; autoselect 'jtag' 
        Info : Initializing remote_bitbang driver 
        Info : Connecting to localhost:9823 
        Info : remote_bitbang driver initialized 
        Info : This adapter doesn't support configurable speed 
        Info : JTAG tap: riscv.cpu tap/device found: 0x00000001 (mfg: 0x000 (<invalid>), part: 0x0000, ver: 0x0) 
        Info : datacount=2 progbufsize=16 
        Info : Disabling abstract command reads from CSRs. 
        Info : Disabling abstract command writes to CSRs. 
        Info : [0] Found 1 triggers 
        Info : Examined RISC-V core; found 1 harts 
        Info :  hart 0: XLEN=64, 1 triggers 
        Info : Listening on port 3333 for gdb connections 
        Info : Listening on port 6666 for tcl connections 
        Info : Listening on port 4444 for telnet connections 
         A -d flag can be added to the command to show further debug information.
         
## Step 5: Launch GDB

In another terminal launch GDB and point to the elf file you would like to load then run it with the debugger (in this example, helloworld): 

        $ ./riscv64-unknown-elf-gdb helloworld 

Is it fails run this: export PATH=$RISCV/bin:$PATH and try again:
This text will appear on screen:

        GNU gdb (GDB) 8.0.50.20170724-git 
        Copyright (C) 2017 Free Software Foundation, Inc. 
        License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
        This is free software: you are free to change and redistribute it. 
        There is NO WARRANTY, to the extent permitted by law.  Type "show copying" and "show warranty" for details. 
        This GDB was configured as "--host=x86_64-pc-linux-gnu --target=riscv64-unknown-elf". 
        Type "show configuration" for configuration details. 
        For bug reporting instructions, please see: 
        http://www.gnu.org/software/gdb/bugs/.
        Find the GDB manual and other documentation resources online at: 
        http://www.gnu.org/software/gdb/documentation/.
        For help, type "help".
        Type "apropos word" to search for commands related to "word"... 
        Reading symbols from ./proj1.out...done. 
        (gdb) 
        
 Compared to Spike, the C Emulator is very slow, so several problems may be encountered due to timeouts between issuing commands and response from the emulator. 
 To solve this problem, we increase the timeout with the GDB set remotetimeout command. 
Then we load our program by performing a load command. This automatically sets the $PC to the _start symbol in our .elf file. 
        
        (gdb) set remotetimeout 2000 
        (gdb) target remote localhost:3333 
        Remote debugging using localhost:3333 
        0x0000000000010050 in ?? () 
        (gdb) load 
        Loading section .text.init, size 0x2cc lma 0x80000000 
        Loading section .tohost, size 0x48 lma 0x80001000 
        Loading section .text, size 0x98c lma 0x80001048 
        Loading section .rodata, size 0x158 lma 0x800019d4 
        Loading section .rodata.str1.8, size 0x20 lma 0x80001b30 
        Loading section .data, size 0x22 lma 0x80001b50 
        Loading section .sdata, size 0x4 lma 0x80001b74 
        Start address 0x80000000, load size 3646 
        Transfer rate: 40 bytes/sec, 520 bytes/write. 
        (gdb)
        
Now we can proceed as with Spike, debugging works in a similar way:

        (gdb) print wait 
        $1 = 1 
        (gdb) print wait=0 
        $2 = 0 
        (gdb) print text 
        $3 = "Vafgehpgvba frgf jnag gb or serr!" 
        (gdb) c 
        Continuing. 
        
        ^C 
        Program received signal SIGINT, Interrupt. 
        main (argc=0, argv=<optimized out>) at src/main.c:33 
        33	    while (!wait) 
        (gdb) print wait 
        $4 = 0 
        (gdb) print text 
        $5 = "Instruction sets want to be free!" 
        (gdb)
        
Further information about GDB debugging is available:
https://sourceware.org/gdb/onlinedocs/gdb/
https://sourceware.org/gdb/onlinedocs/gdb/Remote-Debugging.html#Remote-Debugging
