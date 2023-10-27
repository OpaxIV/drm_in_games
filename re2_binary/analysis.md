# Analysis of the Resident Evil 2 Binary

## About the game
This writeup covers the analysis of the game "Resident Evil 2". The analysis will cover the Steams version of the game which will include, apart from the publishers own anti-piracy system, the Steam Digial Rights Management.

## Analysis Procedure
During the analysis, the goal is to identify individual components of the Steam DRM-system in the binary of RE2. At the beginning, the graph view shall be used to get a broad idea of the binary. In a further step the most important components of the XX obfuscation shall be identified. Some of these components shall be analysed further and presented in this file.
@ fabio: add type of obfuscation, VM based?

The binary has been analysed with the reverse engineering tool IDA64 (because Ghidra would be too sluggish to use when opening a 150 Mb binary).

### Steam DRM
The Steam DRM wrapper verifies games ownership and ensures that Steamworks features work properly by launching Steam before launching the game.
It is not an anti-piracy solution by itself. The Steam DRM protects against extremely casual piracy (i.e. copying all game files to another computer) and has some obfuscation, but it is easily removed by a motivated attacker.
What Steam suggests using in combination to the DRM is using Steamworks features which won't work on non-legitimate copies of the game (e.g. online multiplayer, achievements, leaderboards, trading cards, etc.).


### First Impression
Cold water, beginner analysis, graph view







---
- Resident Evil 2 on Steam - https://store.steampowered.com/app/883710/Resident_Evil_2/
- Steam DRM -https://partner.steamgames.com/doc/features/drm
