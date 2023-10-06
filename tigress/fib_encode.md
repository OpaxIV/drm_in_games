## Encode Arithmetic
### General Definition

The basic idea of the encode arithmetic technique is to rewrite "simpler" functions into more complex expressions. 

<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/87ba2a98-23be-4e94-8603-a4026cdccecc" width="500"/>
<br/>

The following script shows this principle by comparing two functions:
```py
## checks if the two expressins are equal
## only for testing purposes


def genericExp(x: int) -> int:
    y = x * 42
    return y


def arithEnc(x: int) -> int:
    y = x << 5
    y += x << 3
    y += x << 1
    return y


if __name__ == "__main__":
    if genericExp(10) == arithEnc(10):
        print("True")
```
When executed the word "True" is printed into the console.

### Tigress
Tigress is a C-programm obfuscator. It takes a file written in C, obfuscates it by imposing the options added by the user and outputs the obfuscated C-file.

The following command transforms the given fib.c file into an obfuscated fib_out.c file:<br/>
`tigress --Environment=x86_64:Linux:Gcc:13.2.1 --Transform=EncodeArithmetic --Functions=fib,main --out=fib_encode.c /home/training/Desktop/tigress/3.1/fib.c`

Following chapters showcase an analysis of an obfuscated fib.c file. The input, non-obfuscated C-file contains a basic implementation of the Fibonacci series:
```C
#include "3.1/tigress.h"

#include <stdio.h>
#include <stdlib.h>


unsigned int fib(unsigned n) {
  unsigned a=0;
  unsigned b=1;
  unsigned int s = b;
  unsigned i;

  if (n == 0) {
    return 0;
  }


  for (i=2; i<=n; i++) {
    s=a+b;
    a=b;
    b=s;
  }
  return s;
}

int main (int argc, char** argv) {
  if (argc < 2) {
    return -1;
  }
  
  unsigned int n = atoi(argv[1]);
  
  printf("result: %u\n", fib(n));
  
  return 0;
}
```

### Analysis in Ghidra
#### Graph View
Following pictures show the control flow graph of the main function of the standard fib.c implementation and the obfuscated control flow graph of the fib_encode.c file.<br/>
_fib.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5f5d2c9d-1ad1-4667-86a5-d07965ce8cf2" width="500"/>

_fib_encode.c:_<br/>

<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/b94d8dbb-8d7e-4299-bf80-02bc1f8d333b" width="400"/>

The second graph clearly shows some change in its number of instrucions compared to the first graph.
By just looking at the beginning of the assembly code of the main function at `00100000` more instructions are to be seen compared to the same named standard implementation:

_fib.c:_
```
[...]
                           main                                            XREF[2]:     Entry Point(*), 00100130(*)  
        00100059 55              PUSH       RBP
        0010005a 48 89 e5        MOV        RBP,RSP
        0010005d 48 83 ec 20     SUB        RSP,0x20
        00100061 89 7d ec        MOV        dword ptr [RBP + local_1c],EDI
        00100064 48 89 75 e0     MOV        qword ptr [RBP + local_28],RSI
        00100068 83 7d ec 01     CMP        dword ptr [RBP + local_1c],0x1
        0010006c 7f 07           JG         LAB_00100075
        0010006e b8 ff ff        MOV        EAX,0xffffffff
                 ff ff
        00100073 eb 3b           JMP        LAB_001000b0
[...]
```



