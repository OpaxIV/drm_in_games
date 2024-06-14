# Digital Rights Management (DRM) in Video Games
## Security Project (SecProj) at the Hochschule Luzern
This repository contains all data regarding the Security Project.

### Disclaimer
This repository was first made public in June of 2024, before of which any copyrighted material has been removed. It is to be noted, that all information found here is for educational purposes only and all rights belong to the respective owners.

### Acknowledgements
I would like to express my special thanks to Dr. Tim B., who provided me his help and his time during this project.<br>
I sincerely thank him for the support.  

### Sources
All (re-)sources, which served as a basis for this project, have been referenced extensively in each individual chapter.

----------------------------------------------------------------------------------------
### Basic Roadmap
1. Read: https://nicolo.dev/en/blog/fairplay-apple-obfuscation/
<br>&nbsp;1.1 Understand the fundamentals of the article
<br>&nbsp;1.2 Summarize the article in own words.
<br>

2. Install and understand the Tigress Obfuscation Software
<br>&nbsp;2.1 Get a simple C program and obfuscate it with tigress. e.g. https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
<br>&nbsp;2.2 Look at the compiled binary with e.g. Ghidra
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
<br>&nbsp;4.1 Focus on some of its functions.
<br>&nbsp;4.2 Focus on the whole binary and try to "quickly" spot (with the acquired knowledge) obfuscated code.
<br>


----------------------------------------------------------------------------------------
### Annotations
- Multiple folders can be found with names like "archive" or "obsolete". These folders contain mainly analyses, which were not further elaborated upon. This was due to time constraints or "wrong directions", which would have else been taken.
- The file (<a href="hslu_secproj/disasm_vm_obfuscators/symb_exec/analysis.md">"symb_exec"</a>) contains a semi finished analysis of the symbolic execution technique. Unfortunately, this is also unfinished due to not really being in the scope of this project and (again) due to time constraints.
