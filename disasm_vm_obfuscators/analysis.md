## Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop by Dr. Tim Blazytko (linked bellow).

## Definition of an VM Obfuscation
???

### What to look out for
- VM entry
- VM exit
- Dispatcher
- Handlers
- Register used as Virtual Stack Pointer (OR)
- Register used as Virtual Instrucion Pointer (OR)
- ...
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/55528869-41ab-4306-8412-19926d8b745e" width="600">


### How to proceed
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
