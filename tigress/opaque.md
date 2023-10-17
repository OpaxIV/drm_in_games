## Opaque Predicates
Note: Due to an unknown error in tigress, no samples could be generated with the "AddOpaque" option.
Hence another sample has been taken for analysis (referenced bellow).

### General Definition
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e2deada6-8510-4e74-9c36-7e3dc723cee0" width="500"/>
<br/>

In computer science, predicates are conditional expressions that evaluate to true or false. An opaque predicates value (outcome) is kown to the obfuscator at the time of obfuscation, but difficult to find out externally at a later point in time. 
Using opaque predicates will lead to an extremely excessive control flow graph with redudant inexecutable paths.
It is for this reason, that any further analysis based on the control flow graph will turn into tedious work.
In comparasion to other control flow graph obfuscation techniques, opaque predicates act more covert, since it is difficult to separate them from actual functional code in the program.

#### Opaque Expressions
Opaque predicates can be classified as the following types:
P<sup>T</sup> - opaquely true predicate
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/026f70ce-a611-448c-9f9f-0e6fb058f16c" width="200">
<br/>
P<sup>F</sup> - opaquely false predicate
P<sup>?</sup> - opaquely intermediate predicate
E<sup>=v</sup> - for an opaque expression of value v




### Analysis in Ghidra

adress 0x491aa0

---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- What is an "opaque predicate"? - https://reverseengineering.stackexchange.com/questions/1669/what-is-an-opaque-predicate
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Sample - https://github.com/mrphrazer/r2con2020_deobfuscation/blob/master/samples/ac3e087e43be67bdc674747c665b46c2
- Opaque Predicate - https://en.wikipedia.org/wiki/Opaque_predicate
