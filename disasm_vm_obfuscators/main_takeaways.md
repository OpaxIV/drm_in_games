# Writing Disassemblers for VM-based Obfuscators
_Summarized by Fabio Schmidt<br/>
Original Article by Tim Blazytko_<br/>
Ref: https://synthesis.to/2021/10/21/vm_based_obfuscation.html


## Virtual Machines
- Obfuscation by adding a custom "instruction set architecture" (ISA)
- Hide original code in a sequence of bytes (the so-called bytecode) that is interpreted at runtime.
- Control-flow graph: central basic block (dispatcher) which directs control flow to individual basic blocks, which then jump back to the dispatcher.
img 1

- Not Control-flow Flattening: individual basic blocks have a 1:1 mapping to the original code. ??
- In this case these basic blocks are known as handlers and implement the individual instruction semantics of the custom instruction set architecture, such as virtual addition or push/pop instructions. ??
 -High Level Description of VM:
	- Starting at the entry basic block, it backups the native CPU registers and initializes the VM state.
	- Afterward, it walks over the bytecode array, iteratively fetches some bytes, decodes the corresponding instruction and executes the handler that implements the virtual instruction.
	- The fetch-decode-execute (FDE) process is performed in a loop until the VM reaches a specific handler that restores the native CPU registers and leaves the VM. This handler is called the VM exit. 



<img src="" width="">
- Internally, virtual machines often use additional data structures to preserve their internal state (like storing native registers and intermediate values). Two of the most important data structures are the virtual instruction pointer and (sometimes) the virtual stack pointer.
  - Instruction Pointer: Similar to a native instruction pointer register (such as rip on x86-64), points to current instruction in the bytecode and used to decode operands and keep track of the VM execution flow.
  - Virtual Stack Pointer: Keeps track of a VM-internal stack that may be used to store intermediate values.
- The VM takes some parameters as inputs and calculates one or more outputs.
- Uses a prologue and epilogue that saves and restores the native CPU state.
- Internally, it operates on an undocumented state and interprets a sequence of bytes that represents the protected code.


---

## Symbolic Execution
- Program analysis technique which allows to symbolically evaluate and summarize assembly code. Summaries provide concrete insights into  semantics of executed instructions.
- To symbolically execute assembly code, we first lift it into an intermediate representation.
- Afterward, we evaluate the code assignment by assignment and track the individual register and memory assignments in a hashmap that is referred to as symbolic state.
- To propagate the data flow between the instructions, we always use the latest register/memory definitions from the symbolic state.
- Example:
	```
	mov rax, rbx                  ; rax := rbx
	add rax, 0x20                 ; rax := rax + 0x20
	add rbx, rax                  ; rbx := rbx + rax
	xor rcx, rbx                  ; rcx := rcx ^ rbx
	```
 	Assume that the initial symbolic state maps all registers to themselves:
  	```
	rax: rax
	rbx: rbx
	rcx: rcx
	```
  	Going trough the instructions, the following symbolic state is presented:
  	```
	rax: rax + 0x20
	rbx: rbx + (rax + 0x20)
	rcx: rcx ^ (rbx + (rax + 0x20))
	```
## Writing a Disassembler
   
