# List of main takeaways / new learnings of this article

### This file contains definitions and texts of new learned things through the article.

##  Opaque Predicates
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

https://nicolo.dev/en/blog/fairplay-apple-obfuscation/


## Mixed Boolean-Arithmetic Obfuscation
Mixed Boolean-Arithmetic (MBA) obfuscation is a method to perform a semantics-preserving transformation from a simple expression to a representation that is hard to understand and analyze. More specifically, this obfuscation technique consists of the mixture usage of arithmetic operations (e.g., ADD and IMUL) and Boolean operations (e.g., AND, OR, and NOT). Binary code with MBA obfuscation can effectively hide the secret data/algorithm from both static and dynamic reverse engineering, including advanced analyses utilizing SMT solvers. Unfortunately, deobfuscation research against MBA is still in its infancy: state-of-the-art solutions such as pattern matching, bit-blasting, and program synthesis either suffer from severe performance penalties, are designed for specific MBA patterns, or generate too many false simplification results in practice.
Examples of arithmetic-boolean expressions are:
`v298 & 0x52491520 | (2 * (_DWORD) v298) & 0x80120A40) ^ 0x40090520) + ((-1704077140 - (v298 & 0x8924A850)) & 0x8924A854) + ((((unsigned int) v298 & 0x24924288) + 689062540) &0x24924288 | (2 * v298) & 0x506D9510).`

