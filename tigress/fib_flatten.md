## Curve Flattening
### General Definition
The idea of curve flattening consists in altering the control flow of a program by using a dispatcher function.
As seen in the following picture, the graph is literally "flattened":<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/6a6a8777-acfd-4efb-8460-840951b5638d" width="500"/>
<br/>
The edges of the basic blocks are all redirected to a dispatcher function. Decisions, to which blocks of the program one should jump to, are then based on a new artifical variable.




### Tigress
The following command transforms the given fib.c file into an obfuscated fib_out.c file:<br/>
`tigress --Environment=x86_64:Linux:Gcc:13.2.1 --Transform=Flatten --Functions=fib,main --out=fib_flatten.c /home/training/Desktop/tigress/3.1/fib.c`


### Analysis in Ghidra
#### Graph View
_fib.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5f5d2c9d-1ad1-4667-86a5-d07965ce8cf2" width="500"/>

_fib_flatten.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/9dd4d1a4-e320-4599-bef3-00138a762091" width="800"/>

#### Decompiler
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
