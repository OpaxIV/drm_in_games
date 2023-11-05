# Analysis of the EasyAntiCheat.sys Binary

## Introduction
- write definition of an Anti-Cheat Engine
- write About the Anti-Cheat Engine
According to the website, "Easy Anti-Cheat is the industry-leading anti–cheat service, countering hacking and cheating in multiplayer PC games through the use of hybrid anti–cheat mechanisms".
@fabio 2do

## General Overview
The Easy Anti-Cheat Engine (Program) consists of many different components. The most important ones are presented in the following picture, including a small description for each one of them:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/07f3ae69-14df-46d2-a3d3-495034cc5f77" width="700">
<br/>
It is to be stated, that the analysis in this writeup only covers the EasyAntiCheat.sys binary.



### Definition of a Mapper
In the English language mapping is defined as an operation that associates each element of a given set (the domain) with one or more elements of a second set (the range). In the context of computers, a mapper is usually used in conjunction with processes (programs). A widely used component in windows for example are the .dll (Dymanic-link Libraries). The use of DLLs helps promote modularization of code, code reuse, efficient memory usage, and reduced disk space. Certain external components, e.g. libraries or functions are then implemented at runtime, instead of the program having the same code as these external entities. By this the operating system and the programs load faster, run faster, and take less disk space on the computer.



### About the EasyAntiCheat.sys Module
- Manual mapper:
- Basically manual mapping a driver is essentially the same idea as manual mapping a DLL into a process. You are mapping the driver binary into kernel memory, doing some fixups and then you manually call the EntryPoint. This usually is done with the help of a signed vulnerable driver, that exposes a way to read and write to kernel memory over IOCTL. This circumvents DSE and all other official windows mechanisms, that ACs could use to detect your loaded unsinged driver.
- Manual mapping a portable executeable means you manually write target in a way similar how the windows loader loads drivers and dlls. You need to fix the
<br> temp
- allocates extra memory around it's memory -> dynamic code
- maps contents into the game
- 


@fabio: rewrite and understand


## Analysis
### Analysis Procedure
During the analysis, the goal is to identify individual obfuscated components in the Easy Anti-Cheat binary. At the beginning, the graph view shall be used to get a broad idea and in a further step some obfuscated components of the binary shall be analysed in details. Furthermore the obfuscation technique of each of these components shall be identified. Finally the broad way of functioning of this binary shall be stated.
The binary has been analysed with the reverse engineering tool IDA64.

### Before you start
#### Useful IDA64 Shortcuts
- `F5` on a Function - Generate Pseudocode
- `SHIFT` + `E` - Show List of Entry Points
- `SPACE` - Switch between Disassembly and Graph View

@fabio: add rest of found shortcuts


#### Opening the Binary
When opening the binary, IDA64 gives you two options:
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d316db4c-7c0c-420b-87b2-c9a68955592d" width="400">
The first has been chosen for this analysis.
@ tim: was ist der unterschied zu diesen beiden optionen? kommt es drauf an?

