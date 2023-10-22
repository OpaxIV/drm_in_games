# Symbolic Execution
_Summarized by Fabio Schmidt<br/>
Original Article and Video Lecture by Dr. Tim Blazytko_<br/>



## Definition
Symbolic execution is defined as a technique of analyzing computer programs to determine what inputs cause each part of the program to execute.
This technique is based on assumptions, as inputs are placed "symbolically" (hence the name) rather than being the actual inputs when executing the program normally.
These rewrites of code provide concrete insights in the way a program could work without knowing the exact parameters that go with it.
The main focus is to determine the paths, which a program could take during execution: By symbolically placing pseudo-variables as the input, one can find multiple if not all paths that the control flow would take during execution.  

### Example
The following code shall provide an example of a case of symbolic execution.<br>
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
The condition to reach the complete end of this code is to make sure, that the sum of the variables x, y and z is not equal to the value of 3. These variable's values depend on the other three variables a, b and c. The thing is though, that those values are unknown to us. Hence to put some values symbollicaly in their places would help us to understand the control flow and the multiple possible paths the program may takes.
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
As we can see by the tree diagram, by just assuming the variables one can start to uncover and understand the multiple paths presented by the program. Furthermore by evaluating every path it can also be shown, which paths are more likely to be taken, if the goal is to successfully execute the program (hence reach the end of the programm's code).<br>
In this example every path is a valid path expect for the one marked by an `X`. This is the only executable path which would not lead to an complete errorless execution.



---

References:
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- Symbolic execution - https://en.wikipedia.org/wiki/Symbolic_execution
- What Is an Instruction Set Architecture? - https://www.arm.com/glossary/isa
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
- What is symbolic execution for software programs? - https://symflower.com/en/company/blog/2021/symbolic-execution/
- Symbolic Execution for finding bugs by Michael Hicks - https://www.cs.umd.edu/~mwh/se-tutorial/symbolic-exec.pdf
- tree.nathanfriend.io - https://gitlab.com/nfriend/tree-online#what-is-this

