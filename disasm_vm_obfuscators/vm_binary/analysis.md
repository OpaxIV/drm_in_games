# Analysis of the vm_base.bin File

This documents contains an analysis (or an attempt at that) of the vim_base.bin binary of the r2con2021_deobfuscation workshop by Dr. Tim Blazytko (referenced bellow).
The goal is to find the most important components of the VM-based obfuscation and to present their functionality. Doing so will back up the reasoning behind the definition of each one of them.    

## Definition of an VM Obfuscation
The exact definition of an VM-based obfuscation has already been stated under the section https://github.com/OpaxIV/hslu_secproj/blob/0bc6c015a4a7fa69abbd0b94e3960d5773a84f95/disasm_vm_obfuscators/summary.md
<br>
In a couple of words one can say, that this obfuscation technique uses a costum instruction set architecture as the basis of its obfuscation. It hides the original code in a sequence of bytes, which are then interpreted at runtime.
<br/>
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/55528869-41ab-4306-8412-19926d8b745e" width="600">
<br/>

## Analysis Procedure
During the analysis, the goal is to identify individual components of the VM-based obfuscator. At the beginning, the graph view shall be used to get a broad idea of the binary. In a further step the most important components of a VM-based obfuscation shall be identified. In a last step the VM-architecure of the binary is to be stated.

The binary has been analysed with the reverse engineering tool ghidra. When selecting the language, I chose "x86, compiler: gcc". Please make sure to download the file directly from github or with the `curl` terminal command.
The file then should be of type "ELF" when imported in ghidra.


### Identification of the VM Components
Starting at the adress 0x10115a, one can get a broad overview of the VM-obfuscation. The graph already presents a similarity to the picture of the general structure from above:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/4b48d578-4dba-41f7-abce-0f3af633d01a" width="700">
<br/>
Unfortunately, even with tweaking and changing between the different graphing views, it is not possible to get the "perfect" perspective of the graph. Using the "nested code layout" will remove the hierarchical structure but present a better view of the "path" the exectution of the function follows. A combined approach should be taken to get the full picture.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5e576e5e-96c4-4a77-a430-1a4b7a84d245" width="200">
<br/>
In any way, the first thing to point out is the VM-entry, which can be found at the address 0x10115a. The reasoning behind this assumption is the lack of any incoming arrows from the basic block:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/621d6309-fbbc-4c30-a7b7-fecd2ae956ef" width="600">
<br/>

The VM-dispatcher can be found just by looking at the colors and direction of the arrows. The dispatcher, being at the address 00101207, has the most outgoing and incoming arrows compared to all other basic blocks.
Furthermore, as we will see when analyzing some handler functions, the returning point of every handler is the location 0x10120a, which is indeed our dispatcher:
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


Furthermore a great amount of so-called "handlers" can be recognized. Looking at the handler 0x1011a9: This basic block is undirectly coming from and going directly back to the dispatcher at the address 0x0101207.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/344b30f3-e817-4eb6-886a-d31d0352b5cc" width="300">
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/12a2e7e6-84ff-42e2-b2c8-5badecd7e723" width="600">
<br/>

Analyzing the disassembly in more detail will make it possible to also understand the purpose of the `rdx` and `rcx` registers in this function.
#### RDX - Virtual Instruction Pointer
As already seen before, the `rdx` register gets initialized in the VM entry and contains the byte code (address at 0x101178). This byte code represents the instructions from the native CPU, translated into the custom VM Instruction Set Architecure (ISA). It can be seen as an array, of which the entries are instructions, sequences defined as bytes.
Looking at the graph will show, that nearly every handler (and some other components like e.g. the dispatcher) will at least once in their basic block manipulate the named register `rdx`.
_Note: To show any occurences of e.g. registers, highlight the wished register and click on the mouse wheel in ghidra._ 
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d9546851-c640-4b96-a16b-1f92dfeff2a8" width="1100">
<br/>
Looking at the handlers will further emphasize the conclusion, that `rdx` is used as an virtual instruction pointer. Most if not all handlers increment `rdx` by the current instructions size.
Example of the handler at the address 0x1011a9:
By looking at the bytes, we can see that the first instruction starts at the address 0x1011a9.
An addition of the amount of bytes of the current instruction (in this case 4 bytes), should result into the next instructions address (0x1011ad).

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
_Important: All instructions run on the native CPU. Only in combination they compute a virtual instruction._

#### RCX - Virtual Stack Pointer
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
The value contained in EAX is then again copied and pushed back onto the position [RCX + local_120] on the stack.
After doing so, `rcx` and `rdx` both get increment by a defined value and the control flow jumps back to the dispatcher.

In other words it is safe to assume that the register `rcx` is used as a virual stack pointer.

Combining especially those two foundings we can say, that this binary contains a VM obfuscator of the **"stack-based" architecture** type. 

