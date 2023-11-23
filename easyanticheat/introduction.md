# Analysis of the EasyAntiCheat.sys Binary

## Introduction
The following introduction shall provide an initial overview of one of the components of the Easy Anti-Cheat Engine ("EasyAntiCheat.sys").

### Definition of an Anti-Cheat Engine
Anti-cheat engines are wastly used in the gaming landscape. These tools' purpose is to recognize players, which use tools made for manipulating game software (cheating tools). Anti-cheat engines are mostly used in multiplayer games and can be deployed client-sided as an external program, client-sided as an integrated program or server-sided.
According to the website, "Easy Anti-Cheat is the industry-leading anti–cheat service, countering hacking and cheating in multiplayer PC games through the use of hybrid anti–cheat mechanisms".

### Definition of a Mapper
In the English language mapping is defined as an operation that associates each element of a given set (the domain) with one or more elements of a second set (the range). In the context of computers, a mapper is usually used in conjunction with processes (and programs). A widely used component in windows for example are the .dll (Dymanic-link Libraries). The use of DLLs helps promote modularization of code, code reuse, efficient memory usage, and reduced disk space. Certain external components, e.g. libraries or functions are then implemented at runtime, instead of the program having the same code as these external entities. By this the operating system and the programs load faster, run faster, and take less disk space on the computer.


## General Overview of the binary
The Easy Anti-Cheat Engine (Program) consists of many different components. The most important ones are presented in the following picture, including a small description for each one of them:
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/07f3ae69-14df-46d2-a3d3-495034cc5f77" width="700">
<br/>
It is again to be stated, that the analysis in this writeup only covers the EasyAntiCheat.sys binary.

### About the EasyAntiCheat.sys Module
Generally speaking the EasyAntiCheat.sys is a kernel module, acting as a driver for the actual program, which runs in the user space.
More precisely, the component works as a manual mapper: Basically manual mapping a driver is the same as manual mapping a DLL into a process. It follows a mapping of the driver binary into kernel memory, doing some fixups and then manually calling the EntryPoint. Manual mapping a portable executeable means to manually write a target in a way similar how the windows loader loads drivers and dlls. In other words, the EasyAntiCheat.sys component is a bridge between the kernel and user space and maps neccessary contents into the game running in the user land.

## Analysis
### Analysis Procedure
During the analysis, the goal is to identify individual obfuscated components in the Easy Anti-Cheat binary. At the beginning, the graph view shall be used to get a broad idea and in a further step some obfuscated components of the binary shall be analysed in detail. Furthermore the obfuscation technique of each of these components shall be identified (or at least assumed).
The binary has been analysed with the reverse engineering tools IDA64 and BinaryNinja.
All the analyzed functions can be found in a separate folder in this repository.

---

## Miscellaneous
#### Useful IDA64 Shortcuts
- `;` - Enter repeatable Comment
- `:` - Enter Comment
- `SPACE` - Switch between graph view and disassembly
- `SHIFT` + `E` - Show entry points
- `View` >  `Open subviews` > `Pseudocode` or hotkey `F5` - Show decompiler output (when focused on function)

#### Opening the Binary
When opening the binary, IDA64 gives you two options:
<br>
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d316db4c-7c0c-420b-87b2-c9a68955592d" width="400">
<br>
The first has been chosen for this analysis.

Choosing "AMD64 PE" as an option will lead to the following prompt, which can be accepted or denied (since in the end the program won't find anything anyway and you will end up at the same point).
<br>
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/824afc39-fb3a-4183-9af5-57f806b56fe3" width="300">
<br>

_Generally speaking, choosing the type of binary, as which to open the executable in IDA64 (or any other disassembler) should not make any difference.
It depends on the program, since opening the binary and defining it wrongly causes the decompiler to interpret the code in the wrong CPU architecture.
Fortunately this is mostly only the case when analyzing programs made for embedded systems._


---
- Symbols and debugging information - https://hex-rays.com/blog/igors-tip-of-the-week-55-using-debug-symbols/
- Easy Anti-Cheat Website - https://easy.ac/en-us/#about
- IDA Pro Reverse Engineering Tutorial for Beginners - https://www.youtube.com/playlist?list=PLKwUZp9HwWoDDBPvoapdbJ1rdofowT67z
- EasyAntiCheat Exploit to inject unsigned code into protected processes - https://blog.back.engineering/10/08/2021/
- Unknown Cheats Forum - https://www.unknowncheats.me/forum/anti-cheat-bypass/447986-manual-map-mean.html
- What is a DLL - https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library
