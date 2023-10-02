## Curve Flattening

References:
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Curve Flattening on Tigress: https://tigress.wtf/flatten.html

The following command transformers the given fib.c file into an obfuscated fib_out.c file:
`tigress --Environment=x86_64:Linux:Gcc:13.2.1 --Transform=Flatten --Functions=fib,main --out=fib_flatten.c /home/training/Desktop/tigress/3.1/fib.c`


### Analysis in Ghidra
#### Graph View
_fib.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/6a6a8777-acfd-4efb-8460-840951b5638d" width="500"/>

_fib_flatten.c:_<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/9dd4d1a4-e320-4599-bef3-00138a762091" width="800"/>
