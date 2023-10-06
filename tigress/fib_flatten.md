## Curve Flattening
### General Definition
The idea of curve flattening consists in altering the control flow of a program by using a dispatcher function.
As seen in the following picture, the graph is literally "flattened":<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/6a6a8777-acfd-4efb-8460-840951b5638d" width="500"/>
<br/>
The edges of the basic blocks are all redirected to a dispatcher function. Decisions, to which blocks of the program one should jump to, are then based on a new artifical variable.

### Tigress
Tigress is a C-programm obfuscator. It takes a file written in C, obfuscates it by imposing the options added by the user and outputs the obfuscated C-file.

The following command transforms the given fib.c file into an obfuscated fib_out.c file:<br/>
`tigress --Environment=x86_64:Linux:Gcc:13.2.1 --Transform=Flatten --Functions=fib,main --out=fib_flatten.c /home/training/Desktop/tigress/3.1/fib.c`

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
Following pictures show the control flow graph of the standard fib.c implementation and the obfuscated, flattend control flow graph of the fib_flatten.c file.<br/>
_fib.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5f5d2c9d-1ad1-4667-86a5-d07965ce8cf2" width="500"/>

_fib_flatten.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/9dd4d1a4-e320-4599-bef3-00138a762091" width="800"/>

When looking into the second graph one can see, that a switch case was implemented, which wasn't existent before.<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/d798dc0e-bdc9-4e32-afcc-216719758f99" width="1200"/>

Since being a switch case, allmost all branches are outgoing from exactly the basic block at `00100145`  (as seen by the arrows).

#### Decompiler
The decompiler presents the following code for the standard implementation `fib.c`:<br/>
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
Looking at the decompiled output shows a similar result: The newly created fib_flatten.c contains a switch case and an artificial variable `local_10`. The variable is then used to execute the conditional jumps. 

_fib_flatten.c:_
```C
undefined8 main(int param_1,long param_2,undefined8 param_3)

{
  int iVar1;
  uint uVar2;
  undefined8 local_10;
  
  megaInit();
  local_10 = 1;
  _global_argv = param_2;
  _global_argc = param_1;
  _global_envp = param_3;
  do {
    switch(local_10) {
    case 0:
      iVar1 = atoi(*(char **)(param_2 + 8));
      uVar2 = fib(iVar1);
      printf("result: %u\n",(ulong)uVar2);
      local_10 = 3;
      break;
    case 1:
      local_10 = 2;
      break;
    case 2:
      if (param_1 < 2) {
        local_10 = 4;
      }
      else {
        local_10 = 0;
      }
      break;
    case 3:
      return 0;
    case 4:
      return 0xffffffff;
    }
  } while( true );
}
```


---
References:
- What is a "control-flow flattening" obfuscation technique? - https://reverseengineering.stackexchange.com/questions/2221/what-is-a-control-flow-flattening-obfuscation-technique
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Curve Flattening on Tigress: https://tigress.wtf/flatten.html
