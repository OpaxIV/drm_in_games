## Virtual Machines
- Obfuscation by adding a costum "instruction set architecture" (ISA)
- Hide original code in a sequence of bytes (the so-called bytecode) that is interpreted at runtime.
- Control-flow graph: central basic block (dispatcher) which directs control flow to individual basic blocks, which then jump back to the dispatcher.
img 1

- Not Control-flow Flattening: individual basic blocks have a 1:1 mapping to the original code. ??
- In this case these basic blocks are known as handlers and implement the individual instruction semantics of the custom instruction set architecture, such as virtual addition or push/pop instructions. ??
 -High Level Description of VM:
	- Starting at the entry basic block, it backups the native CPU registers and initializes the VM state.
	- Afterward, it walks over the bytecode array, iteratively fetches some bytes, decodes the corresponding instruction and executes the handler that implements the virtual instruction.
	- The fetch-decode-execute (FDE) process is performed in a loop until the VM reaches a specific handler that restores the native CPU registers and leaves the VM. This handler is called the VM exit. 
img2
- Internally, virtual machines often use additional data structures to preserve their internal state (like storing native registers and intermediate values). Two of the most important data structures are the virtual instruction pointer and (sometimes) the virtual stack pointer.
  - Instruction Pointer: Similar to a native instruction pointer register (such as rip on x86-64), points to current instruction in the bytecode and used to decode operands and keep track of the VM execution flow.
  - Virtual Stack Pointer: Keeps track of a VM-internal stack that may be used to store intermediate values.
- The VM takes some parameters as inputs and calculates one or more outputs.
- Uses a prologue and epilogue that saves and restores the native CPU state.
- Internally, it operates on an undocumented state and interprets a sequence of bytes that represents the protected code.
## Symbolic Execution
