# List of main takeaways / new learnings of this article
This file contains definitions and texts of new learned things through the article. Some parts have intentionaly been copied directly from the appended sources.
<br>

## Table of Contents
1. [Opaque Predicates](#opaque)
2. [Mixed Boolean-Arithmetic Obfuscation](#mixedboolean)
3. [Control-Flow Flattening](#controlflow)
<br>

## Opaque Predicates <a name="opaque"></a>
Opaque predicates are another very “cheap” technique for introducing obfuscation within instructions. This technique consists of introducing some always true or always false conditions that cause the decompiler to explore blocks of instructions with zero utility.
The always true or always false conditions include a direct or indirect jump to basic blocks that will never be executed: they do not present additional functionality, they only add complexity to the functions being analyzed.

Assume, for example, that we have the following source code:
```C
int sum(int a, int b){
	int result = a + b + c;
	return result;
}
```

To protect the given part of the result we can rewrite the code like this:
```C
int sum(int a, int b){
	int result = a + b;

	if(a == 0 && a == 1 && a - 4 >= 55 && (a * 4 - 36 * 0xff - 0xc) < 2){
		result += 1 * 4 << 2 - 0x5c;
	} else if (a == 5 && a != 5){
		if(b > 4 && b < 4 && b != 4 && 352610 == 122){
			result += 50 * 0xf5 * 352610;
			result += decrypt(key);
		}
	} else {
		return result + decrypt(key);
	}
}
```

To our amazement we will find that the decompiler would struggle to reconstruct the original check, so it would miss some always true or always false checks. The defense given by these opaque predicates is proportional to the degree of illegibility and complexity given by the check made between the (high-level) ifs or the various algebraic arithmetic statements given before the cmp statement. Code within the ifs is presented as dead code i.e., code that will never be executed: during decompilation, however, it is not possible to recognize between code that will run and dead code. Opaque predicates therefore add degrees of “confusion” to binary analysis.

References:
- https://nicolo.dev/en/blog/fairplay-apple-obfuscation/

---

## Mixed Boolean-Arithmetic Obfuscation <a name="mixedboolean"></a>
Mixed Boolean-Arithmetic (MBA) obfuscation is a method to perform a semantics-preserving transformation from a simple expression to a representation that is hard to understand and analyze. More specifically, this obfuscation technique consists of the mixture usage of arithmetic operations (e.g., ADD and IMUL) and Boolean operations (e.g., AND, OR, and NOT). Binary code with MBA obfuscation can effectively hide the secret data/algorithm from both static and dynamic reverse engineering, including advanced analyses utilizing SMT solvers. Unfortunately, deobfuscation research against MBA is still in its infancy: state-of-the-art solutions such as pattern matching, bit-blasting, and program synthesis either suffer from severe performance penalties, are designed for specific MBA patterns, or generate too many false simplification results in practice.
Examples of arithmetic-boolean expressions are:
<br/>
`v298 & 0x52491520 | (2 * (_DWORD) v298) & 0x80120A40) ^ 0x40090520) + ((-1704077140 - (v298 & 0x8924A850)) & 0x8924A854) + ((((unsigned int) v298 & 0x24924288) + 689062540) &0x24924288 | (2 * v298) & 0x506D9510).`

References:
- https://nicolo.dev/en/blog/fairplay-apple-obfuscation/
- https://www.usenix.org/conference/usenixsecurity21/presentation/liu-binbin

---

## Control-Flow Flattening <a name="controlflow"></a>
The basic method for ﬂattening a function is the following.

First, we break up the body of the function to basic blocks, and then we put all these blocks, which were originally at diﬀerent nesting levels, next to each other.
The now equal-leveled basic blocks are encapsulated in a selective structure (a switch statement in the C++ language) with each block in a separate case, and the selection is encapsulated in turn in a loop.

Finally, the correct ﬂow of control is ensured by a control variable representing the state of the program, which is set at the end of each basic block and is used in the predicates of the enclosing loop and selection.

*Image showing how control-flow flattening obfuscation alters code that contains loop structures.*
![image](https://github.com/OpaxIV/hslu_secproj/assets/93701325/cc931a63-ae8a-491e-9933-bbff04a057f4)

```C
int original()
{
    print "Do"
    print "you"
    print "like"
    print "milk?"
}


int obfuscated()
{
    int ctrFlowVar = 1;

    while(ctrFlowVar != 0)
    {
        switch(ctrFlowVar)
        {
            case 1:
                print "do"
                ctrFlowVar = 2;
                break;
            
            case 2:
                print "you"
                ctrFlowVar = 3;
                break;
            
            case 3:
                print "like"
                ctrFlowVar = 4;
                break;
            
            case 4:
                print "milk?"
                ctrFlowVar = 0;
                break;
        }
    }
}
```

If you are familiar with how switch statements are written in assembly (i know 2 ways, the if-style and the jumptable one) then the above example is easy to de-obfuscate.The break; instruction is a jmp.You could make it jump to the block thats supposed to be next.

References:
- https://reverseengineering.stackexchange.com/questions/2221/what-is-a-control-flow-flattening-obfuscation-technique

---

