## Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop.
Ref: https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples

## Definition of an VM Obfuscation

### Ghidra
The binary has been analysed with ghidra. When selecting the language, I chose "x86, compiler: gcc".

#### Before your start
Make sure to download the file directly from github or with the `curl` terminal command.
The file then should be of type "ELF" when imported in ghidra.

#### Graphing View
<img src="" width="200">

Only when zooming in, it is then, that some functions / basic blocks start to appear:<br/>
<img src="" width="400">
