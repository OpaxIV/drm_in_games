# Analysis of the EasyAntiCheat.sys Binary

## Introduction

### Definition of an Anti-Cheat Engine
@fabio 2do

### About the Anti-Cheat Engine
According to the website, "Easy Anti-Cheat is the industry-leading anti–cheat service, countering hacking and cheating in multiplayer PC games through the use of hybrid anti–cheat mechanisms".


## Analysis
### Analysis Procedure
During the analysis, the goal is to identify individual obfuscated components in the Easy Anti-Cheat binary. At the beginning, the graph view shall be used to get a broad idea and in a further step some obfuscated components of the binary shall be analysed in details. Furthermore the obfuscation technique of each of these components shall be identified. Finally the broad way of functioning of this binary shall be stated.
The binary has been analysed with the reverse engineering tool IDA64.

### Before you start
#### Useful IDA64 Shortcuts
- `F5` on a Function - Generate Pseudocode
- `SHIFT` + `E` - Show List of Entry Points




#### Opening the Binary
When opening the binary, IDA64 gives you two options:
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d316db4c-7c0c-420b-87b2-c9a68955592d" width="400">
The first has been chosen for this analysis.
@ tim: was ist der unterschied zu diesen beiden optionen? kommt es drauf an?

Choosing "AMD64 PE" as an option will lead to the following prompt, which can be accepted or denied (since in the end the program won't find anything anyway and you will end up at the same point).
<img src ="https://github.com/OpaxIV/hslu_secproj/assets/93701325/824afc39-fb3a-4183-9af5-57f806b56fe3" width="300">


### Graphing View




# Temp Screenshots:






---
- Symbols and debugging information - https://hex-rays.com/blog/igors-tip-of-the-week-55-using-debug-symbols/
- Easy Anti-Cheat Website - https://easy.ac/en-us/#about
- IDA Pro Reverse Engineering Tutorial for Beginners - https://www.youtube.com/playlist?list=PLKwUZp9HwWoDDBPvoapdbJ1rdofowT67z
