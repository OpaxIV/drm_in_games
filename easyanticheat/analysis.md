# Analysis of the EasyAntiCheat.sys Binary

## Introduction

### Definition of an Anti-Cheat Engine
@fabio 2do

### About the Anti-Cheat Engine
According to the website, "Easy Anti-Cheat is the industry-leading anti–cheat service, countering hacking and cheating in multiplayer PC games through the use of hybrid anti–cheat mechanisms".


## Analysis
### Analysis Procedure
During the analysis, the goal is to identify individual components of the Steam DRM-system in the binary of RE2. At the beginning, the graph view shall be used to get a broad idea of the binary. In a further step the most important components of the XX obfuscation shall be identified. Some of these components shall be analysed further and presented in this file.
@ fabio: add type of obfuscation, VM based?

The binary has been analysed with the reverse engineering tool IDA64 (because Ghidra would be too sluggish to use when opening a 150 Mb binary).

### Before you start
Settings in IDA (type of binary etc)
<CTRL> + <E>

setting entry point

graphing

### First Impression
Cold water, beginner analysis, graph view




# Temp Screenshots:






---
- Symbols and debugging information - https://hex-rays.com/blog/igors-tip-of-the-week-55-using-debug-symbols/
- Easy Anti-Cheat Website - https://easy.ac/en-us/#about
