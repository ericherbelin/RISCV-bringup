# How to build the Rocket Chip repository:

Before you begin, make sure you have sudo permissions in order to install needed packages.

All information is from https://github.com/freechipsproject/rocket-chip and from https://github.com/ucb-bar/project-template. 

Mainly, there are two ways to install the rocket chip: the hard way, by doing all the steps by yourself, and the easy way, that gets you the basic installtion. 

## Let us begin with the hard way:

First, you need to check out the code:

        $ git clone https://github.com/ucb-bar/rocket-chip.git
        $ cd rocket-chip
        $ git submodule update --init

To build the rocket-chip repository, you must point the RISCV environment variable to your riscv-tools installation directory:

        $ export RISCV=/path/to/riscv/toolchain/installation

For example, in our system:
rotemshahar@asic2-serv3:~$ echo $RISCV
/home/rotemshahar/project-template/rocket-chip/riscv-tools

The riscv-tools repository is already included in rocket-chip as a Git submodule. You must build this version of riscv-tools: (on linux, install Newlib)

      $ cd rocket-chip/riscv-tools
      $ git submodule update --init --recursive 
      $ export RISCV=/path/to/install/riscv/toolchain 
      $ export MAKEFLAGS="$MAKEFLAGS -jN" # Assuming you have N cores on your host system 
      $ ./build.sh 
      $ ./build-rv32ima.sh (if you are using RV32).
  
### Ubuntu packages needed:

    $ sudo apt-get install autoconf automake autotools-dev curl device-tree-compiler libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev
    
### Install chisel3:

1.	Install Java
    
        $ sudo apt-get install default-jdk
        
2.	Install sbt, which isn't available by default in the system package manager
    https://www.scala-sbt.org/release/docs/Installing-sbt-on-Linux.html
    
        $ echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list 
        $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
        $ sudo apt-get update
        $ sudo apt-get install sbt

### Install Verilator: (the recommended version is 3.904)

1.	Install prerequisites (if not installed already): 

        $ sudo apt-get install git make autoconf g++ flex bison
        
2.	Clone the Verilator repository: 

        $ git clone http://git.veripool.org/git/verilator
        
3.	In the Verilator repository directory, check out a known good version:  

        $ git pull 
        $ git checkout verilator_3_904
        
4.	In the Verilator repository directory, build and install: 

        $ unset VERILATOR_ROOT # For bash, unsetenv for csh 
        $ autoconf # Create ./configure script 
        $ ./configure 
        $ make 
        $ sudo make install

### Build the project: 

(follow the instructions on this link)
https://github.com/freechipsproject/rocket-chip#building-the-project

Now, there is another way to install and use the rocket chip, in this way all you have to do is follow those instructions 
(from here https://github.com/ucb-bar/project-template):
First steps, as usual, check out the code:

        $ git clone https://github.com/ucb-bar/project-template.git
        $ cd project-template
        $ git submodule update --init --recursive 

Then export those variables:

        $ export RISCV=/path/to/install/dir
        $ export PATH=$RISCV/bin:$PATH
        
next step, build the riscv-tools:

        $ cd rocket-chip/riscv-tools
        $ ./build.sh
        
And that’s it. You can then use this to create your own project:

        $ make PROJECT=yourproject CONFIG=YourConfig
        $ ./simulator-yourproject-YourConfig ...
For example, run this test:

        $ ./simulator-example-DefaultExampleConfig $RISCV/riscv64-unknown-elf/share/riscv-tests/isa/rv64ui-p-simple
