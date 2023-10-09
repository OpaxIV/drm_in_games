# hslu_secproj
Repo for the Security Project at the Hochschule Luzern


## Personal 2DO
- Summarize VM Article and Main Takeaways (replace copy-paste parts)
- May try to properly analyze the opaque predicates tigress sample 

---

## Milestones
- 19.09.2023 Beginning of the Project
- 


## Resources
- https://tigress.wtf/flatten.html
- https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- https://megagames.com/fixes/resident-evil-2-2019-v20191218-all-no-dvd-codex
- https://ceres-c.it/2021/11/21/DRM-reversing/
- https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- https://nicolo.dev/en/blog/fairplay-apple-obfuscation/

## Working Plan

1. Read: https://nicolo.dev/en/blog/fairplay-apple-obfuscation/

2. Install and understand the Tigress Obfuscation Software

2.1 Get a simple C program and obfuscate it with tigress.
e.g. https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c

2.2 Look at the binary compiled binary with e.g. Ghidra
<br/>
  2.2.1 Focus on the following obfuscation techniques (one per one):
  - Control Flattening
  - Opaque Predicates
  - Encode Arithmetic

3. Read: https://synthesis.to/2021/10/21/vm_based_obfuscation.html




XX. Try to understand the DRM system of the RE2 Remake: https://megagames.com/fixes/resident-evil-2-2019-v20191218-all-no-dvd-codex