### Indentifing Handler Functionalities
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

As already defined, `rdx` is used as the virtual instruction pointer. So whatever is at [RDX + 0x1] will be copied into the register `rax`.
It follows an addition between `rsi` and `rax`, of which the result is then stored in `rax`. Since `rcx` was also defined as the virtual stack pointer, the result is then pushed onto the stack at the position [RCX + local_120].
Before handling the control flow back again to the dispatcher, the virtual stack pointer gets decrementet in size by adding 0x8 and the instruction pointer repositioned to point to the next instruction.


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

The hexadecimal value 0x1 is added to the virtual instruction pointer. The value currently in [rcx], gets dereferenced and then moved into `rax`. The same occurs for `edi`, in which the dereferenced value of [RCX + local_130] gets copied.
The value from `edi` gets then again moved into the address in memory at [rax].
Before the control flow is then handed back again to the dispatcher, the virtual stack pointer is incremented by adding 0x10 and thus pointing to the top of the stack.


#### Handler 0x101281
This handler is different from the previous occurences. As seen by the graph view, the handler consists of multiple basic blocks:
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/b5252f42-d22a-4fd1-8271-b923e632cbd5" width="700">

The first block is represented by the following code:
```
00101281 CMP             83 39 00                dword ptr [RCX]=>local_128,0x0
00101284 JZ              74 12                   LAB_00101298
```
It is first checked if the top of the virtual stack [rcx] is equal to 0. If that is the case, then a jump occurs to the block at 0x101298. This clearly represents a condition statement like e.g. an if-statement.

In the first basic block, the virtual instruction pointer gets incremented by 0x5. The follow up instruction then handles the control flow via JMP-instruction to the basic block at 0x10128f.
```
LAB_00101298
	00101298 ADD             48 83 c2 05             RDX,0x5
	0010129c JMP             eb f1                   LAB_0010128f
```

On the other hand, if the condition is not satisfied, the "else" block is executed:
```
00101286 MOVSXD          48 63 42 01             RAX,dword ptr [RDX + 0x1]=>DAT_00104061
0010128a LEA             48 8d 54 02 01          RDX,[RDX + RAX*0x1 + 0x1]
```
The value at [RDX + 0x1] gets copied into `rax`. Furthermore the address [RDX + RAX*0x1 + 0x1]  gets computed from the byte code and loaded into `rdx` (hence pointing it to it).


In the end both statements end up here:
```
LAB_0010128f
    0010128f SUB             48 83 e9 08             RCX,0x8
    00101293 JMP             e9 6f ff ff ff          LAB_00101207
```
The virtual stack pointer is incremented by 0x8, pointing again on top of the stack and the control flow is once again passed onto the dispatcher.


The last component we can spot is the VM- xit: It is to be found at the adress 0x101245:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/a87b5790-c8b9-402a-bf47-fa16da1100e8" width="1000">
<br/>

The most notible aspects of the assembly instructions are:
The `JNZ LAB_001012b9` instruction, which checks the stack canarys value (at address 001012b9):

```
LAB_001012b9                                    XREF[1]:     00101258(j)  
	001012b9 e8 72 fd        CALL       <EXTERNAL>::__stack_chk_fail                     undefined __stack_chk_fail()
			 ff ff
					 -- Flow Override: CALL_RETURN (CALL_TERMINATOR)
```
Most notable are the `MOV EAX,dword ptr [RCX]=>local_128` instruction, which copies the return value into `eax`, the `ADD RSP,0x138` instruction, which restores the original non-virtual stacks state, and the `RET` instruction, which lets the control flow return.
```
LAB_00101245                                    XREF[1]:     0010121a(j)  
	00101245 8b 01           MOV        EAX,dword ptr [RCX]=>local_128
	00101247 48 8b 94        MOV        RDX,qword ptr [RSP + local_10]
			 24 28 01 
			 00 00
	0010124f 64 48 2b        SUB        RDX,qword ptr FS:[0x28]
			 14 25 28 
			 00 00 00
	00101258 75 5f           JNZ        LAB_001012b9
	0010125a 48 81 c4        ADD        RSP,0x138
			 38 01 00 00
	00101261 c3              RET
```
 These specifications further underline the fact, that this is the exit point of the VM-obfuscator.

#### Type Architecure Type

stack-based architecture
• pop arguments from stack
• push results onto stack
• examples: JVM, CPython, WebAssembly, …

register-based architecture
• pass arguments in virtual registers
• store results in virtual registers
• examples: Dalvik, Lua, LLVM, …
• hybrid architectures possible



---
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- r2con2021_deobfuscation GitHub Repo - https://github.com/mrphrazer/r2con2021_deobfuscation/tree/main/samples
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- Ghidra 101: Cursor Text Highlighting - https://www.tripwire.com/state-of-security/ghidra-101-cursor-text-highlighting
