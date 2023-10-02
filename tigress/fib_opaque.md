## Opaque Predicates
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5c05eea2-9e56-4aec-8ae0-12795bd4b669" width="500"/>

References:
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Opaque Predicate on Tigress: [https://tigress.wtf/flatten.html](https://tigress.wtf/addOpaque.html)

The following command transforms the given fib.c file into an obfuscated fib_out.c file:<br/>
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=AddOpaque --Functions=main,fib --out=fib_opaque.c /home/training/Desktop/tigress/3.1/fib.c`

### Analysis in Ghidra
#### Graph View
_fib.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/5f5d2c9d-1ad1-4667-86a5-d07965ce8cf2" width="500"/>

_fib_flatten.c:_<br/>
<img src="" width="800"/>

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

_fib_opaque.c:_
```C

/* WARNING: Control flow encountered bad instruction data */
/* WARNING: Instruction at (ram,0x0010016f) overlaps instruction at (ram,0x0010016e)
    */

void UndefinedFunction_00100148(int param_1,undefined8 param_2,long param_3,char *param_4)

{
  byte *pbVar1;
  char *pcVar2;
  byte bVar3;
  byte bVar4;
  int in_EAX;
  undefined3 uVar6;
  undefined4 in_register_00000004;
  int unaff_EBX;
  undefined4 unaff_0000001c;
  undefined4 uVar5;
  
  *(int *)CONCAT44(in_register_00000004,in_EAX) =
       *(int *)CONCAT44(in_register_00000004,in_EAX) + in_EAX;
  bVar4 = (byte)in_EAX;
  *(byte *)CONCAT44(in_register_00000004,in_EAX) =
       *(char *)CONCAT44(in_register_00000004,in_EAX) + bVar4;
  *(byte *)CONCAT44(in_register_00000004,in_EAX) =
       *(char *)CONCAT44(in_register_00000004,in_EAX) + bVar4;
  bVar3 = *(byte *)CONCAT44(in_register_00000004,in_EAX);
  *(byte *)CONCAT44(in_register_00000004,in_EAX) =
       *(char *)CONCAT44(in_register_00000004,in_EAX) + bVar4;
  uVar6 = (undefined3)((uint)in_EAX >> 8);
  bVar4 = bVar4 + CARRY1(bVar3,bVar4);
  uVar5 = CONCAT31(uVar6,bVar4);
  *(byte *)CONCAT44(in_register_00000004,uVar5) =
       *(char *)CONCAT44(in_register_00000004,uVar5) + bVar4;
  *(byte *)CONCAT44(in_register_00000004,uVar5) =
       *(char *)CONCAT44(in_register_00000004,uVar5) + bVar4;
  *(byte *)CONCAT44(in_register_00000004,uVar5) =
       *(char *)CONCAT44(in_register_00000004,uVar5) + bVar4;
  *(int *)(param_3 + 0x52) = *(int *)(param_3 + 0x52) + param_1;
  *param_4 = *param_4 + bVar4;
  if (-1 < *param_4) {
    *(int *)CONCAT44(unaff_0000001c,unaff_EBX) =
         *(int *)CONCAT44(unaff_0000001c,unaff_EBX) + unaff_EBX;
    uVar5 = CONCAT31(uVar6,bVar4);
    pbVar1 = (byte *)((CONCAT44(in_register_00000004,uVar5) | 7) + 0x1c000001);
    *pbVar1 = *pbVar1 | (byte)param_3;
    *(byte *)(CONCAT44(in_register_00000004,uVar5) | 7) =
         *(char *)(CONCAT44(in_register_00000004,uVar5) | 7) + (bVar4 | 7);
    pcVar2 = (char *)((CONCAT44(in_register_00000004,uVar5) | 7) +
                     (CONCAT44(in_register_00000004,uVar5) | 7));
    *pcVar2 = *pcVar2 + (char)unaff_EBX;
    *(byte *)(CONCAT44(in_register_00000004,uVar5) | 7) =
         *(char *)(CONCAT44(in_register_00000004,uVar5) | 7) + (bVar4 | 7);
                    /* WARNING: Bad instruction - Truncating control flow here */
    halt_baddata();
  }
  pcVar2 = (char *)(CONCAT44(in_register_00000004,uVar5) + 0x7fffffe);
  *pcVar2 = *pcVar2 + (byte)param_3;
  *(byte *)CONCAT44(in_register_00000004,uVar5) =
       *(char *)CONCAT44(in_register_00000004,uVar5) + bVar4;
  *(byte *)CONCAT44(in_register_00000004,uVar5) =
       *(char *)CONCAT44(in_register_00000004,uVar5) + bVar4;
                    /* WARNING: Bad instruction - Truncating control flow here */
  halt_baddata();
}

```
