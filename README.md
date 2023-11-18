# Digial Rights Management (DRM) in Video Games
## Security Project (SecProj) at the Hochschule Luzern
This repository contains all data regarding the Security Project.
<br>
I want to aknowledge the help and insights, which Dr. Tim Blazytko provided me for this paper. I sincerely thank him for the support.  

## Sources
All (re-)sources, which served as a basis for this project, have been referenced extensively in eacht individual chapter. 

## Milestones
- 19.09.2023 Beginning of the Project
- 
@fabio: wegnehmen sofern nicht vorgegeben



## Basic Roadmap
<br>

1. Read: https://nicolo.dev/en/blog/fairplay-apple-obfuscation/
<br>&nbsp;1.1 Understand the fundamentals of the article
<br>&nbsp;1.2 Summarize the article in own words.
<br>

2. Install and understand the Tigress Obfuscation Software
<br>&nbsp;2.1 Get a simple C program and obfuscate it with tigress. e.g. https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
<br>&nbsp;2.2 Look at the binary compiled binary with e.g. Ghidra
<br>&nbsp;&nbsp;2.2.1 Focus on the following obfuscation techniques (one per one):
    - Control Flattening
    - Opaque Predicates
    - Encode Arithmetic
<br>

3. Read: https://synthesis.to/2021/10/21/vm_based_obfuscation.html
<br>&nbsp;3.1 Summarize the article in own words.
<br>&nbsp;3.2 Analyze the vm_base.bin binary.
<br>

4. Analyze the EasyAntiCheat.sys binary.
<br>&nbsp;4.1 Focus on some of it's functions.
<br>&nbsp;4.2 Focus on the whole binary and try to "quickly" spot (with the acquired knowledge) obfuscated code.
<br>

## Annotations
- Multiple folders can be found with names like "archive" oder "obsolete". These folders contain mainly analysis, which were not further elaborated upon. This is due to time constraint or "wrong directions", which would else been taken.
- The folder (<a href="hslu_secproj/disasm_vm_obfuscators/symb_exec">"symb_exec"</a>) contains a semi finished analysis of the symbolic execution technique. Unfortunately, this is also unfinished due to not really being in the scope of this project and to time constraint.
