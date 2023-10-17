## Opaque Predicates
Important: Due to an error in tigress, no samples could be generated with the "AddOpaque" option.
Hence another sample has been taken for analysis (referenced bellow).
### General Definition

<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e2deada6-8510-4e74-9c36-7e3dc723cee0" width="500"/>
<br/>
Defined as expression that evaluates to either "true" or "false", for which the outcome is known by the programmer beforehand, but which, for a variety of reasons, still needs to be evaluated at run time. Opaque predicates are basically branches of which the outcome is already clear. There might 
An opaque predicate is a part of a program, which might be missed during analysis. This is given if the program analysis is not sophisticated enough.




Opaque Predicates can have the following types:
- P^T = opaquely true predicate
- P^F = opaquely false predicate
- P^? = opaquely intermediate predicate
- E^=v = opaque expression of value v

Examples:<br/>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/623a0dfc-1caa-4d9b-b9c7-8ccb79e9a008" width="500"/>
<br/>

### Analysis in Ghidra

adress 0x491aa0

---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- What is an "opaque predicate"? - https://reverseengineering.stackexchange.com/questions/1669/what-is-an-opaque-predicate
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Sample - https://github.com/mrphrazer/r2con2020_deobfuscation/blob/master/samples/ac3e087e43be67bdc674747c665b46c2
- Opaque Predicate - https://en.wikipedia.org/wiki/Opaque_predicate
