# Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop by Dr. Tim Blazytko (referenced bellow).

## Definition of an VM Obfuscation
The exact definition of an VM-based obfuscation has already been stated under the section https://github.com/OpaxIV/hslu_secproj/blob/0bc6c015a4a7fa69abbd0b94e3960d5773a84f95/disasm_vm_obfuscators/summary.md
In a couple of words one can say, that this obfuscation technique uses a costum instruction set architecture as the basis of its obfuscation. Is hides the original code in a sequence of bytes, which are then interpreted at runtime.

### What to look out for
- VM entry
- VM exit
- Dispatcher
- Locate the bytecode
- Handlers (at least some)
- Functionality of rdx and rcx
  - Register used as Virtual Stack Pointer (OR)
  - Register used as Virtual Instrucion Pointer (OR)
- ...
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/55528869-41ab-4306-8412-19926d8b745e" width="600">


### Analysis Procedure
During the analysis, the goal is to identify individual components of the VM-based obfuscator. They include: 
1. Graph View
2. Identify the individual parts of the VM
3. Identify the architecture (stack- or instruction-based, hybrid)

---
### Ghidra
The binary has been analysed with ghidra. When selecting the language, I chose "x86, compiler: gcc".
#### Before your start
Make sure to download the file directly from github or with the `curl` terminal command.
The file then should be of type "ELF" when imported in ghidra.

#### 

#### Graphing View
<img src="" width="200">

Only when zooming in, it is then, that some functions / basic blocks start to appear:<br/>
<img src="" width="400">





---
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- r2con2021_deobfuscation GitHub Repo - https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
