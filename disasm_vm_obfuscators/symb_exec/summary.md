# Symbolic Execution
_Summarized by Fabio Schmidt<br/>
Original Article and Video Lecture by Dr. Tim Blazytko_<br/>

Ref: https://synthesis.to/2021/10/21/vm_based_obfuscation.html

Ref: https://www.youtube.com/watch?v=b6udPT79itk

## Symbolic Execution
- Symbolic execution is a program analysis technique which allows us to symbolically evaluate and summarize assembly code.
- To determine what inputs cause each part of a program to execute. An interpreter follows the program, assuming symbolic values for inputs rather than obtaining actual inputs as normal execution of the program would.
- It thus arrives at expressions in terms of those symbols for expressions and variables in the program, and constraints in terms of those symbols for the possible outcomes of each conditional branch.
- Finally, the possible inputs that trigger a branch can be determined by solving the constraints.
-  



---

References:
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- Symbolic execution - https://en.wikipedia.org/wiki/Symbolic_execution
- What Is an Instruction Set Architecture? - https://www.arm.com/glossary/isa
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- 
   

