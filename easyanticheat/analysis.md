# Analysis of the EasyAntiCheat.sys Binary

## Introduction
- write definition of an Anti-Cheat Engine
- write About the Easy Anti-Cheat Engine
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

@fabio: rewrite. Idea is to identify obfuscated code and MAYBE understand what the binary does.

### Before you start
#### Useful IDA64 Shortcuts
- `F5` on a Function - Generate Pseudocode
- `SHIFT` + `E` - Show List of Entry Points
- `SPACE` - Switch between Disassembly and Graph View


#### Opening the Binary
When opening the binary, IDA64 gives you two options:
<br>
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d316db4c-7c0c-420b-87b2-c9a68955592d" width="400">
<br/>
The first has been chosen for this analysis.

@tim: was ist der unterschied zu diesen beiden optionen? kommt es drauf an?

Choosing "AMD64 PE" as an option will lead to the following prompt, which can be accepted or denied (since in the end the program won't find anything anyway and you will end up at the same point).
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/824afc39-fb3a-4183-9af5-57f806b56fe3" width="300">


### Functions
The following functions were randomly chosen out of the list contained in the binary. The idea is to understand those subroutines and to map them (if possible) to an existing type of obfuscated code.

#### 0x140004EF0
The first function to look at can be found at the address 0x140004EF0. By looking at the graph view we can see some conditional jumps: 
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/a40e8bc5-5a78-4fdd-8803-88d637171bfd" width="600">
Interesting to understand would be, if this would be a possible candidate for an opaque predicate. For this we need to look further into the basic blocks, which contain those conditional jumps.








---
- Symbols and debugging information - https://hex-rays.com/blog/igors-tip-of-the-week-55-using-debug-symbols/
- Easy Anti-Cheat Website - https://easy.ac/en-us/#about
- IDA Pro Reverse Engineering Tutorial for Beginners - https://www.youtube.com/playlist?list=PLKwUZp9HwWoDDBPvoapdbJ1rdofowT67z
- EasyAntiCheat Exploit to inject unsigned code into protected processes - https://blog.back.engineering/10/08/2021/
- Unknown Cheats Forum - https://www.unknowncheats.me/forum/anti-cheat-bypass/447986-manual-map-mean.html
- What is a DLL - https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library
