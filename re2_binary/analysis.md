# Analysis of the Resident Evil 2 Binary

## Introduction
### About the game
This writeup covers the analysis of the game "Resident Evil 2". The analysis will be taken on the Steams version of the game which will include, apart from the publishers own anti-piracy system, the Steam Digial Rights Management.

### Definition of a DRM
Digital rights management (DRM) is the use of technology to control and manage access to copyrighted material. This means that not the owner of the digital content decides, if he can watch it, but rather a software decides if the user has the neccessary permissions to watch it. DRM aims to protect the copyright holder’s rights and prevents content from unauthorized distribution and modification.
@ fabio umschreiben in eigenen worten

### About the Steam DRM
The Steam DRM wrapper verifies games ownership and ensures that Steamworks features work properly by launching Steam before launching the game.
It is not an anti-piracy solution by itself. The Steam DRM protects against extremely casual piracy (i.e. copying all game files to another computer) and has some obfuscation, but it is easily removed by a motivated attacker.
What Steam suggests using in combination to the DRM is using Steamworks features which won't work on non-legitimate copies of the game (e.g. online multiplayer, achievements, leaderboards, trading cards, etc.).

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

![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/3b5dd815-7384-4016-b18d-3c4cdeb4d18a)
![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/184ddcaa-35b7-44f1-b10e-2e1e2c4041ab)

entry points CTRL + E:
![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/97fe3a24-cfe6-48e9-b0c7-6430d6768458)

Func: CAKRegistryMgr(...)
![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/a9f1fff8-3ffb-4117-9967-3ea648a915a0)

![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/0acd15ed-e0d0-4de4-9244-157fd8f8c68a)

![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/3fc24d89-7096-4e9c-9532-7de6955c6775)

![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/ce0c8d9f-5153-4eec-9427-d0276f92386f)

![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/f7d9b0ae-d657-410f-87f2-b39d18d50156)










---
- Resident Evil 2 on Steam - https://store.steampowered.com/app/883710/Resident_Evil_2/
- Steam DRM -https://partner.steamgames.com/doc/features/drm
- Symbols and debugging information - https://hex-rays.com/blog/igors-tip-of-the-week-55-using-debug-symbols/
