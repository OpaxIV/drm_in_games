## Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop by Dr. Tim Blazytko.

## Definition of an VM Obfuscation
???

### What to look out for
- VM entry
- VM exit
- Dispatcher
- Handlers
- Register used as Virtual Stack Pointer
- ...

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
-  r2con2021_deobfuscation GitHub Repo - https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples
