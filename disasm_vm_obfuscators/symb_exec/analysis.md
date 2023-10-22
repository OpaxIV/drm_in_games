# Symbolic Execution and Analysis of the vm_base.bin file
_Summarized by Fabio Schmidt<br/>
Original Article and Video Lecture by Dr. Tim Blazytko_<br/>

## General
The following document contains further analytical aspects of the vm_base.bin binary file. The main focus is to symbolically execute a part of the binary to get an understanding of how this technique works. 

### MIASM
Miasm is a free and open source (GPLv2) reverse engineering framework. The program aims to analyze / modify / generate binary programs.
In the following Miasm is going to be used as an intermediate language to symbolically exemplary execute some parts of the binary for academical purposes.




---

References:
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- Symbolic execution - https://en.wikipedia.org/wiki/Symbolic_execution
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- What is symbolic execution for software programs? - https://symflower.com/en/company/blog/2021/symbolic-execution/
- Symbolic Execution for finding bugs by Michael Hicks - https://www.cs.umd.edu/~mwh/se-tutorial/symbolic-exec.pdf
- MIASM - https://github.com/cea-sec/miasm
