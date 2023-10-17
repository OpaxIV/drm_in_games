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

_P<sup>T</sup> - opaquely true predicate_ <br>
The condition is always satisfied, hence "true". The following example shows, that either expression `2` or `(x^2 + x)^T` must be correct. Since `2` is always true, the branch on the left will always be taken.
```C
if (2|(x^2 + x)^T){
...
}
else {
...
}
```


_P<sup>F</sup> - opaquely false predicate_ <br>
The condition is never satisfied, hence "false".
Consider the following example:
```C
if (2 == 3){
...
}
else {
...
}
```
The expression `2 == 3` will never be true, hence the else-statement is always the executed branch.


_P<sup>?</sup> - opaquely intermediate predicate_ <br>
This type of opaque predicate combines the two types above. It is uncertain, which path gets taken since the expression can be true in one case and be false in another case.
```C
if (x mod 2 == 0){
...
}
else {
...
}
```
This code snipplet presents the expression `x mod 2 == 0`, which depending on the value of `x`, can be either true or false.
For example for `x` being equal to 2, the if-statement would be executed, since this would fullfil the condition (2 mod 2 == 0).
On the other hand `x = 4` would not satisfy this condition and the code in the else-block would be executed instead. 


_E<sup>=v</sup> - for an opaque expression of value v_ <br>

@ fabio: zu ergänzen


### Analysis in Ghidra
Beginning at the address 0x491aa0, one can regognize the high complexity of the control flow graph of this binarys function.
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/e507b626-41a5-43dd-8d44-d87edf457f1a" width="700">
<br/>
Since the amount is so vast, only a sample set of randomly chosen candidates will be analyzed in further detail.

#### Sample at 0x491b38
<br>
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/ec0d8dde-7edc-4b11-af79-f4d8d884cede" width="900">
<br/>




#### Sample at 0x491b38
<br>
<img src="" width="900">
<br/>




#### Sample at 0x491b38
<br>
<img src="" width="900">
<br/>




















---
References:
- Code Obfuscation - https://www2.cs.arizona.edu/~collberg/Teaching/553/2011/Resources/obfuscation.pdf
- What is an "opaque predicate"? - https://reverseengineering.stackexchange.com/questions/1669/what-is-an-opaque-predicate
- Used sample: https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c
- Sample - https://github.com/mrphrazer/r2con2020_deobfuscation/blob/master/samples/ac3e087e43be67bdc674747c665b46c2
- Opaque Predicate - https://en.wikipedia.org/wiki/Opaque_predicate
