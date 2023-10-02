## Encode Arithmetic
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/8efea02c-87eb-4df0-a2bd-2f0f06c919c4" width="500"/>

References:
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Curve Flattening on Tigress: https://tigress.wtf/flatten.html

The following command transforms the given fib.c file into an obfuscated fib_out.c file:<br/>
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=EncodeArithmetic --Functions=main,fib --out=fib_encode.c /home/training/Desktop/tigress/3.1/fib.c`

### Analysis in Ghidra
#### Graph View
_fib.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5f5d2c9d-1ad1-4667-86a5-d07965ce8cf2" width="500"/>

_fib_encode.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/8dbc0a6b-3b09-49bf-91ad-fa3d3dafe630" width="300"/>

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
