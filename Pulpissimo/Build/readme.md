# How to build the Pulpissimo repository

There are three repositories to build (in this order):

1.	Pulp-SDK (=software development kit)
2.	Pulp-Riscv-gnu-toolchain
3.	Pulpissimo

Start follow the instructions (pulp-SDK):
from https://github.com/pulp-platform/pulp-sdk/blob/master/README.md 
 
    $ sudo apt install git python3-pip gawk texinfo libgmp-dev libmpfr-dev libmpc-dev swig3.0 libjpeg-dev lsb-core doxygen python-sphinx sox graphicsmagick-libmagick-dev-compat libsdl2-dev libswitch-perl libftdi1-dev cmake 
    $sudo pip3 install artifactory twisted prettytable sqlalchemy pyelftools openpyxl xlsxwriter pyyaml numpy  

from https://github.com/pulp-platform/pulp-riscv-gnu-toolchain 
 
    $ git clone --recursive https://github.com/pulp-platform/pulp-riscv-gnu-toolchain 
 
    $ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev 
 
    $ export PATH=$PATH:/opt/riscv/bin 
    $ cd pulp-riscv-gnu-toolchain/ 
    $ ./configure --prefix=/opt/riscv --with-arch=rv32imc --with-cmodel=medlow --enable-multilib   //This command chooses the ISA type you want to work with
    $ sudo chmod 775 * 

In the lab, we need admin approval for few commands, so we use "sudo"

    $ sudo make 
 
 
Back to https://github.com/pulp-platform/pulp-sdk/blob/master/README.md#dependencies-setup 
 
    $ export PULP_RISCV_GCC_TOOLCHAIN=/opt/riscv/ 
    $ git clone --recursive https://github.com/pulp-platform/pulpissimo.git 
    $ export VSIM_PATH=/home/rotemshahar/pulpissimo/sim 
    $ git clone --recursive https://github.com/pulp-platform/pulp-sdk.git -b master  
    $ cd pulp-sdk 
    $ source configs/pulpissimo.sh  // choose the core type
    $ source configs/platform-rtl.sh // options platform-board.sh  platform-fpga.sh   platform-gvsoc.sh  platform-rtl.sh
    $ make all 
    $ source pkg/sdk/dev/sourceme.sh // run every time-build things for our choice  
 
moving to https://github.com/pulp-platform/pulpissimo

    $ cd ../pulpissimo 
    $ export VSIM_PATH=/home/<your_home_dir>/pulpissimo/sim 

**in our repository: export VSIM_PATH=/home/rotemshahar/pulpissimo/sim

    $ ./update-ips 
    $ source setup/vsim.sh 
    $ cd sim/ 
    $ export LM_LICENSE_FILE=5280@132.68.55.55  //modelsim license
    $ export PATH=$PATH:"/home/<your_home_dir>/<modelsim_dir>/ bin/"
 
**in our repository:
export PATH=$PATH:"/home/rotemshahar/new_modelsim/modeltech/bin/"

    $ make clean lib build opt 
    $ git clone https://github.com/pulp-platform/pulp-rt-examples.git 
 
# Quick Start from our repository:

We worked with SSH: 132.68.59.10

    export PULP_RISCV_GCC_TOOLCHAIN=/opt/riscv/ 
    export VSIM_PATH=/home/rotemshahar/pulpissimo/sim 
    cd pulp-sdk 
    source configs/pulpissimo.sh 
    source configs/platform-rtl.sh 
    source pkg/sdk/dev/sourceme.sh 
    cd ../pulpissimo 
    source setup/vsim.sh 
    cd sim/ 
    export LM_LICENSE_FILE=5280@132.68.55.55 
    export PATH=$PATH:"/home/rotemshahar/new_modelsim/modeltech/bin/" 
    make clean lib build opt 
    cd *name of test folder* 
    make conf gui=1   // for modelsim gui
    make clean all run
 
