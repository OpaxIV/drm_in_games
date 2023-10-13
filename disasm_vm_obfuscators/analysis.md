# Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop by Dr. Tim Blazytko (referenced bellow).
The goal is to find the most important components of the VM-based obfuscation and to present their functionality. Doing so will back up the reasoning behind the definition of each one of them.    

## Definition of an VM Obfuscation
The exact definition of an VM-based obfuscation has already been stated under the section https://github.com/OpaxIV/hslu_secproj/blob/0bc6c015a4a7fa69abbd0b94e3960d5773a84f95/disasm_vm_obfuscators/summary.md
In a couple of words one can say, that this obfuscation technique uses a costum instruction set architecture as the basis of its obfuscation. Is hides the original code in a sequence of bytes, which are then interpreted at runtime.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/55528869-41ab-4306-8412-19926d8b745e" width="600">
<br/>

## Analysis Procedure
During the analysis, the goal is to identify individual components of the VM-based obfuscator. At the beginning the graph view shall be used to get a broad idea of the binary. In a further step the goal is to identify each one the most important components of a Vm-based obfuscation. In the end, if possible, it shall be stated of which architecure (stack- or instruction-based, hybrid) the presented binary is.

The binary has been analysed with the reverse engineering tool ghidra. When selecting the language, I chose "x86, compiler: gcc". Please make sure to download the file directly from github or with the `curl` terminal command.
The file then should be of type "ELF" when imported in ghidra.


### Identification of the VM Components
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

Analyzing the disassembly in more detail will make it possible to understand the purpose of the rdx and rcx registers in this function. As already seen before the rdx register gets initialized at the beginning, then and then contains the bytecode (00101178). Looking at the graph will show, that nearly every handler (and some other components like the dispatcher) will contain at least an operation, that manipulates the named register rdx.
_Note: To show any occurences of e.g. registers, highlight the wished register and click on the mouse wheel in ghidra._ 
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d9546851-c640-4b96-a16b-1f92dfeff2a8" width="1100">
<br/>
Looking at the handlers will lead to the conclusion, that rdx is used as an virtual instruction pointer. Reason for this is, that most if not all handlers increment rdx by the instructions size.
Example of the handler at 0x1011a9:
By looking at the bytes, we can see that the first instruction starts the address 0x1011a9.
By Adding the amount of bytes of the current instruction (in this case 4) should land us onto the next instruction at 0x1011ad.

```
001011a9 ADD             48 83 c2 01   RDX,0x1
001011ad MOV             8b 01         EAX,dword ptr [RCX]=>local_128
001011af ADD             01 41 f8      dword ptr [RCX + local_130],EAX
001011b2 SUB             48 83 e9 08   RCX,0x8
001011b6 JMP             eb 4f         LAB_00101207
```
Using python (or any other tool that can process the following calculation), we get:
```
Python 3.11.4 (tags/v3.11.4:d2340ef, Jun  7 2023, 05:45:37) [MSC v.1934 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> hex(0x1011a9 + 4)
'0x1011ad'
>>>
```
Which is indeed the address of the next instruction.



Another important register is rcx:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/0f545161-d026-4c3f-8efc-73a860829ade" width="1100">
<br/>
Looking at the handler at the address 0x1011e1 will show the following code:
```
001011e1 MOV             8b 42 01                EAX,dword ptr [RDX + 0x1]=>DAT_00104061
001011e4 MOV             89 41 08                dword ptr [RCX + local_120],EAX
001011e7 ADD             48 83 c1 08             RCX,0x8
001011eb ADD             48 83 c2 05             RDX,0x5
001011ef JMP             eb 16                   LAB_00101207
```
What basically happens here is, that first the position on the stack at [RDX + 0x1] is popped and copied into EAX.
The value contained in EAX is then again copied and pushed back to the position [RCX + local_120] on the stack.
After doing so, RCX and RDX both get increment by a defined value and the control flow jumps back to the dispatcher.