_fib_encode.c:_
```asm
[...]
                            main                                            XREF[4]:     Entry Point(*), 001001e8(*), 
                                                                                          _elfSectionHeaders::00000050(*), 
                                                                                          _elfSectionHeaders::000000d0(*)  
        00100000 55              PUSH       RBP
        00100001 48 89 e5        MOV        RBP,RSP
        00100004 48 83 ec 30     SUB        RSP,0x30
        00100008 89 7d ec        MOV        dword ptr [RBP + local_1c],EDI
        0010000b 48 89 75 e0     MOV        qword ptr [RBP + local_28],RSI
        0010000f 48 89 55 d8     MOV        qword ptr [RBP + local_30],RDX
        00100013 e8 cb 00        CALL       megaInit                                         undefined megaInit()
                 00 00
        00100018 8b 45 ec        MOV        EAX,dword ptr [RBP + local_1c]
        0010001b 89 05 57        MOV        dword ptr [_global_argc],EAX                     = ??
                 01 00 00
        00100021 48 8b 45 e0     MOV        RAX,qword ptr [RBP + local_28]
        00100025 48 89 05        MOV        qword ptr [_global_argv],RAX                     = ??
                 44 01 00 00
        0010002c 48 8b 45 d8     MOV        RAX,qword ptr [RBP + local_30]
        00100030 48 89 05        MOV        qword ptr [_global_envp],RAX                     = ??
                 49 01 00 00
        00100037 c7 45 f0        MOV        dword ptr [RBP + local_18],0x1
                 01 00 00 00
        0010003e 83 7d ec 02     CMP        dword ptr [RBP + local_1c],0x2
        00100042 0f 9e c0        SETLE      AL
        00100045 0f b6 d0        MOVZX      EDX,AL
        00100048 b8 02 00        MOV        EAX,0x2
                 00 00
        0010004d 2b 45 ec        SUB        EAX,dword ptr [RBP + local_1c]
        00100050 0f af c2        IMUL       EAX,EDX
        00100053 c1 f8 1f        SAR        EAX,0x1f
        00100056 89 c6           MOV        ESI,EAX
        00100058 83 7d ec 02     CMP        dword ptr [RBP + local_1c],0x2
        0010005c 0f 9e c0        SETLE      AL
        0010005f 0f b6 d0        MOVZX      EDX,AL
        00100062 b8 02 00        MOV        EAX,0x2
                 00 00
        00100067 2b 45 ec        SUB        EAX,dword ptr [RBP + local_1c]
        0010006a 89 d1           MOV        ECX,EDX
        0010006c 0f af c8        IMUL       ECX,EAX
        0010006f 83 7d ec 02     CMP        dword ptr [RBP + local_1c],0x2
        00100073 0f 9e c0        SETLE      AL
        00100076 0f b6 d0        MOVZX      EDX,AL
        00100079 b8 02 00        MOV        EAX,0x2
                 00 00
        0010007e 2b 45 ec        SUB        EAX,dword ptr [RBP + local_1c]
        00100081 0f af c2        IMUL       EAX,EDX
        00100084 c1 f8 1f        SAR        EAX,0x1f
        00100087 31 c1           XOR        ECX,EAX
        00100089 89 ca           MOV        EDX,ECX
        0010008b 89 f0           MOV        EAX,ESI
        0010008d 29 d0           SUB        EAX,EDX
        0010008f 85 c0           TEST       EAX,EAX
        00100091 79 07           JNS        LAB_0010009a
        00100093 b8 ff ff        MOV        EAX,0xffffffff
                 ff ff
        00100098 eb 47           JMP        LAB_001000e1
[...]
```

#### Decompiler
The decompiler presents the following code for the standard and obfuscated implementation of the main function:<br/>
_fib.c:_
```C
undefined8 main(int param_1,long param_2)

{
  int iVar1;
  uint uVar2;
  undefined8 uVar3;
  
  if (param_1 < 2) {
    uVar3 = 0xffffffff;
  }
  else {
    iVar1 = atoi(*(char **)(param_2 + 8));
    uVar2 = fib(iVar1);
    printf("result: %u\n",(ulong)uVar2);
    uVar3 = 0;
  }
  return uVar3;
}
```
<br/> 

_fib_encode.c:_
```C
undefined8 main(int param_1,long param_2,undefined8 param_3)

{
  int iVar1;
  uint uVar2;
  undefined8 uVar3;
  
  megaInit();
  _global_argv = param_2;
  _global_argc = param_1;
  _global_envp = param_3;
  if ((int)(((int)((2 - param_1) * (uint)(param_1 < 3)) >> 0x1f) -
           ((uint)(param_1 < 3) * (2 - param_1) ^ (int)((2 - param_1) * (uint)(param_1 < 3)) >> 0x1f
           )) < 0) {
    uVar3 = 0xffffffff;
  }
  else {
    iVar1 = atoi(*(char **)(param_2 + 8));
    uVar2 = fib(iVar1);
    printf("result: %u\n",(ulong)uVar2);
    uVar3 = 0;
  }
  return uVar3;
}
```
<br/>
Looking at the decompiled output, the newly created fib_encode.c contains the following "rewritten" expression:<br/>
```C
[...]
  if ((int)(((int)((2 - param_1) * (uint)(param_1 < 3)) >> 0x1f) -
           ((uint)(param_1 < 3) * (2 - param_1) ^ (int)((2 - param_1) * (uint)(param_1 < 3)) >> 0x1f
           )) < 0) {
    uVar3 = 0xffffffff;
  }
[...]
```


---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Encode Arithmetic on Tigress: https://tigress.wtf/encodeArithmetic.html
