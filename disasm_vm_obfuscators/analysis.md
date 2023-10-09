## Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop.
Ref: https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples

## Definition of an VM Obfuscation

### Ghidra
The binary has been analysed with ghidra. When selecting the language, I chose "x86, compiler: gcc".
#### Graphing View
When opening the binary with the graph view, one can see a semingly never ending graph.<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/02ad9701-acc8-4ca4-b97c-bdc050513e53" width="200">

Only when zooming in, it is then, that some functions / basic blocks start to appear:<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/fd8402df-dc07-4176-8a4a-c4b4e17659b9" width="400">
