## Curve Flattening
References:
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Curve Flattening on Tigress: https://tigress.wtf/flatten.html

The following command transformers the given fib.c file into an obfuscated fib_out.c file:
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=Flatten --Functions=fib --out=fib_out.c fib.c`



## Troubleshooting
The following error occurs in my case when trying to run the command above:
```
fabio@fabio-VirtualBox:~/Documents/tigress_workspace$ tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=Flatten --Functions=fib --out=fib_out.c fib.c

fib_out.c:606:54: error: ‘fclose’ undeclared here (not in a function)

  606 | extern FILE *tmpfile(void)  __attribute__((__malloc__(fclose,1), __malloc__)) ;

      |                                                      ^~~~~~

fib_out.c:2518:12: error: ‘reallocarray’ undeclared here (not in a function)

 2518 | __malloc__(reallocarray,1), __alloc_size__(2,3))) ;
```
It does seem that the header files from <stdio.h> and <stdlib.h> are not correctly included, but that is just my speculation. A web search won't give any further informations about it. // need to recheck
