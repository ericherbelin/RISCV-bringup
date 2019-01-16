# Pulpissimo simulation using Modelsim

Using the Modelsim GUI very recommended, it allows watching the different objects, waveforms, signals (list report), etc.

We used the example tests, and re-wrote the code as our own tests. The code written in C. 

In order to activate the GUI, follow the instructions:

    cd <name of test folder>
    make conf gui=1  
    make clean all run

Results located in "build" repository.

## Important log/results files:

1.	"<test_repository path>/build/pulpissimo/trace_core_1f_0.log" – it contains the Assembly instructions, registers status and PC number.
2.	The List output from the Modelsim simulation. It contain timestamp and buses values at the time. 

The relevant buses which indicates the core's activity from the outside  are:

    •	instr_addr_o
    •	instr_rdata_I 
    •	data_addr_o 
    •	data_wdata_o 
    •	data_rdata_I

It is possible to convert the output to excel chart and analyze it.

Output of "Hello" test, without Modelsim's gui:

![ch3](https://user-images.githubusercontent.com/31187462/51264029-2ebb0580-19be-11e9-9616-9b9c6c21e1b6.png)


# Conclusions from Modelsim simulations

We tried to understand the memory behavior.  We discovered it uses an algorithm in order to save memory usage. 
Therefore, the program we executed used the same memory cell for all variables. 
Moreover, the simulation uses JTAG to read new instruction (simulates the instruction cache)

        #include <stdio.h>
        #include "pulp.h"
        int main()
        {
        int a=0xffff;
        int b=0xeeee;
        int c=0xdddd;
        int d=0xcccc;
        int e=0xbbbb;
        int f=0xaaaa0000;
        int sum_a,sum_b,sum_c,sum_d,sum_e,sum_f,sum_g=0;
        sum_a=a;
        sum_b=b;
        sum_c=c;
        sum_d=d;
        sum_e=e;
        sum_f=f+d;
        sum_g=f+a;
          printf("Sum=%d\n",sum_a);
          printf("Sum=%d\n",sum_b);
          printf("Sum=%d\n",sum_c);
          printf("Sum=%d\n",sum_d);
          printf("Sum=%d\n",sum_e);
          printf("Sum=%d\n",sum_f);
          printf("Sum=%d\n",sum_g);
          printf("Hello !\n");
          return 0;
        }
                ©Varun Tandon

The output list, converted to excel chart:

![ch4](https://user-images.githubusercontent.com/31187462/51264279-d2a4b100-19be-11e9-9e69-92f644349963.png)

We can see the values stored in the same place 


