Ref: https://synthesis.to/2021/10/21/vm_based_obfuscation.html
    Writing Disassemblers for VM-based Obfuscators

[Twitter](https://twitter.com/mr_phrazer) [Training](https://www.hexacon.fr/trainer/blazytko/) [About Me](/index.html)

Writing Disassemblers for VM-based Obfuscators

Writing Disassemblers for VM-based Obfuscators
==============================================

21 Oct 2021 - Tim Blazytko

After I recently gave a workshop on the _Analysis of Virtualization-based Obfuscation_ at [r2con2021](https://rada.re/con/2021/) ([slides](/presentations/r2con2021-deobfuscation.pdf), [code & samples](https://github.com/mrphrazer/r2con2021_deobfuscation/) and the [recording](https://www.youtube.com/watch?v=b6udPT79itk) are available online), I would like to use this blog post for a brief summary on how to write disassemblers for VM-based obfuscators based on symbolic execution.

While we cover only a basic implementation of virtualization-based obfuscation, we focus on a generic approach to build disassemblers that scale for most virtual machines that can be found in the wild. As a leading example for this blog post, we will use a simple virtual machine that protects a [Fibonacci implementation](https://en.wikipedia.org/wiki/Fibonacci_number) using the [Tigress C obfuscator](https://tigress.wtf). In the end, we will also have a look at how the original code can be reconstructed from the VM disassembly.

If you would like to play around with the code (based on [Miasm](https://github.com/cea-sec/miasm)) and samples, you can find them [here](https://github.com/mrphrazer/r2con2021_deobfuscation/).

Virtual Machines
----------------

[Virtual machines](https://tigress.wtf/virtualize.html) (VM) belong to the most sophisticated obfuscation schemes, since they create an additional layer of complexity for reverse engineers: By introducing a custom [instruction set architecture (ISA)](https://en.wikipedia.org/wiki/Instruction_set_architecture) in software (similar to the [Java virtual machine](https://en.wikipedia.org/wiki/Java_virtual_machine)), they hide the original code in a sequence of bytes (the so-called _bytecode_) that is _interpreted_ at runtime. In other words, the bytecode is an array of bytes that encodes the to-be-protected program as machine code; without further knowledge of the virtual machine implementing the custom instruction set architecture, the bytes aren’t _meaningful_ for a reverse engineer. To give them a meaning, we have to fully understand the VM and its execution flow, such that we can build a disassembler for the custom instruction set and decode the bytecode.

To achieve this, let us start with opening a virtual machine in the disassembler of our choice. For the simplest virtual machines, we see a control-flow graph like the following:

![vm](/images/vm1.svg)

We see a control-flow graph that has a central basic block—the _dispatcher_—which directs the control flow to individual basic blocks, which jump back to the dispatcher. While this reminds us of [control-flow flattening](/2021/03/03/flattening_detection.html), there is a fundamental difference: In the case of control-flow flattening, the individual basic blocks have a 1:1 mapping to the original code. However, for virtual machines, these basic blocks are known as _handlers_ and implement the individual instruction semantics of the custom instruction set architecture, such as virtual addition or push/pop instructions.

On a high-level, the virtual machine operates as follows: Starting at the _entry_ basic block, it backups the native CPU registers and initializes the VM state. Afterward, it walks over the bytecode array, iteratively _fetches_ some bytes, _decodes_ the corresponding instruction and _executes_ the handler that implements the virtual instruction. The _fetch-decode-execute_ (FDE) process is performed in a loop until the VM reaches a specific handler that restores the native CPU registers and leaves the VM. This handler is called the VM _exit_. Given this context, we can annotate the control-flow graph as follows:

![vm](/images/vm2.svg)

Internally, virtual machines often use additional data structures to preserve their internal state (like storing native registers and intermediate values). Two of the most important data structures are the _virtual instruction pointer_ and (sometimes) the _virtual stack pointer_. Similar to a native instruction pointer register (such as `rip` on x86-64), the virtual instruction pointer points to the current instruction in the bytecode and is used to decode operands and keep track of the VM execution flow. Analogously, the virtual stack pointer keeps track of a VM-internal stack that _may_ be used to store intermediate values.

Overall, we can understand a VM as an obfuscated function with a custom calling convention: It takes some parameters as inputs and calculates one or more outputs. To preserve the outer execution context, it makes use of a [prologue and epilogue](https://en.wikipedia.org/wiki/Function_prologue_and_epilogue) (VM entry and exit) that saves and restores the native CPU state. Internally, it operates on an undocumented state and interprets a sequence of bytes that represents the protected code. To reconstruct the underlying code, we have to locate the virtual machine’s inputs/outputs, understand its internal state and instruction set. Then, we can write a disassembler for the custom instruction set architecture which decodes the bytecode into human-readable pseudocode. In the following, we will have a look at how we can write a generic disassembler based on symbolic execution. Beforehand, we make a short excursion into symbolic execution.

Symbolic Execution
------------------

_Symbolic execution_ is a program analysis technique which allows us to symbolically evaluate and summarize assembly code. These summaries provide concrete insights into the semantics of executed instructions.

To symbolically execute assembly code, we first lift it into an intermediate representation. Afterward, we evaluate the code assignment by assignment and track the individual register and memory assignments in a hashmap that is referred to as _symbolic state_. To propagate the data flow between the instructions, we always use the latest register/memory definitions from the symbolic state. For example, consider the following assembly code and the corresponding assignments:

    mov rax, rbx                  ; rax := rbx
    add rax, 0x20                 ; rax := rax + 0x20
    add rbx, rax                  ; rbx := rbx + rax
    xor rcx, rbx                  ; rcx := rcx ^ rbx
    

Let us assume that the _initial_ symbolic state maps all registers to themselves:

    rax: rax
    rbx: rbx
    rcx: rcx
    

After the first instruction, we update the symbolic state such that `rax` now maps to `rbx`. If we evaluate the second instruction, we propagate this information and assign `state[rax] + 0x20 = rbx + 0x20` to `rax`. After we symbolically executed all instructions, we have the following symbolic state:

    rax: rax + 0x20
    rbx: rbx + (rax + 0x20)
    rcx: rcx ^ (rbx + (rax + 0x20))
    

The symbolic state reveals the underlying semantics of a sequence of code in a comprehensive manner. If we want to enrich symbolic analysis with (partially) concrete values, we can provide them to the initial symbolic state and perform a _concolic execution_. For example, if we initialize `rax` with `0x10`, we obtain

    rax: 0x30
    rbx: rbx + 0x30
    rcx: rcx ^ (rbx + 0x30)
    

In a purely concolic scenario, symbolic execution can be understood as an emulator for assembly code. However, if we keep at least some registers/memory locations symbolic, we get a very powerful technique which allows us to automatically explore virtual machine interpreters. In the following, we will make use of this to build a generic disassembler.

Writing a Disassembler
----------------------

Our goal is to design a generic disassembler approach that works for most virtual machines in the wild, no matter their complexity. Starting from scratch with nearly no prior knowledge of the virtual machine, we interactively add more and more knowledge until we have a fully-fledged disassembler. For this, we build a system which symbolically explores the virtual machine and enrich the output with useful information about the symbolic state. On a high-level, the approach consists of two steps:

1.  We build a symbolic executor that _follows the VM execution flow_ from the VM entry to the VM exit.
2.  Afterward, we add _callbacks to interesting code locations (such as VM handlers)_ and dump useful information from the symbolic state.

Building such a symbolic executor is an interactive process: Starting at the VM entry, we symbolically follow the execution flow until the symbolic executor stops. This is always the case if the VM does a control-flow transfer that relying on symbolic values. Then, the symbolic executor has _insufficient_ information to derive a concrete value. In these cases, we manually inspect why this happens and add additional knowledge to the symbolic state. Often, these are either missing information about the bytecode or (conditional) jumps which depend on user input (inputs to the VM from the outside). For the former, we concretize the symbolic memory around the bytecode; for the latter, we can hardcode specific input values.

Once the symbolic executor follows the execution from VM entry to exit without any interruption, we can manually reverse engineer specific VM components and add more and more knowledge about the VM to the executor. Typically, we start by dumping the virtual instruction pointer and track all the executed handlers. Afterward, we manually inspect individual handlers, understand their semantics and add this information to the symbolic executor. In the end, we receive a fully-fledged disassembler which allows us to dump concrete values at runtime, print and optimize the disassembly or reconstruct the control-flow graph. In theory, it is even possible to perform a devirtualization and emit equivalent x86-64 code, although this may take quite some time to implement correctly.

In Practice
-----------

After we discussed the generic approach to build a disassembler based on symbolic execution, we will now exemplify the process on a simple virtual machine. For a better understanding, we will first give a detailed overview of the virtual machine. Afterward, we explore the process from symbolically following the execution flow to building a fully-fledged disassembler.

### Sample

Our [sample of choice](https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/vm_basic.bin?raw=true) protects an [implementation of Fibonacci](https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/samples/src/fib.c) with a virtual machine whose control-flow graph is comparable to the one above. The virtual machine is _stack-based_; it uses a dedicated stack to store intermediate values; for example, a calculation such as `5 + 8` is performed by the following pseudocode:

    push 5
    push 8
    add
    

The VM entry is at `0x115a`. The virtual machine uses `rdx` as the virtual instruction pointer and `rcx` as the virtual stack pointer; the bytecode is located at `0x4060`. The handler `0x11a9` performs a stack-based addition:

    000011a9  4883c201           add     rdx, 0x1
    000011ad  8b01               mov     eax, dword [rcx]
    000011af  0141f8             add     dword [rcx-0x8], eax
    000011b2  4883e908           sub     rcx, 0x8
    000011b6  eb4f               jmp     0x1207
    

For this, it fetches two arguments from the stack—`[rcx]` and `[rcx-0x8]`—adds them and stores them on the top of the stack. It also increments the virtual instruction pointer in `0x11a9` and the virtual stack pointer in `0x11b2`. Overall, the virtual machine has the following 11 handlers:

*   `0x129e`: CMPBE (compare below equal)
*   `0x1238`: PUSHFROMVAR (load integer from local variable and push onto stack)
*   `0x126d`: CHECKZERO (if constant == 0x0 then push `var_c` on top of stack)
*   `0x11c4`: PUSHPTR (push pointer to local variable)
*   `0x1262`: GOTO (jump to an address)
*   `0x11a9`: ADD (add two dwords)
*   `0x1245`: VMEXIT (leave the VM)
*   `0x11f1`: CMPE (compare if two values are equal)
*   `0x11e1`: PUSH (push constant onto stack)
*   `0x1281`: JCC (conditional jump)
*   `0x1226`: POPTOVAR (assign value to local variable)

To perform the Fibonacci calculation, it expects an input parameter in `rdi` representing the n-th Fibonacci number to calculate; after the VM exit, `rax` holds the output (the n-th Fibonacci number).

If you want to reverse engineer the virtual machine on your own, use the provided information as a guidance. However, let us now forget these analysis details and start writing the disassembler without any prior knowledge.

### Following the VM Execution Flow

To build a symbolic executor that follows the VM execution flow from the VM entry to the exit, we use the script [follow\_execution\_flow.py](https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/follow_execution_flow.py). It hardcodes the VM entry at `0x115a` and uses the following loop to symbolically follow the execution flow:

    # init worklist
    basic_block_worklist = [ExprInt(start_addr, 64)]
    
    # worklist algorithm
    while basic_block_worklist:
        # get current block
        current_block = basic_block_worklist.pop()
    
        print(f"current block: {current_block}")
    
        # symbolical execute block -> next_block: symbolic value/address to execute
        next_block = sb.run_block_at(ira_cfg, current_block)
    
        print(f"next block: {next_block}")
    
        # is next block is integer or label, continue execution
        if next_block.is_int() or next_block.is_loc():
            basic_block_worklist.append(next_block)
    

In short, it implements a worklist algorithm that iteratively executes the next basic block _until_ the calculation of the next basic block relies on symbolic values. Initially, the output looks as follows:

    current block: 0x115A
    next block: 0x1207
    current block: 0x1207
    next block: 0x120A
    current block: 0x120A
    next block: (@8[0x4060] == 0x80)?(0x129E,0x1212)
    

We successfully executed the basic blocks `0x115A`, `0x1207` and `0x120A`. However, the next basic block relies on a memory value that the symbolic executor cannot resolve: `@8[0x4060]` (which can be understood as an 8-bit memory read from the address `0x4060`). Let’s have a look at the corresponding disassembly:

    00001178  488d15e12e0000     lea     rdx, [rel data_4060]
    [...]
    00001207  0fb602             movzx   eax, byte [rdx]
    [...]
    0000120a  3c80               cmp     al, 0x80
    

We notice that the comparison with `al` relies on the memory dereference `[rdx]`, which again relies on the address `0x4060`. In other words, the symbolic executor propagated the memory read to the byte comparison: `@8[0x4060] == 0x80`. Semantically, the virtual machine reads the first byte from the bytecode and dispatches it to find the first handler. To allow the symbolic executor to perform the dispatching, we empower it with knowledge of the bytecode by adding it to the symbolic memory:

    # add bytecode to symbolic memory -- start address and size (highest address - lowest address)
    sym_address, sym_value = constraint_memory(0x4060, 0x4140 - 0x4060)
    sb.symbols[sym_address] = sym_value
    

Afterward, the symbolic executor runs successfully. The next time it stops it looks as follows:

    [...]
    current block: 0x120A
    next block: 0x1212
    current block: 0x1212
    next block: 0x1218
    current block: 0x1218
    next block: 0x121C
    current block: 0x121C
    next block: 0x121E
    current block: 0x121E
    next block: 0x1281
    current block: 0x1281
    next block: RDI[0:32]?(0x1298,0x1286)
    

This time, we stop at a conditional jump that relies on `rdi`. After some manual investigation, we come to the conclusion that this is an input parameter to the VM. Let us initialize it with `0` as follows:

    # constraint VM input (rdi, first function argument). The value in `ExprInt` represents the function's input value.
    rdi = ExprId("RDI", 64)
    sb.symbols[rdi] = ExprInt(0, 64)
    

If we re-run again, the symbolic executor stops here:

    [...]
    next block: 0x1245
    current block: 0x1245
    next block: 0x125A
    current block: 0x125A
    next block: @64[RSP]
    

Now, the next basic block is an address taken from the stack; a typical pattern for a `ret` instruction. Indeed, if we look at the corresponding handler we see that it leaves the VM; we found the VM exit. In other words, our symbolic executor now runs from the VM entry to the VM exit. If we have a closer look at the VM exit, we also see the following assembly line:

    00001245  8b01               mov     eax, dword [rcx]
    

Here, we use the virtual stack pointer `rcx` and load a value into `rax`. In other words, we found the VM output. To verify our assumption, we can add the following code lines after the worklist algorithm to print the VM output:

    # dump VMs/functions' return value -- only works if SE runs until the end
    rax = ExprId("RAX", 64)
    value = sb.symbols[rax]
    print(f"VM return value: {value}")
    

If we re-run the script, we receive `0` as output; if we change the input value in `rdi` to `10`, we get as output `55`. In other words, the symbolic executor correctly emulates the VM execution for the provided inputs and calculates the same output as the binary (remember that it calculates the n-th Fibonacci number).

In summary, we have not only built a symbolic executor following the VM execution flow from the VM entry to its exit; we were also able to emulate the whole VM and inspect its input and output. In the following, we will extend the analysis and enrich the symbolic emulator with information about the individual VM handlers.

### From Symbolic Execution to VM Disassembly

In the next step, we want to turn the symbolic executor into a fully-fledged disassembler. For this, we want to add callbacks for the individual handlers which dump additional information from the symbolic state. To realize this, we first start by manually creating a list of handlers. Then, we add the callback in the worklist algorithm as it can be seen in [vm\_disassembler.py](https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/vm_disassembler.py):

    # worklist algorithm
    while basic_block_worklist:
        # get current block
        current_block = basic_block_worklist.pop()
    
        # if current block is a VM handler, dump handler-specific knowledge
        if current_block.is_int() and int(current_block) in VM_HANDLERS:
            disassemble(sb, current_block)
        [...]
    

If the current symbolically executed basic block is the beginning of a VM handler, we call the function `disassemble` which takes the handler and the symbolic state as input. Within the function, we dispatch the individual handler addresses and dump handler-specific information. For now, let us just create a template and print the virtual instruction pointer:

        # fetch concrete value of current virtual instruction pointer
        vip = sb.symbols[ExprId("RDX", 64)]
    
        # catch the individual handlers and print execution context
        if int(address) == 0x129e:
            print(f"{vip}: handler {address}")
        [...]
    

If we execute the script, we get the following handler trace:

    0x4060: handler 0x11E1
    0x4065: handler 0x11C4
    0x406A: handler 0x1226
    0x406B: handler 0x11E1
    0x4070: handler 0x11C4
    0x4075: handler 0x1226
    0x4076: handler 0x11C4
    0x407B: handler 0x1238
    0x407C: handler 0x11C4
    0x4081: handler 0x1226
    0x4082: handler 0x1262
    0x4087: handler 0x11E1
    0x408C: handler 0x126D
    0x4091: handler 0x1238
    0x4092: handler 0x11F1
    0x4093: handler 0x1281
    0x409D: handler 0x11E1
    0x40A2: handler 0x11C4
    0x40A7: handler 0x1226
    0x40A8: handler 0x1262
    0x40AD: handler 0x1262
    0x4136: handler 0x11C4
    0x413B: handler 0x1238
    0x413C: handler 0x1245
    

Looking at the trace, we see that some handlers are executed more than once. To give the instruction trace more meaning, we can now manually inspect the individual handlers and replace the output with handler-specific information. For example, we could start and replace the last line `handler 0x1245` with `VMEXIT`. This process might be time-consuming; however, it becomes easier the more handlers we already know since we permanently improve our understanding of how the VM works internally. In the end, we can enrich the disassembler as it can be seen in [vm\_disassembler\_final.py](https://github.com/mrphrazer/r2con2021_deobfuscation/blob/main/vm_disassembler_final.py). Its final output is:

    0x4060: PUSH 0x0
    0x4065: PUSHPTR var_0x4 (push pointer to local variable var_0x4)
    0x406A: POPTOVAR (assign value to local variable)
    0x406B: PUSH 0x1
    0x4070: PUSHPTR var_0x8 (push pointer to local variable var_0x8)
    0x4075: POPTOVAR (assign value to local variable)
    0x4076: PUSHPTR var_0x8 (push pointer to local variable var_0x8)
    0x407B: PUSHFROMVAR (load integer from local variable and push onto stack)
    0x407C: PUSHPTR var_0xC (push pointer to local variable var_0xC)
    0x4081: POPTOVAR (assign value to local variable)
    0x4082: GOTO 0x4087
    0x4087: PUSH 0x0
    0x408C: if 0x0 == 0x0 then push var_c (rdi) on top of stack
    0x4091: PUSHFROMVAR (load integer from local variable and push onto stack)
    0x4092: CMPE
    0x4093: conditional jump
    0x409D: PUSH 0x0
    0x40A2: PUSHPTR var_0x10 (push pointer to local variable var_0x10)
    0x40A7: POPTOVAR (assign value to local variable)
    0x40A8: GOTO 0x40AD
    0x40AD: GOTO 0x4136
    0x4136: PUSHPTR var_0x10 (push pointer to local variable var_0x10)
    0x413B: PUSHFROMVAR (load integer from local variable and push onto stack
    0x413C: VMEXIT
    

Given this detailed VM disassembly output, we can now have a closer look at the VM disassembly, detect patterns, simplify the output and reconstruct the high-level code.

### Code Reconstruction

To reconstruct the high-level code for the observed VM trace, let’s first have a closer look at some instruction patterns:

    0x4060: PUSH 0x0
    0x4065: PUSHPTR var_0x4 (push pointer to local variable var_0x4)
    0x406A: POPTOVAR (assign value to local variable)
    

This pattern occurs several times; it pushes a constant onto the stack, afterward a local variable offset. Finally, it assigns the value to the local variable. In short, this pattern can be simplified to `var_0x4 = 0x0`.

    0x4076: PUSHPTR var_0x8 (push pointer to local variable var_0x8)
    0x407B: PUSHFROMVAR (load integer from local variable and push onto stack
    0x407C: PUSHPTR var_0xC (push pointer to local variable var_0xC)
    0x4081: POPTOVAR (assign value to local variable)
    

This instruction pattern first pushes a pointer to a local variable onto the stack, afterward it fetches its value. Then, we push another pointer to a variable and assign it the value from the stack. In short, it can be written as `var_0xc = var_0x8`.

Let’s also have a closer look at the conditional jump:

    0x4087: PUSH 0x0
    0x408C: if 0x0 == 0x0 then push var_c on top of stack
    0x4091: PUSHFROMVAR (load integer from local variable and push onto stack)
    0x4092: CMPE
    0x4093: conditional jump
    

We first push `0x0` onto the stack. Then, we compare `0x0 == 0x0` and if this is true (which it is in this case), we put `var_c` onto the stack. We didn’t explore it in depth, but `var_c` holds the value of `rdi`, the input to the VM. This way, we put `rdi` onto the stack. Finally, we compare the two stack values if they are the equal: `if rdi == 0x0: [...]`.

If we do this for all instruction patterns and remove the gotos, we receive a cleaned output:

    ; var_0x4 = 0x0
    0x4060: PUSH 0x0
    0x4065: PUSHPTR var_0x4 (push pointer to local variable var_0x4)
    0x406A: POPTOVAR (assign value to local variable)
    
    ; var_0x8 = 0x1
    0x406B: PUSH 0x1
    0x4070: PUSHPTR var_0x8 (push pointer to local variable var_0x8)
    0x4075: POPTOVAR (assign value to local variable)
    
    ; var_0xc = var_0x8
    0x4076: PUSHPTR var_0x8 (push pointer to local variable var_0x8)
    0x407B: PUSHFROMVAR (load integer from local variable and push onto stack
    0x407C: PUSHPTR var_0xC (push pointer to local variable var_0xC)
    0x4081: POPTOVAR (assign value to local variable)
    
    ; push rdi (== 0) onto stack
    0x4087: PUSH 0x0
    0x408C: if 0x0 == 0x0 then push var_c on top of stack
    0x4091: PUSHFROMVAR (load integer from local variable and push onto stack
    
    ; if rdi == 0
    0x4092: CMPE
    0x4093: conditional jump
    
    ; var_0x10 = 0x0
    0x409D: PUSH 0x0
    0x40A2: PUSHPTR var_0x10 (push pointer to local variable var_0x10)
    0x40A7: POPTOVAR (assign value to local variable)
    
    ; return var_0x10
    0x4136: PUSHPTR var_0x10 (push pointer to local variable var_0x10)
    0x413B: PUSHFROMVAR (load integer from local variable and push onto stack
    
    0x413C: VMEXIT
    

If we also remove the VM disassembler output and re-write our comments in pseudocode, we obtain:

    var_0x4 = 0x0
    var_0x8 = 0x1
    var_0xc = var_0x8
    
    if rdi == 0:
        var_0x10 = 0x0
        return var_0x10
    else:
        print("not seen in VM trace")
    

We might see that it is the initialization of the Fibonacci function where we initialize two values and compare the input to `0x0`. Since we initialized `rdi` with `0x0` in the symbolic executor, the VM execution triggers exactly this case. To dispel the last doubts, we can also compare it with the original, unprotected code:

    unsigned int fib(unsigned n) {
      unsigned a=0;
      unsigned b=1;
      unsigned int s = b;
    
      if (n == 0) {
        return 0;
      }
      [...]
    

As we can see, our reconstruction nearly fits the original code snippet line by line. To reconstruct the missing parts of the original code, we can assign other values to `rdi` and re-run the whole process.

This has been a long journey, but in the end we analyzed a virtual machine, wrote an interactive disassembler based on symbolic execution and reconstructed the original code based on the VM disassembly.

Setting the Scene
-----------------

In this post, we discussed the fundamentals of virtualization-based obfuscation and learned a generic approach on how to build disassemblers for virtual machine implementations using symbolic execution. Afterward, we put it into practice on a simple virtual machine for which we not only constructed a disassembler, but also recovered parts of the protected high-level code. Overall, the whole process has been an interplay between manual analysis and automation; especially the manual work required a lot of time. In general, large parts of this time-consuming manual work can be simplified by using heuristics, pattern matching and other analysis techniques; we could, for example, detect the handlers automatically and also extract their semantics. This may be a topic for a future post.

In the wild, there exist a plethora of virtualization-based obfuscators of varying complexity that are deployed in malware, commercial applications and even in the [Windows kernel](/2021/08/10/obfuscation_detection.html). Naturally, the example code from this post will break horribly on such samples: function calls, indirect calls, jump tables, external/API calls, additional layers of obfuscation and others must be treated in special ways. Nevertheless, the core concepts remain the same.

If you would like to dive deeper into these topics, learn how to combine this approach with other deobfuscation techniques such as compiler optimizations, taint analysis, SMT solving & program synthesis or if you would like to know how you can handle the aforementioned special cases, then have a look at my [training class on code deobfuscation techniques](/training_software_deobfuscation.html).

Contact
-------

For questions, feel free to reach out via Twitter [@mr\_phrazer](https://twitter.com/mr_phrazer), mail [tim@blazytko.to](mailto:tim@blazytko.to) or various other channels.
