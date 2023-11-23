# Opaque Predicates
Note: Due to an unknown error in tigress, no samples could be generated with the "AddOpaque" option.
Hence another sample has been taken for analysis (referenced bellow).
<br>

## Table of Contents
1. [General Definition](#generaldefinition)
2. [Analysis in Ghidra](#analysisinghidra)
<br>

## General Definition <a name="generaldefinition"></a>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e2deada6-8510-4e74-9c36-7e3dc723cee0" width="500"/>
<br/>

In computer science, predicates are conditional expressions that evaluate to true or false. An opaque predicates value (the outcome) is known to the obfuscator at the time of obfuscation, but difficult to find out externally at a later point in time. 
Using opaque predicates will lead to an extremely excessive control flow graph with redudant inexecutable paths.
It is for this reason, that any further analysis based on the control flow graph will turn into tedious work.
In comparasion to other control flow graph obfuscation techniques, opaque predicates act more covert, since it is difficult to separate them from actual functional code in the program.

### Opaque Expressions
Opaque predicates can be classified as the following types:

_P<sup>T</sup> - opaquely true predicate_ <br>
The condition is always satisfied, hence "true". The following example shows, that either expression `2` or `(x^2 + x)^T` must be correct. Since `2` is always true out of these two conditions, the if-case will always be executed.
```C
if (2|(x^2 + x)^T){
...
}
else {
...
}
```


_P<sup>F</sup> - opaquely false predicate_ <br>
The condition is never satisfied, hence "false".
Consider the following example:
```C
if (2 == 3){
...
}
else {
...
}
```
The expression `2 == 3` will never be true, hence the else-statement is always the executed branch.


_P<sup>?</sup> - opaquely intermediate predicate_ <br>
This type of opaque predicate combines the two types above. It is uncertain, which path gets taken since the expression can be true in one case and be false in another case.
```C
if (x mod 2 == 0){
...
}
else {
...
}
```
This code snipplet presents the expression `x mod 2 == 0`, which depending on the value of `x` can be either true or false.
For example for `x` being equal to 2 the if-statement would be executed, since this would fulfill the condition (2 mod 2 == 0).
On the other hand `x = 4` would not satisfy this condition and the code in the else-block would be executed instead. 


_E<sup>=v</sup> - for an opaque expression of value v_ <br>
Available informations about this type of opaque predicate are vague. It can be seen as a generalisation of the preceding types. This expression defines an opaque predicate `E` as being evaluated to the value of `v`. The value `v` can either be true or false.


## Analysis in Ghidra <a name="analysisinghidra"></a>
Beginning at the address 0x491aa0, one can recognize the high complexity contained in this binary.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e507b626-41a5-43dd-8d44-d87edf457f1a" width="700">
<br/>
Since the amount is so vast, only a sample set of randomly chosen candidates will be analyzed in further detail.


### Sample at 0x491b38
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/ec0d8dde-7edc-4b11-af79-f4d8d884cede" width="900">
<br/>

Three basic blocks compose this opaque predicate.
The first can be seen as the "if-statement", in which the condition gets checked.
Basic block containing a conditional check (starting at `0x491b38`):
```asm
                             LAB_00491b38                                    XREF[1]:     00491b1e(j)  
        00491b38 8b 0d 44        MOV        this,dword ptr [DAT_0058dd44]                    = ??
                 dd 58 00
        00491b3e 8b 3d 40        MOV        EDI,dword ptr [DAT_0058dd40]                     = ??
                 dd 58 00
        00491b44 0f af c9        IMUL       this,this
        00491b47 41              INC        this
        00491b48 b8 02 00        MOV        EAX,0x2
                 00 00
        00491b4d 31 d2           XOR        EDX,EDX
        00491b4f f7 f1           DIV        this
        00491b51 89 fa           MOV        EDX,EDI
        00491b53 0f af d2        IMUL       EDX,EDX
        00491b56 83 c2 03        ADD        EDX,0x3
        00491b59 39 d0           CMP        EAX,EDX
        00491b5b 74 3b           JZ         LAB_00491b98
```
The jump occurs if `eax` and `edx` are equal. This is computed by subtracting both values currently resting in the registers and checking their result.
If the result ends up being equal to zero, then the jump occurs. Every other outcome would lead to a continuation of the code at `0x0491b5d`.

Interesting to know would be, what the actual if-statement would be like, hence to understand what the values in the registers `eax` and `edx` are.
We can somewhat get an idea of this by looking at the decompiler output:
```C
  uVar9 = DAT_0058dd44 * DAT_0058dd44 + 1;
  if ((int)(2 / (ulonglong)uVar9) == DAT_0058dd40 * DAT_0058dd40 + 3) goto LAB_00491b98;
  while( true ) {
    puStack_6c = (uint *)0x491b67;
    ppuStack_34 = &puStack_70;
    puStack_74 = (uint *)0x491b7b;
    ppuVar14 = (uint **)&uStack_78;
    if ((int)(2 / (ulonglong)uVar9) != DAT_0058dd40 * DAT_0058dd40 + 3) break;
LAB_00491b98:
    DAT_0058dd40 = 0;
  }
```
Most notably is the line `if ((int)(2 / (ulonglong)uVar9) == DAT_0058dd40 * DAT_0058dd40 + 3) goto LAB_00491b98;`.
To get a better understanding, we need to simplify this expression. By looking one line above, we can see for what the `uVar9` variable stands for.
By substituting `uVar9` with `DAT_0058dd44 * DAT_0058dd44 + 1` we get:
```C
  if ((int)(2 / (ulonglong)DAT_0058dd44 * DAT_0058dd44 + 1) == DAT_0058dd40 * DAT_0058dd40 + 3) goto LAB_00491b98;
```

Rewritten with some examplary variables for better visualisation:
```
x = DAT_0058dd44
y = DAT_0058dd40

if ((2 /  (x * x + 1)) ==  y * y + 3) goto LAB_00491b98;
```

Now we can check this expression using the python tool "z3-solver":
_Note: The number `32` in `BitVecs(...)` stands for the values' size. Since we are working with 32-bit registers, the number to input here is 32._
```py
[training@vm ~]$ python
Python 3.11.5 (main, Sep  2 2023, 14:16:33) [GCC 13.2.1 20230801] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from z3 import *
>>> x, y = BitVecs('x y', 32)
>>> f = (2 /  (x * x + 1))
>>> g = y * y + 3
>>> f
2/(x*x + 1)
>>> g
y*y + 3
>>> solver = Solver()
>>> solver.add(f == g)
>>> solver.check()
unsat
>>>
```

As we can see, this expression is not satisfiable. So any occurence of `((2 /  (x * x + 1)) ==  y * y + 3)` will lead it to never be true.
This branch can be described as dead weight, since it will never be executed.

We can conclude, that the conditon is never satisfied and hence the control flow will directly continue onto the adress `0x491b5d`.
```
                             LAB_00491b5d                                    XREF[1]:     00491ba4(j)  
        00491b5d b8 04 00        MOV        EAX,0x4
                 00 00
        00491b62 e8 19 81        CALL       __alloca_probe                                   undefined __alloca_probe(void)
                 00 00
        00491b67 89 e0           MOV        EAX,ESP
        00491b69 83 e0 f8        AND        EAX,0xfffffff8
        00491b6c 89 46 34        MOV        dword ptr [ESI + 0x34],EAX
        00491b6f 89 c4           MOV        ESP,EAX
        00491b71 b8 04 00        MOV        EAX,0x4
                 00 00
        00491b76 e8 05 81        CALL       __alloca_probe                                   undefined __alloca_probe(void)
                 00 00
        00491b7b 89 e0           MOV        EAX,ESP
        00491b7d 83 e0 f8        AND        EAX,0xfffffff8
        00491b80 89 46 38        MOV        dword ptr [ESI + 0x38],EAX
        00491b83 89 c4           MOV        ESP,EAX
        00491b85 b8 02 00        MOV        EAX,0x2
                 00 00
        00491b8a 31 d2           XOR        EDX,EDX
        00491b8c f7 f1           DIV        this
        00491b8e 0f af ff        IMUL       EDI,EDI
        00491b91 83 c7 03        ADD        EDI,0x3
        00491b94 39 f8           CMP        EAX,EDI
        00491b96 75 0e           JNZ        LAB_00491ba6
```

### Sample at 0x492d09
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/b8f8b709-5d42-4252-969b-8f82047110ee" width="600">
<br/>

As a second example the opaque predicate at the address `0x492d09` has been chosen. The following code shows a familiarity with the preceding example
```asm
00492d09 a1 bc dc        MOV        EAX,[DAT_0058dcbc]                               = ??
		 58 00
00492d0e 8b 0d b8        MOV        this,dword ptr [DAT_0058dcb8]                    = ??
		 dc 58 00
00492d14 89 4e 2c        MOV        dword ptr [ESI + 0x2c],this
00492d17 0f af c0        IMUL       EAX,EAX
00492d1a 89 46 3c        MOV        dword ptr [ESI + 0x3c],EAX
00492d1d 6b c0 07        IMUL       EAX,EAX,0x7
00492d20 48              DEC        EAX
00492d21 0f af c9        IMUL       this,this
00492d24 39 c8           CMP        EAX,this
00492d26 a1 94 08        MOV        EAX,[DAT_00590894]                               = ??
		 59 00
00492d2b 89 46 24        MOV        dword ptr [ESI + 0x24],EAX
00492d2e 0f 84 94        JZ         LAB_00492dc8
		 00 00 00

```
The conditional jump occurs only if the value at the dereferenced stack pointer's position `[ESI + 0x24]` is equal to the current value in `eax`.  

```C
[...]
    if ((uint *)**ppuStack_38 == *ppuStack_1c) {
      ppuStack_3c = DAT_0058dcb8;
      ppuStack_2c = (uint **)(DAT_0058dcbc * DAT_0058dcbc);
      ppuStack_44 = DAT_00590894;
      if ((int)ppuStack_2c * 7 + -1 == (int)DAT_0058dcb8 * (int)DAT_0058dcb8) goto LAB_00492dc8;
      while( true ) {
        ppuStack_48 = (uint **)*ppuVar6;
        ppuStack_40 = (uint **)(DAT_00590890 * DAT_00590890);
        if ((uint **)((int)ppuStack_44 * (int)ppuStack_44 * 7 - 1U) == ppuStack_40)
        goto LAB_00492d75;
        while ((int)(2 / (ulonglong)((int)ppuStack_44 * (int)ppuStack_44 + 1)) ==
               (int)ppuStack_40 + 3U) {
[...]
```
The conditional jump can be seen in the decompiler output at the following line `if ((int)ppuStack_2c * 7 + -1 == (int)DAT_0058dcb8 * (int)DAT_0058dcb8) goto LAB_00492dc8;`. By looking in the upper sector of the output we can see the "actual" value of `ppuStack_2c`, which is `(uint **)(DAT_0058dcbc * DAT_0058dcbc)`.
<br>
By replacing this expression we get:
`if ((uint **)(DAT_0058dcbc * DAT_0058dcbc) * 7 + -1 == (int)DAT_0058dcb8 * (int)DAT_0058dcb8) goto LAB_00492dc8;`
If we further simplify this statement like in the example before, we get:
```C
x =  DAT_0058dcbc
y = DAT_0058dcb8

if ((x * x) * 7 + -1 == y * y) goto LAB_00492dc8;
```

Now we can check this expression using the python tool "z3-solver":
<br>
_Note: The number -1 in the expression `((x * x * 7) + -1` gets converted into its 32-bit counterpart. Rewriting the expression to  `((x * x * 7) - 1)` would lead to the same result._

```py
[training@vm ~]$ python
Python 3.11.5 (main, Sep  2 2023, 14:16:33) [GCC 13.2.1 20230801] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from z3 import *
>>> x, y = BitVecs('x y', 32)
>>> f = ((x * x * 7) + -1)
>>> g = y * y
>>> f
x*x*7 + 4294967295
>>> g
y*y
>>> solver = Solver()
>>> solver.add(f == g)
>>> solver.check()
unsat
>>> 
```

As we can see, this expression is not satisfiable. So any occurence of `(x * x) * 7 + -1 == y * y` will lead it to never be true.
This branch can be described as dead weight, since it will never be executed.

We can conclude, that the conditon is never satisfied and hence the control flow will directly continue onto the adress 0x492d5b.

---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- What is an "opaque predicate"? - https://reverseengineering.stackexchange.com/questions/1669/what-is-an-opaque-predicate
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Sample - https://github.com/mrphrazer/r2con2020_deobfuscation/blob/master/samples/ac3e087e43be67bdc674747c665b46c2
- Opaque Predicate - https://en.wikipedia.org/wiki/Opaque_predicate
- Z3 Solver - https://github.com/Z3Prover/z3
- Semi-automatic Code Deobfuscation (r2con2020 workshop) - https://www.youtube.com/watch?v=_TsV0RXoIQE
