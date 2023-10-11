# Writing Disassemblers for VM-based Obfuscators
_Summarized by Fabio Schmidt<br/>
Original Article by Tim Blazytko_<br/>
Ref: https://synthesis.to/2021/10/21/vm_based_obfuscation.html

Some information also refers to the video lecture of the same author and topic:
Ref: https://www.youtube.com/watch?v=b6udPT79itk

## Virtual Machines
- One of the most sophisticated opfuscation schemes.
- Obfuscation by adding a custom "instruction set architecture" (ISA).
- Hide original code in a sequence of bytes (bytecode) that is interpreted at runtime.
- Control-flow graph: Consists of a central basic block (dispatcher) which directs control flow to individual basic blocks, which then jump back to the dispatcher.
- Similar but not same as Control-flow Flattening:
	- In Control-Flow Flattening the individual basic blocks have a 1:1 mapping to the original code.
 	- In the case of VM-based obfuscation these basic blocks are known as handlers and implement the individual instruction semantics of the custom ISA (e.g. virtual addition or push/pop instructions).
- High Level Description of VM-based obfuscation:
	- Start is at the entry basic block, it backups the native CPU registers and initializes the VM state.
	- Afterwards, it walks over the bytecode array, iteratively fetches some bytes, decodes the corresponding instruction and executes the handler that implements the virtual instruction.
	- The fetch-decode-execute (FDE) process is performed in a loop until the VM reaches a specific handler that restores the native CPU registers and leaves the VM, called "VM exit".
 
Combining it all together, the graph could look like the following: 
<img src="https://github.com/OpaxIV/hslu_secproj/assets/93701325/80e2498c-f847-4abe-bd74-cc79fa5daf7d" width="800">


- Internally, virtual machines often use additional data structures to preserve their internal state (like storing native registers and intermediate values).
- Two of the most important data structures are the **virtual instruction pointer** and (sometimes) the **virtual stack pointer**.
  - Instruction Pointer: Similar to a native instruction pointer register (such as rip on x86-64), points to current instruction in the bytecode and used to decode operands and keep track of the VM execution flow.
  - Virtual Stack Pointer: Keeps track of a VM-internal stack that may be used to store intermediate values.
- The VM takes some parameters as inputs and calculates one or more outputs.
- Uses a prologue and epilogue that saves and restores the native CPU state.
- Internally, it operates on an undocumented state and interprets a sequence of bytes that represents the protected code.

### Architectures
#### Stack-Based Architectures
- pop arguments from stack
- push results onto stack
- examples: JVM, CPython, WebAssembly, …

#### Register-Based Architectures
- pass arguments in virtual registers
- store results in virtual registers
- examples: Dalvik, Lua, LLVM, …

_Note: hybrid architectures also possible!_




---

References:
- Analysis of Virtualization-based Obfuscation (r2con2021workshop) - https://www.youtube.com/watch?v=b6udPT79itk
- What Is an Instruction Set Architecture? - https://www.arm.com/glossary/isa
- Writing Disassemblers for VM-based Obfuscators - https://synthesis.to/2021/10/21/vm_based_obfuscation.html
   
