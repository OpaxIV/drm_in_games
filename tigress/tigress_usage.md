## Usage

When running these commands, for the love of god make sure to not touch the folder in which tigress is installed.
Just use the REVE2 VM and the folder on the desktop.


### Flatten
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=Flatten --Functions=main,fib --out=fib_flatten.c /home/training/Desktop/tigress/3.1/fib.c`



### Encode Arithmetic
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=EncodeArithmetic --Functions=main,fib --out=fib_encode.c /home/training/Desktop/tigress/3.1/fib.c`



### Opaque Predicates
`tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=AddOpaque --Functions=main,fib --out=fib_opaque.c /home/training/Desktop/tigress/3.1/fib.c`



