# How to add a new instruction (partial)

this is an example on how to change an existing command, and which files to look up to. For example, we want to change the instruction add\sub- increase the value by 1.
 
  1.	The RTL located in "rocket-chip/src/main/scala/rocket/" at .scala files.
  
  2.	Instructions.Scala- the file which contain the instruction object.
  
  3.	IDecode.Scala- contain the translation of the assembler command into parameters.
  
  4.	Constd.Scala- contain the opcode constants.
  
  5.	ALU.Scala- the implementation of the alu.
  
  6.	RVC.Scala- implementation of the immidiate instructions- 
  
including Addi lines- 74-79

    def q1 = { 
        def addi = inst(Cat(addiImm, rd, UInt(0,3), rd, UInt(0x13,7)), rd, rd, rs2p) 
         …. 
        } 
        
Inst- translate the instruction to a struct:  

    def inst(bits: UInt, rd: UInt = x(11,7), rs1: UInt = x(19,15), rs2: UInt = x(24,20), rs3: UInt = x(31,27)) = { 
        val res = Wire(new ExpandedInstruction) 
        res.bits := bits 
        res.rd := rd 
        res.rs1 := rs1 
        res.rs2 := rs2 
        res.rs3 := rs3 
        Res 
    } 
    
7.	RocketCore.Scala- execute stage (line 272) ALU is implemented by:

inputs: From Intctrlsigs 	
  ex_op1 + ex_op2 - mux lookup in core_scala
  Ex_inn- value of immidiate data 
  Ex_ctl.Dw- double word. 
  Ex_ctl.fn 	 
Outputs: 
  results 	 
  
8.	split to lsb + msb:

Chain of muxs that find key in inputs. The output is the operand- "val_ex_op1"
