## Curve Flattening
References:
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Curve Flattening on Tigress: https://tigress.wtf/flatten.html

The following command transformers the given fib.c file into an obfuscated fib_out.c file:
`tigress --Environment=x86_64:Linux:Gcc:13.2.1 --Transform=Flatten --Functions=fib,main --out=fib_flatten.c /home/training/Desktop/tigress/3.1/fib.c`


### Analysis in Ghidra
![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/f2de9d38-1e3d-49f5-8a89-5a2ce7cbeb7d)

