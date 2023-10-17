## Opaque Predicates
Note: Due to an unknown error in tigress, no samples could be generated with the "AddOpaque" option.
Hence another sample has been taken for analysis (referenced bellow).

### General Definition
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e2deada6-8510-4e74-9c36-7e3dc723cee0" width="500"/>
<br/>

In computer science, predicates are conditional expressions that evaluate to true or false. An opaque predicates value (outcome) is kown to the obfuscator at the time of obfuscation, but difficult to find out externally at a later point in time. 
Using opaque predicates will lead to an extremely excessive control flow graph with redudant inexecutable paths.
It is for this reason, that any further analysis based on the control flow graph will turn into tedious work.
In comparasion to other control flow graph obfuscation techniques, opaque predicates act more covert, since it is difficult to separate them from actual functional code in the program.

#### Opaque Expressions
Opaque predicates can be classified as the following types:

_P<sup>T</sup> - opaquely true predicate_ <br>
The condition is always satisfied, hence "true". The following example shows, that either expression `2` or `(x^2 + x)^T` must be correct. Since `2` is always true, the branch on the left will always be taken.
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
This code snipplet presents the expression `x mod 2 == 0`, which depending on the value of `x`, can be either true or false.
For example for `x` being equal to 2, the if-statement would be executed, since this would fullfil the condition (2 mod 2 == 0).
On the other hand `x = 4` would not satisfy this condition and the code in the else-block would be executed instead. 


_E<sup>=v</sup> - for an opaque expression of value v_ <br>

@ fabio: zu ergänzen


### Analysis in Ghidra
Beginning at the address 0x491aa0, one can recognize the high complexity of the control flow graph of this binarys function.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e507b626-41a5-43dd-8d44-d87edf457f1a" width="700">
<br/>
Since the amount is so vast, only a sample set of randomly chosen candidates will be analyzed in further detail.

#### Sample at 0x491b38
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/ec0d8dde-7edc-4b11-af79-f4d8d884cede" width="900">
<br/>

if-statement at 0x491b38
```
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

  
if-block
```
                             LAB_00491b98                                    XREF[1]:     00491b5b(j)  
        00491b98 c7 05 40        MOV        dword ptr [DAT_0058dd40],0x0                     = ??
                 dd 58 00 
                 00 00 00 00
        00491ba2 31 ff           XOR        EDI,EDI
        00491ba4 eb b7           JMP        LAB_00491b5d

```


```
LAB_00491b98:
    DAT_0058dd40 = 0;
  }
```




else block / ending block
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

```
  while( true ) {
    puStack_6c = (uint *)0x491b67;
    ppuStack_34 = &puStack_70;
    puStack_74 = (uint *)0x491b7b;
    ppuVar14 = (uint **)&uStack_78;
    if ((int)(2 / (ulonglong)uVar9) != DAT_0058dd40 * DAT_0058dd40 + 3) break;
```







#### Sample at XXX
<br>
<img src="" width="900">
<br/>



(may only 2 samples?)
#### Sample at XXX
<br>
<img src="" width="900">
<br/>




















---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- What is an "opaque predicate"? - https://reverseengineering.stackexchange.com/questions/1669/what-is-an-opaque-predicate
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Sample - https://github.com/mrphrazer/r2con2020_deobfuscation/blob/master/samples/ac3e087e43be67bdc674747c665b46c2
- Opaque Predicate - https://en.wikipedia.org/wiki/Opaque_predicate
