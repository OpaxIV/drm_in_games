# Pattern Analysis
After having consulted multiple writeups and analysed a set of functions from programs, the idea of the following writeup is document the steps, that are used to quickly identify obfuscated code in a binary. It is certain, that a professional might use other ways to complete this task. This document should provide in insight in the procedure from a beginner's point of view.

## Table of Contents
1. [Analysis Procedure](#analysisprocedure)
2. [Analysis of a Random Set of Functions](#randomanalysis)
3. [Project Conclusion](#typeofobfuscation)

## Analysis Procedure <a name="analysisprocedure"></a>
The goal is to quickly find and identify obfuscated code in the binary. Hence the idea is to go trough the list of functions and trying to find a set of functions, which contain some general classifiers for obfuscated code: 
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly / pseudo C-code.
- The function's code does not represent "normal / usual" code.

This analysis shall cover a handfull of functions, which are indeed obfuscated and to make a point, some functions which were correctly identified as not being obfuscated.

## Analysis of a Random Set of Functions <a name="randomanalysis"></a>
### sub_14000590c (Obfuscated)
- The function is only made up by a single basic block. - **FALSE**

<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/a894a973-ea02-45a4-ac23-916405a2c733" width="200">
<br>

- The function contains rare or unusual instructions in the disassembly / pseudo C-code. - **FALSE**
- The function's code does not represent "normal / usual" code. - **TRUE**
  - By looking at the pseudo C-code (or the disassembly) we can see multiple ussages of XOR-instructions and byte shiftig operations.
  - If we were to combine some of the used expressions, we would get code similar to Arithmetic Encoding.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/254b00d7-21e5-4562-9706-d6bd7a444623" width="850">
<br> 

- Further Annotations:
  - The last instruction is a jump to a register (`1401a9837  jmp     rsi`).
  - Many jump instructions, may also be an opaque predicate (would need to be evaluated).

<br>

### sub_14000f2d4 (Obfuscated)
- The function is only made up by a single basic block. - **FALSE**
  - Ideed, this function is more complex than consisting of only one basic block.
  - Complex functions do not necessarily need to be obfuscated, but by looking in conjunction with the code (next point) we can clearly see the sheer amount of jumps/gotos. 
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/b23393b4-00c9-4dc8-805e-edfc2527f298" width="450">
<br>

- The function contains rare or unusual instructions in the disassembly / pseudo C-code. - **FALSE** 
- The function's code does not represent "normal / usual" code. - **TRUE**
  - High amount of jumps/goto labels
  - Possible arithmetic encoding from `0x14000f3ed` to `0x14000f3f6`.
```C
[...]
14000f2ee          if (arg3 == 0)
14000f2eb          {
14000f2ee              goto label_14000f33c;
14000f2ee          }
14000f2f0          uint32_t r11_1 = ((uint32_t)*(uint8_t*)arg3);
14000f2f7          if (r11_1 == 0)
14000f2f4          {
14000f2f7              goto label_14000f33c;
14000f2f7          }
14000f2f9          rax = ((uint64_t)*(uint32_t*)(arg3 + 0x14));
14000f304          int64_t r9_1;
14000f304          if ((TEST_BITD(rax, 9)))
14000f300          {
14000f308              if ((rax & 4) != 0)
14000f306              {
14000f30a                  rdx = 1;
14000f32d              label_14000f32d:
14000f32d                  r9_1 = 2;
14000f330                  r10 = 2;
14000f330              }
14000f311              else
14000f311              {
14000f311                  if ((rax & 8) != 0)
14000f30f                  {
14000f313                      rdx = 2;
14000f315                      goto label_14000f32d;
14000f315                  }
14000f319                  if ((rax & 0x10) != 0)
14000f317                  {
14000f31b                      rdx = 4;
14000f320                      goto label_14000f32d;
14000f320                  }
14000f322                  r9_1 = 0;
14000f327                  if ((rax & 0x20) != 0)
14000f325                  {
14000f329                      rdx = 8;
14000f329                      goto label_14000f32d;
14000f329                  }
14000f329              }
14000f335              if (rdx == 0)
14000f333              {
14000f335                  goto label_14000f33c;
14000f335              }
14000f33a              if (r10 == 0)
14000f337              {
14000f33a                  goto label_14000f33c;
14000f33a              }
14000f33a              goto label_14000f375;
14000f33a          }
14000f34b          if ((rax & 0x40) == 0)
14000f349          {
14000f359              if (rax < 0)
14000f357              {
14000f35b                  rdx = 2;
14000f35b              }
14000f363              else
14000f363              {
14000f363                  if ((!(TEST_BITD(rax, 8))))
14000f35f                  {
14000f363                      goto label_14000f33c;
14000f363                  }
14000f365                  rdx = 4;
14000f365              }
14000f36a              r9_1 = 1;
14000f36a              goto label_14000f375;
14000f36a          }
14000f34d          rdx = 1;
14000f352          r9_1 = 1;
14000f375      label_14000f375:
14000f37b          int16_t* i;
14000f37b          for (i = (((uint64_t)(r11_1 - rdx)) + arg1); i > arg1; i = ((char*)i - 1))
14000f3f8          {
14000f381              if (r9_1 == 1)
14000f37d              {
14000f386                  if (rdx == r9_1)
14000f383                  {
14000f388                      rax = arg3[0x10];
14000f38e                      if (*(uint8_t*)i == rax)
14000f38c                      {
14000f38e                          break;
14000f38e                      }
14000f38e                  }
14000f39c                  if ((rdx == 2 && *(uint16_t*)i == *(uint16_t*)(arg3 + 0x10)))
14000f394                  {
14000f39c                      break;
14000f39c                  }
14000f3a1                  if (rdx == 4)
14000f39e                  {
14000f3a3                      rax = ((uint64_t)*(uint32_t*)(arg3 + 0x10));
14000f3a3                      goto label_14000f3a7;
14000f3a3                  }
14000f3a3              }
14000f3ae              else if (r9_1 == 2)
14000f3ab              {
14000f3d4                  if ((rdx == 2 && *(uint16_t*)i == *(uint16_t*)(arg3 + 8)))
14000f3cc                  {
14000f3d4                      break;
14000f3d4                  }
14000f3b3                  if (rdx == 1)
14000f3b0                  {
14000f3b5                      rax = arg3[8];
14000f3bb                      if (*(uint8_t*)i == rax)
14000f3b9                      {
14000f3bb                          break;
14000f3bb                      }
14000f3bb                  }
14000f3ed                  bool cond:0_1;
14000f3ed                  if ((((rdx == 2 && *(uint16_t*)i != *(uint16_t*)(arg3 + 8)) || (((rdx != 1 && rdx != 2) || rdx == 1) && rdx != 4)) && rdx == 8))
14000f3ea                  {
14000f3f3                      cond:0_1 = *(uint64_t*)i == *(uint64_t*)(arg3 + 8);
14000f3ef                  }
14000f3c0                  if ((((rdx != 1 && rdx != 2) || rdx == 1) && rdx == 4))
14000f3bd                  {
14000f3c2                      rax = ((uint64_t)*(uint32_t*)(arg3 + 8));
14000f3a7                  label_14000f3a7:
14000f3a7                      cond:0_1 = *(uint32_t*)i == rax;
14000f3a7                  }
14000f3f6                  if ((((((rdx == 2 && *(uint16_t*)i != *(uint16_t*)(arg3 + 8)) || (((rdx != 1 && rdx != 2) || rdx == 1) && rdx != 4)) && rdx == 8) || (((rdx != 1 && rdx != 2) || rdx == 1) && rdx == 4)) && cond:0_1))
14000f3f6                  {
14000f3f6                      break;
14000f3f6                  }
14000f3f6              }
14000f3ab          }
14000f37b          if (i <= arg1)
14000f378          {
14000f3de              goto label_14000f33c;
14000f3de          }
14000f3de          sub_1400586c0(i, 0xaa, ((uint64_t)rdx));
14000f3e3          rax = 1;
14000f3e3      }
14000f348      return rax;
14000f348  }
```
<br>

- Other findings: None
<br>

### XXX (Obfuscated)
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly / pseudo C-code.
- The function's code does not represent "normal / usual" code.
- Other findings:
<br>

### sub_1400098b0 (Not Obfuscated)
- The function is only made up by a single basic block. - **FALSE**

<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/62fd055d-a1af-4574-a1fd-83988e906929" width="300">
<br>

- The function contains rare or unusual instructions in the disassembly / pseudo C-code. - **FALSE**
- The function's code does not represent "normal / usual" code. - **FALSE**

<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/31762477-9710-4b89-ba28-8a808daa7f7f" width="650">
<br>

- Other findings: None
<br>

### XXX (Not Obfuscated)
- The function is only made up by a single basic block.
- The function contains rare or unusual instructions in the disassembly / pseudo C-code.
- The function's code does not represent "normal / usual" code.
- Other findings:
<br>

@tim: to check if asumptions are true, really obfuscated or not?




## Analysis Procedure <a name="analysisprocedure"></a>
