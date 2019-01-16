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
