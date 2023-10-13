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


#### Identification of VM Components
Starting at the adress 0010115a, one can get a broad overview of the VM-obfuscation. The graph already presents a similarity to the picture of the general structure above:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/4b48d578-4dba-41f7-abce-0f3af633d01a" width="700">
<br/>
Unfortunately, even with tweaking the different graphing views, it is not possible to get the "perfect" view of the graph. Using the "nested code layout" will remove the hierarchical structre but present a better view of the "path" the exectution of the function takes.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5e576e5e-96c4-4a77-a430-1a4b7a84d245" width="200">
<br/>
In any way, one can spot the VM dispatcher just by looking at the colors and direction of the arrows. The dispatcher at the adress 00101207 has the most outgoing and incoming arrows compared to all other basic blocks.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5b11ccea-a063-45c8-aff4-3a3e2a3cfca1" width="500">
<br/>

A further component to look out for is the bytecode. In this example, the bytecode can be found in the VM entry at the address 00101178.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/743e1229-ca82-4a1c-9c2a-721d0caf136a" width="300">
<br/>
```
        00101178 48 8d 15        LEA        RDX,[DAT_00104060]                               = D5h
                 e1 2e 00 00
```


Furthermore one will immediately identify a large number of handlers. As an example, let us pick the one at address 001011a9. This basic block is undirectly comming from and going directly back to the dispatcher at address 00101207.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/344b30f3-e817-4eb6-886a-d31d0352b5cc" width="300">
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/12a2e7e6-84ff-42e2-b2c8-5badecd7e723" width="600">
<br/>




- What are the functions of rdx and rcx?




---
TEMP (to remove or rearrange afterwards):

#### Graphing View


Only when zooming in, it is then, that some functions / basic blocks start to appear:<br/>
<img src="" width="400">



#### Recovering Handler Semantics I
Open the sample vm_basic.bin and analyze the handler at 0x11e1.
- How fetches the handler its argument?
- What does it do with the argument?
- What else does the handler do?



#### Recovering Handler Semantics II
Open the sample vm_basic.bin and analyze the handler at 0x11a9.
- How fetches the handler its arguments?
- What does it compute?
- What else does the handler do?

#### Recovering Handler Semantics III
Open the sample vm_basic.bin and analyze the handler at 0x1281.
- What does the handler check?
- Why does it branch?
- What does the handler do with rdx and rax?

#### Recovering Handler Semantics ???




---
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- r2con2021_deobfuscation GitHub Repo - https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