In other words it is safe to assume that the register rcx is used as a virual stack pointer.


### Indentifing some Handler Functionalities
The number of handlers is vast. Hence only few will be analysed in great detail.

#### Handler 0x1011c4
By looking at the handler at the address 0x1011c4 one can see the following code:

```
001011c4 MOVSXD          48 63 42 01             RAX,dword ptr [RDX + 0x1]=>DAT_00104061
001011c8 ADD             48 01 f0                RAX,RSI
001011cb MOV             48 89 41 08             qword ptr [RCX + local_120],RAX
001011cf ADD             48 83 c1 08             RCX,0x8
001011d3 ADD             48 83 c2 05             RDX,0x5
001011d7 JMP             eb 2e                   LAB_00101207
```

As already defined, rdx is used as the virtual instruction pointer. So whatever is at [RDX + 0x1] will be copied into the register rax.
It follows an addition between rsi and rax, of which the result is then stored in rax. Since rcx was also defined as the virtual stack pointer, the result is then pushed onto the stack at the position [RCX + local_120].
Before the control flow is then handed again to the dispatcher, the virtual stack pointer decrementet in size by adding 0x8 and the instruction pointer is set to the next instruction.


#### Handler 0x101226
The handlers code at 0x101226 is shown as follows:

```
00101226 ADD             48 83 c2 01             RDX,0x1
0010122a MOV             48 8b 01                RAX,qword ptr [RCX]=>local_128
0010122d MOV             8b 79 f8                EDI,dword ptr [RCX + local_130]
00101230 MOV             89 38                   dword ptr [RAX],EDI
00101232 SUB             48 83 e9 10             RCX,0x10
00101236 JMP             eb cf                   LAB_00101207
```

0x1 is added to the virtual instruction pointer. The value currently in rcx is then moved into rax and the same occurs for [RCX + local_130] into edi.
The moved value from edi gets then again moved into the memory address at [RAX].
Before the control flow is then handed again to the dispatcher, the virtual stack pointer incremented in size by adding 0x10 and thus pointing on the top of the stack.


#### Handler 0x1281
This handler is different from the previous occurences. As seen by the graph view, the handler consists of multiple basic blocks:
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/b5252f42-d22a-4fd1-8271-b923e632cbd5" width="700">

The first block is represented by the following code:
```
00101281 CMP             83 39 00                dword ptr [RCX]=>local_128,0x0
00101284 JZ              74 12                   LAB_00101298
```
It is first checked if the top of the virtual stack [RCX] is equal to 0. If that is the case, then a jump occurs to the following basic block.
```
LAB_00101298
	00101298 ADD             48 83 c2 05             RDX,0x5
	0010129c JMP             eb f1                   LAB_0010128f
```
This basic block basically represents an if-statement. Hence if the condition is true, then the virtual instruction pointer gets incremented by 0x5.


On the other hand, if the condition is not satisfied, the "else" block is executed:
```
00101286 MOVSXD          48 63 42 01             RAX,dword ptr [RDX + 0x1]=>DAT_00104061
0010128a LEA             48 8d 54 02 01          RDX,[RDX + RAX*0x1 + 0x1]
```
The value at [RDX + 0x1] gets copied into rax. Furthermore the address [RDX + RAX*0x1 + 0x1] from the bytecode gets computed and loaded into rdx (hence pointing it to it).


In the end both statements end up here:
```
LAB_0010128f
    0010128f SUB             48 83 e9 08             RCX,0x8
    00101293 JMP             e9 6f ff ff ff          LAB_00101207
```
The virtual stack pointer is incremented by 0x8, hence pointing again at the top of the stack and the control flow is again passed to the dispatcher.





---
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- r2con2021_deobfuscation GitHub Repo - https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- Ghidra 101: Cursor Text Highlighting - https://www.tripwire.com/state-of-security/ghidra-101-cursor-text-highlighting