Choosing "AMD64 PE" as an option will lead to the following prompt, which can be accepted or denied (since in the end the program won't find anything anyway and you will end up at the same point).
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/824afc39-fb3a-4183-9af5-57f806b56fe3" width="300">



### Looking at some Functions
The following functions were randomly chosen out of the list contained in the binary.

#### 0x140004EF0
The first function to look at can be found at the address `0x140004EFE`. By looking at the graph view we can see some conditional jumps: 
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/a40e8bc5-5a78-4fdd-8803-88d637171bfd" width="600">
Interesting to understand would be, if this would be a possible candidate for an opaque predicate. For this we need to look further into the basic blocks, which contain those conditional jumps.

The first conditional jump is to be found in the first basic block at `0x140004EFE` specifically:
```asm
.text:0000000140004EF0
.text:0000000140004EF0
.text:0000000140004EF0
.text:0000000140004EF0 sub_140004EF0 proc near
.text:0000000140004EF0 sub     rsp, 28h
.text:0000000140004EF4 mov     rax, cs:qword_14007E2F8
.text:0000000140004EFB test    rax, rax
.text:0000000140004EFE jnz     short loc_140004F52
```
The important part here would be the `test rax, rax` instruction, by which the control flow is controlled. Generally speaking, in the x86 assembly language, the `TEST` instruction performs a bitwise AND on two operands. If the result is 0, the Zero Flag (ZF) is set to 1, otherwise set to 0.<br>
An example would be the following:
```asm
; Conditional Jump with NOT
test cl, cl   ; set ZF to 1 if cl == 0
jnz 0x8004f430  ; jump if ZF == 0, hence cl != 0
```
In the case of our disassembly, we need to check wheter the value of `rax` fulfills this condition.

@tim: view value of rax, is jump taken?
@tim: possible opaque predicate
@fabio: write rest of the text

#### 0x1400299FC
When opening the function at 0x1400299FC we can already spot some similarities to the vm-based obfuscation technique.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e32cccd5-69a0-4d68-9454-514140832a9f" width="300">
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/53aca78c-1875-46fd-8cff-95992fdf9f2b" width="600">
<br/>
The high amount of branches which all lead to a central point, would point us further to this conclusion. To elaborate further on this point, we need to analyse the function in greater detail. At first we need to make assumptions about the functioning of the individual components to see if they could map to the functional parts of a vm-based obfuscation.
<br>

##### VM-Entry (?)
At first we can see some sort of vm-entry directly at the beginning of the function:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/7f0a9a81-f6b6-4b44-9b02-ac4fd246c933" width="300">
<br/>
By looking further into the disassembly we can see the usual "preparations" when a function is called, in e.g. the pushing of registers and making of space on the stack.
```asm
[...]
text:00000001400299FC                  mov     [rsp-8+arg_10], rbx
.text:0000000140029A01                 mov     [rsp-8+arg_18], r9d
.text:0000000140029A06                 push    rbp
.text:0000000140029A07                 push    rsi
.text:0000000140029A08                 push    rdi
.text:0000000140029A09                 lea     rbp, [rsp-3Fh]
.text:0000000140029A0E                 sub     rsp, 0D0h
.text:0000000140029A15                 xor     eax, eax
.text:0000000140029A17                 xorps   xmm0, xmm0
.text:0000000140029A1A                 mov     rsi, r8
.text:0000000140029A1D                 mov     [rbp+4Fh+var_70], rax
.text:0000000140029A21                 mov     ebx, edx
.text:0000000140029A23                 mov     rdi, rcx
.text:0000000140029A26                 xor     edx, edx
.text:0000000140029A28                 lea     rcx, [rbp+4Fh+var_60]
.text:0000000140029A2C                 lea     r8d, [rax+48h]
.text:0000000140029A30                 movups  [rbp+4Fh+var_90], xmm0
.text:0000000140029A34                 movups  [rbp+4Fh+var_80], xmm0
.text:0000000140029A38                 call    sub_1400586C0
.text:0000000140029A3D                 mov     [rbp+4Fh+var_98], rdi
.text:0000000140029A41                 mov     [rbp+4Fh+arg_8], ebx
```
Most notable is at the address `0x140029A38`, at which a subroutine is called, leading to `0x1400586C0`:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/f786dc97-0361-4965-95d5-b1e94b7cf09c" width="600">
<br/>
The first basic block contains a conditional jump, which is executed if the value in `r8` is greater than `0x8`, executing the left set of branches.
```asm
[...]
.text:00000001400586C3 cmp     r8, 8
.text:00000001400586C7 jb      short loc_140058710
[...]
```
As we can see, the left set of branches is made up of three basic blocks:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/7ff7a62c-a125-49d9-8343-b572f985e47a" width="400">
<br/>
It gives us two possibilities: If the value in register `r8` is equal to 0 (hence the result of the AND operation being 0), then we directly jump to the last basic block in the set at `0x140058720`. On the other hand, if `r8` is indeed not equal to 0, the control flow gets passed to the middle block and a loop occurs. This loop decrements the value of `r8` until it is equal to 0 (`dec     r8`), which finally also leads to the last basic block and returning us to the proceeding calling function.




---
- Symbols and debugging information - https://hex-rays.com/blog/igors-tip-of-the-week-55-using-debug-symbols/
- Easy Anti-Cheat Website - https://easy.ac/en-us/#about
- IDA Pro Reverse Engineering Tutorial for Beginners - https://www.youtube.com/playlist?list=PLKwUZp9HwWoDDBPvoapdbJ1rdofowT67z
- EasyAntiCheat Exploit to inject unsigned code into protected processes - https://blog.back.engineering/10/08/2021/
- Unknown Cheats Forum - https://www.unknowncheats.me/forum/anti-cheat-bypass/447986-manual-map-mean.html
- What is a DLL - https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library
- TEST (x86 instruction) - https://en.wikipedia.org/wiki/TEST_(x86_instruction)
