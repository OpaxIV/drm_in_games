# Symbolic Execution
_Summarized by Fabio Schmidt<br/>
Original Article and Video Lecture by Dr. Tim Blazytko_<br/>



## Definition
Symbolic execution is defined as a technique of analyzing a computer program to determine what inputs cause each part of a program to execute.
This technique is based on assumptions, as inputs are placed "symbolically" (hence the name) rather than being the actual inputs when executing the program normally.
These rewrites of code provide concrete insights in the way a program could work without knowing the correct parameters that go with it. It can be imagined as a tool to uncover the inner workings of an even blacker box.
Possible inputs that trigger a branch can then be determined by solving these constraints.

@fabio erneut anpassen, nicht so sicher und korrekt

### Example
The following example shall provide an example for a case of symbolic execution.<br>
Consider the following C-code:
```C
int a = α, b = β, c = γ; 		// unknown, only symbolic
int x = 0, y = 0, z = 0;

if (a) {
	x = -2;
}

if (b < 5) {
	if (!a && c) {
		y = 1;
	}
	z = 2;
}

assert(x+y+z!=3)
```
The condition to reach the end of this code is to make sure, that the sum of the variables x, y and z is not equal to the value 3. These variables values depend on the other three variables a, b and c. The thing is though, that those values are unknown to us. Hence to put some values symbollicaly in their places would help us to understand the control flow and the multiple possible paths it may takes.
Every possible path has been written down in a three diagram to further elaborate on this point.
_Note: path conditions are written at the end of every path._
<br>
```
O = valid path
X = invalid path

.
├── x=0, y=0, z=0
└── α
    ├── true
    │   └── x=-2
    │       └── β<5
    │           ├── true
    │           │   └── z=2
    │           │       └── α∧(β<5)
    │           │           └── O
    │           └── false
    │               └── α∧(β≥5)
    │                   └── O
    └── false
        └── β<5
            ├── true
            │   └── ¬α∧γ
            │       ├── true
            │       │   └── y=1
            │       │       └── z=2
            │       │           └── ¬α∧(β<5)∧γ
            │       │               └── X
            │       └── false
            │           └── z=2
            │               └── ¬α∧(β<5)∧¬γ
            │                   └── O
            └── false
                └── ¬α∧(β≥5)
                    └── O
```
<br/>
Each symbolic execution path stands for many actually program runs. In fact, exactly the set of runs whose concrete values satisfy the path condition.
By this procedure one can cover a greater part of the program’s execution space than using e.g. testing as a technique.
@ fabio, gegebenfalls erneut anpassen




---

References:
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- Symbolic execution - https://en.wikipedia.org/wiki/Symbolic_execution
- What Is an Instruction Set Architecture? - https://www.arm.com/glossary/isa
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- What is symbolic execution for software programs? - https://symflower.com/en/company/blog/2021/symbolic-execution/
- Symbolic Execution for finding bugs by Michael Hicks - https://www.cs.umd.edu/~mwh/se-tutorial/symbolic-exec.pdf
- tree.nathanfriend.io - https://gitlab.com/nfriend/tree-online#what-is-this

