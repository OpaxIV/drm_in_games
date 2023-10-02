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

```

_fib_flatten.c:_
```C
```
