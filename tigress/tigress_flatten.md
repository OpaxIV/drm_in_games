## Curve Flattening
References:
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Curve Flattening on Tigress: https://tigress.wtf/flatten.html

The following command transformers the given fib.c file into an obfuscated fib_out.c file:
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=Flatten --Functions=fib --out=fib_out.c fib.c`
