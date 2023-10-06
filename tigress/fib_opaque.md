## Opaque Predicates
### General Definition

<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e2deada6-8510-4e74-9c36-7e3dc723cee0" width="500"/>
<br/>

An opaque predicate is a part of a program, which might be missed during analysis. This is given if the program analysis is not sophisticated enough.
It is referred to as a branch that always executes in one direction, which is known to the creator of the program, and which is unknown a priori to the analyzer.

Furthermore it might be defined as an expression whose value is known to the maker/the person implementing it, but which is difficult for an attacker to find out.

Opaque Predicates can have the following types:
- P^T = opaquely true predicate
- P^F = opaquely false predicate
- P^? = opaquely intermediate predicate
- E^=v = opaque expression of value v

Examples:<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/623a0dfc-1caa-4d9b-b9c7-8ccb79e9a008" width="500"/>
<br/>


### Tigress
Tigress is a C-programm obfuscator. It takes a file written in C, obfuscates it by imposing the options added by the user and outputs the obfuscated C-file.

The following command transforms the given fib.c file into an obfuscated fib_out.c file:<br/>
`tigress --Environment=x86_64:Linux:Gcc:13.2.1 --Transform=AddOpaque --Functions=fib,main --out=fib_opaque.c /home/training/Desktop/tigress/3.1/fib.c`

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
_fib.o:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5f5d2c9d-1ad1-4667-86a5-d07965ce8cf2" width="500"/>

_fib_opaque.o:_<br/>

<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e5ca362f-b24c-4542-9a33-f2255922667c" width="400"/>

Appart from an increased number of instructions, the control flow graphs seem to be similar. 


#### Decompiler
The decompiler presents the following code for the standard and obfuscated implementation of the main function:<br/>
_fib.o:_
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

_fib_opaque.o:_
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


---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- What is an "opaque predicate"? - https://reverseengineering.stackexchange.com/questions/1669/what-is-an-opaque-predicate
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Encode Arithmetic on Tigress: https://tigress.wtf/encodeArithmetic.html
