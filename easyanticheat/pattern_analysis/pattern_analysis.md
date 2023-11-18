# Pattern Analysis
After having consulted multiple writeups and analysed a set of functions from programs, the idea of the following writeup is document the steps, that are used to quickly identify obfuscated code in a binary. It is certain, that a professional might use other ways to complete this task. This document should provide in insight in the procedure from a beginner's point of view.

## Table of Contents
1. [Analysis Procedure](#analysisprocedure)
2. [Analysis of a Random Set of Functions](#randomanalysis)
3. [Project Conclusion](#typeofobfuscation)

## Analysis Procedure <a name="analysisprocedure"></a>
The goal is to quickly find and identify obfuscated code in the binary. Hence the idea is to go trough the list of functions and trying to find a set of functions, which contain some general classifiers for obfuscated code: 
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly.
- The function's code does not represent "normal / usual" code.

This analysis shall cover a handfull of functions, which are indeed obfuscated and to make a point, some functions which were correctly identified as not being obfuscated.

## Analysis of a Random Set of Functions <a name="randomanalysis"></a>
### sub_14000590c (Obfuscated)
- The function is only made up by a single basic block. - **FALSE**

<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/a894a973-ea02-45a4-ac23-916405a2c733" width="200">
<br>

- The function contains rare or unusual instructions in the disassembly. - **FALSE**
- The function's code does not represent "normal / usual" code. - **TRUE**
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/254b00d7-21e5-4562-9706-d6bd7a444623" width="850">
<br>
  - By looking at the pseudo C-code (or the disassembly) we can see multiple ussages of XOR-instructions and byte shiftig operations.

- Further Annotations:
  - exit jump to register
  - many XOR and strange pseudo code
  - many jump instructions

<br>

### XXX (Obfuscated)
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly.
- The function's code does not represent "normal / usual" code.
- Other findings:
<br>

### XXX (Obfuscated)
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly.
- The function's code does not represent "normal / usual" code.
- Other findings:
<br>

### XXX (Not Obfuscated)
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly.
- The function's code does not represent "normal / usual" code.
- Other findings:
<br>

### XXX (Not Obfuscated)
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly.
- The function's code does not represent "normal / usual" code.
- Other findings:
<br>

@tim: to check if asumptions are true, really obfuscated or not?




## Analysis Procedure <a name="analysisprocedure"></a>
