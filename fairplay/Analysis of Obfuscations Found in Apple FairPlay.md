Analysis of Obfuscations Found in Apple FairPlay
================================================

August 28, 2023 – 38 min – 8082 words

**FairPlay** comprises a set of algorithms created by Apple for digital rights management (also called DRM, _digital rights management_). FairPlay is currently used to manage the decryption of iOS applications during their installation on Apple devices. In fact, we know that Apple distributes all applications in the Apple Store through the IPA file format. The IPA file format contains encrypted information that will then be used by the operating system to install an application. All of the encrypted information is handled through FairPlay, which takes care of keeping the decryption key and the whole process secure to avoid the possibility of decrypting the contents of.ipa files to share the contents of an app (perhaps paid for) in the wrong hands.

In this article, we are going to summarize some static protection measures that I was able to find within user-space daemons running FairPlay, the DRM system used by Apple. All information is believed to be current as of the date of the article; the operating system from which the binaries were extracted is macOS 13.5.1.

DRM systems and the protection of Apple applications
----------------------------------------------------

Protecting intellectual property in digital form has always been a goal on the part of companies that distribute copyrighted material. How can content be distributed without users being able to copy, view, edit, and redistribute it? Most current systems use DRM technology. Very briefly, DRM systems work in the following way: they receive as input a secret (it can be a film, an image, or an algorithm) that is not readable, for example, raw files, process the information, and transmit as output the original content. All this is done while trying to keep the process by which the original information was obtained as hidden as possible. DRM systems are most widely used for viewing copyrighted content: the user signs a contract with a service provider that promises to send the requested information (movies, TV series, books). To prevent copying of the content (the contract provides for only one user authorized to view it with that contract), the information must be accessible so that no copying or exporting of the content is expected.

From an Apple perspective, let’s assume that we want to download a paid application or new game through the Apple Store. We make the transaction, and our device gets a standalone archive, ready to be installed. Technically speaking, if there were no protection measures, a person could copy the app installer[1](#fn:1) and pass it on to any other person. Result? Loss of revenue on the part of Apple and the developer who published the application since the archive copy is free. Paradoxically, by treating the IPA file format as a kind of container, there would be no technical measure to stop application sharing (archives are standalone). This is where [FairPlay™](https://en.wikipedia.org/wiki/FairPlay) technology comes into play.

How can we protect the information contained within the archive? It is clear that we must somehow hide the contents of the archive; by doing so, even if they were to extract the IPA archive from the iPhone, attackers would not be able to access the contents. The information would have to be read only by the system processes that are in charge of installing the application. The next question would be: how can we hide the content? Here, an unparalleled Pandora’s box opens. One of the simplest and least expensive ideas from an archival point of view is to encrypt the information using a secret key. The information would be made readable again by a simple call to `decrypt(content, key)`.

The choice of secret key, however, is not a **banal** problem. If we think about the installation process, we know that the information will at some point have to be decrypted, that is, made readable again. Can we set a static key that applies to all iPhones sold? Certainly not. The choice of a static (i.e., hardcoded) key poses a variety of nuisances: attackers would only have to find a single key in order to gain access to all archives created by the Apple Store. However, once the key was discovered, the rest would come naturally: installers would be decrypted in no time, and the network would find itself flooded with “cracked” archives. The choice of the static key might seem advantageous to apply, perhaps by splitting the “visibility” of a set of keys per device (iPhone 12 will have a certain key, iPhone 13 a different one) by exploiting hardware[2](#fn:2). But even this inventiveness does not work; the key to sharing is across multiple devices!

The solution actually includes the generation of “dynamic” keys, i.e., depending solely on the installation device, the account of the user who paid for the applications, and a set of metadata exchanged during the transaction (to avoid spoofing). Apple encrypts the contents of the IPA file the moment it receives a new request from the Apple Store: encryption is done with a public key associated with the Apple account. Once received, the archive is decompressed and the individual binary is decrypted using the private key within the device. The application is thus installed in plaintext and remains in plaintext within the Apple device.

The point at which the decryption takes place constitutes a large centralization point that could be useful to analyze to try to get the IPA decrypted. If attackers could figure out how the private key is generated from the device and associated account, Apple’s protection system would fail instantly. With respect to the risk of spoofing, Apple includes within its devices certain stratagems to declare to cloud services (Apple Store, iCloud, Apple Signing) that indeed the packet exchange originates from an Apple device and not from an **issued** or **simulated** Apple device. Unfortunately due to a number of technical limitations it is not possible to prove 100% “_I am not a simulated device, give me the binary, I am \[myaccount@icloud.com\]_”.

Static analysis and obfuscation techniques
------------------------------------------

FairPlay constitutes a very important technological part of Apple’s technical expertise in intellectual property protection. Several patents have been developed to describe this technology and protect it: [_US8934624B2: Decoupling rights in a digital content unit from download_](https://patents.google.com/patent/US8934624B2), [_ES2373131T3: Safe distribution of content using descifrado keys_](https://patents.google.com/patent/ES2373131T3). Protecting how the decryption process works is the main goal of another set of “anti-reverse engineering” technologies named software obfuscation.

In fact, we know that among the wide range of techniques that any industry insider can devise to analyze software, there is a procedure called static analysis that allows for more specific investigation of certain parts of a piece of software without sending it into execution. The main technique of static analysis is reverse engineering, which is the reconstruction of the original code by inferring information from raw binary code.

Raw binaries in fact consist of two main pieces of information to function: data and instructions. Through a multitude of steps, programs such as [Ghidra](https://ghidra-sre.org/), [IDA](https://hex-rays.com/) or [Binary Ninja](https://binary.ninja/) can reconstruct much of the original source code. Although not perfectly, the software analyst is able to infer much of the semantics of the software: how it works, what methods it calls, and what information it uses from the operating system are some of the examples of questions we can answer through reverse engineering.

Reverse engineering allows one to derive to a good degree of approximation the algorithms that FairPlay uses in order to decrypt the content. Through the data it is then possible to try to figure out how the secret key is constructed, and with enough effort an attacker could figure out how to copy the decryption method to develop a decryption tool. Deriving the algorithm would be a problem for Apple and its investors. The solution to protect the instructions and data is to apply some forms of obfuscation that make the process of reverse engineering analysis more difficult.

Forms of obfuscation are part of a discipline of computer science called Software Security that aims to protect the code and information contained within a program. Examples of applications of obfuscation include: protecting intellectual property, making reverse engineering more difficult to prevent the discovery of vulnerabilities, and mitigating exploits. These techniques are applied to the syntax of a program (i.e., the raw instructions) and make it possible to hide much of the original semantics. Manipulating raw code is very complex, so it is necessary to have a solid foundation to be able to modify the code without side effects.

As we will see in the following paragraphs, FairPlay was constructed in a way that hides much of the instruction and data. Are these techniques, however, really effective? We will attempt to answer this at the end of the article. Recall, in fact, that obfuscation and code protection are techniques that must always be changed from release to release since there is no definitive solution to protect a piece of software. Through obfuscation we are able to complicate all attempts at reverse engineering, but we certainly cannot prevent or prevent the analysis of the binary.

### The fairplayd daemon

In the next few paragraphs we will attempt to analyze in detail a user-side daemon that can be found within Apple’s family of operating systems. The use of FairPlay is not limited only to the iOS mobile platform: FairPlay is also used by macOS to manage a secure channel through which to convey the digital content to be protected (movies, TV series..). This sub-technology is called [FairPlay Streaming](https://developer.apple.com/streaming/fps/) and allows copyrighted content to be distributed by encrypting the content. A document that summarizes at a high level how it works is [_FairPlay Streaming Overview_](https://developer.apple.com/streaming/fps/FairPlayStreamingOverview.pdf).

We then want to find out more about the use of FairPlay within macOS and see if the algorithms have been protected through obfuscations. We find that much of the management of FairPlay is assigned to a framework called `CoreFP.Framework` (**Core** **F**air **P**lay) found within the `/System/Library/PrivateFrameworks` folder. Private frameworks are a set of libraries dedicated to certain specific macOS features that are considered private, meaning that they have not been released for public use and all methods contained within are to be considered valid only for Apple applications.

CoreFP.Framework is currently used by some applications and daemons such as Safari and AMPLibraryAgent. Side note before continuing: if you have a private framework and are curious where it is currently being used, you can run the command `lsof | grep -i [name_framework]` where name\_framework is the name of the framework library. As a result we will see a number of active processes that have opened the private framework; in this case, we briefly dwell on AMPLibraryAgent. AMPLibraryAgent is a user-space daemon that is used to manage the user’s media (`TV.app` and `Music.app`). AMPLibraryAgent is a kind of intermediate process between the encrypted content coming from Apple’s servers and the end user interacting with the decrypted content through the `TV.app` and `Music.app` clients.

FairPlayd is the daemon that is invoked by AMPLibraryAgent and is used in practice to decrypt the contents. We then begin to analyze the contents of the `CoreFP.Framework` folder. Although the privateframeworks are contained within a dynamic cache called `dyld`, the binaries of `CoreFP.Framework` are available without any special arrangements that the user has to make on the dynamic cache. Within CoreFP, we can find: `CoreFP` and `fairplayd`. `CoreFP` is the binary that is used by system processes and constitutes the library, while `fairplayd` is the user space daemon. The user space daemon uses the kernel component called [FairPlayIOKit](https://github.com/pwn0rz/fairplay_research/blob/master/files/FairPlayIOKit_macOS_11.2_20D64).

We extract the x86\_64 version of `fairplayd` and save it to a folder of our choice via the command `lipo -extract x86_64 fairplayd ~/fairplayd`. The `fairplayd` file is a classic executable Mach-O file (object file format used by systems based on the Mach kernel), it has no particular fields within the header worth mentioning. So let\`s import it inside one of our reverse engineering tools (at will we can use Ghidra, IDA, Binary Ninja or Hopper). We use IDA for convenience in this article, although we must be especially careful when importing the binary into other tools (we will explain why at the end of the article). We let IDA work to reconstruct the information within the binary (through section parsing, disassembling, decompiling).

<img src="../../_resources/0f5726caa29c268b58a83c8026b027b5.png" width="1000"/>

By default IDA opens the binary by placing the reader on the main symbol, namely `_main`, the entry point of the `fairplayd` daemon. As we can see, we immediately touch on the power of obfuscation, and after a while we realize that the entire binary was actually constructed in such a way as to obfuscate all instructions. Confirmation of this is also given by the decompiler, below is a brief excerpt:

```C
    v298 = &v297;
    v297 = (((unsigned int) v298 &0x52491520 | (2 *(_DWORD) v298) &0x80120A40) ^ 0x40090520) + ((-1704077140 - ((unsigned int) v298 &0x8924A850)) &0x8924A854) + ((((unsigned int) v298 &0x24924288) + 689062540) &0x24924288 | (2 *(_DWORD) v298) &0x506D9510);
    v302 = 62;
    ((void(__fastcall*)(__int64, _QWORD, _QWORD, _QWORD))((char*) *(&off_1002B0BF0 + (unsigned int)(unsigned __int8)((unsigned __int8) v298 ^ byte_100237430[byte_1002AEEF0[(unsigned __int8) v298] ^ 0x3A]) +944) -790860942))(31 LL,0 LL,0 LL,0 LL);
    LODWORD(v303) = 1312628203 *((unsigned int) &v303 ^ 0x4EBE92AB) + 8;
    ((void(__fastcall*)(unsigned __int64 *))((char*) *(&off_1002B0BF0 +(unsigned int)(unsigned __int8)(byte_100237330[byte_1002AEDF0[(unsigned__int8) &v297] ^ 0x18] ^ (unsigned __int8) &v297) +	532) -1051853286))(&v303);
    LODWORD(v303) = 2064956458 - 1106503637 *(((unsigned int) &v303 - 2 *((unsigned int) &v303 &0x6F01D10) + 116399381) ^ 0x92F4EBC6);
    sub_10015D450(&v303);
    v21 = HIDWORD(v303);
    v22 = (HIDWORD(v303) == 1923298241) | 2; 
```
**Panic**! The analyst faced with such code has few choices: leave reverse engineering or try to investigate further. Most people applying obfuscation techniques hope that most of the time the analyst will choose the first path. If it is too complicated to analyze the behavior of an application by reverse engineering, it is not worth it. However, it is necessary for obfuscation to be well done before there can be any major slips. In this article we will try to find out that it actually only takes a simple look at the raw code to identify some common obfuscation patterns. In the next few paragraphs we will introduce some obfuscation techniques and how they are implemented.

Before continuing, we should mention that there are different obfuscation techniques depending on the resource to be protected. As we will see later, different obfuscation techniques have a different cost: the opaque predicate technique has an entirely different cost than control flow flattening. That said, let’s get started!

****Helpful Tip**: With the presence of obfuscations applied to instructions and data, it is a good idea to try to use the decompiler view as little as possible. In fact, the decompiler deduces some high-level information from how the machine instructions were placed within the binary and what data they work on. Since most of the instructions are intended to complicate the decompiler result, abandoning the decompiler result is always a good idea.**

### Mixed Boolean Arithmetic Expression

The first obfuscation technique we address is data obfuscation using mixed expressions with algebraic (sum, subtraction, multiplication, division) and boolean (logical operations) operators. These types of expressions are used when there is a need to hide a given numeric constant within a program. A constant datum could represent a value used in an encryption algorithm ([“Nothing-up-my-sleeve-number”](https://en.wikipedia.org/wiki/Nothing-up-my-sleeve_number)), but also a string, a memory address, and more. But not only that! If during processing, such as during decryption, a cryptographic algorithm performs a simple addition, it is possible to make the arithmetic expression more complex.

Examples of arithmetic-boolean expressions are: `v298 & 0x52491520 | (2 * (_DWORD) v298) & 0x80120A40) ^ 0x40090520) + ((-1704077140 - (v298 & 0x8924A850)) & 0x8924A854) + ((((unsigned int) v298 & 0x24924288) + 689062540) &0x24924288 | (2 * v298) & 0x506D9510)`. I have already written in a previous post [how to be able to create](https://nicolo.dev/blog/introduzione-pocket-offuscatore/) these expressions by applying transformation rules. The process Apple used is the same: take a constant from the code, rewrite the constant using arithmetic operators, and then apply the transformations. Do we already have an expression? We continue to apply the rules of transformations. Note that only some transformations can be applied since they do not change the semantics of the original expression. At the end of the process, the expression is translated back into machine language so that it can be reinserted within the binary.

In the case of `fairplayd`, numeric constants most often represent addresses at which to jump from one basic block to another. Addresses refer to other basic blocks or to stubs used to call methods from other libraries. We will discuss this option in more detail in obfuscating the control flow.

We can actually see two types of obfuscation of Boolean expressions depending on the type of assembly instruction used. We can identify a classic example of obfuscation by a Boolean expression:
```
    *(_BYTE *)(a1 + v2) = -13 * (-71 * (59 * *(_BYTE *)(a1 + v2) - 107) + 71 * (v2 & 0xF ^ 0x9C)) - 111;
``` 

This in assembly is translated as:

```
    cdqe
    movzx   ecx, byte ptr [rdi+rax]
    imul    ecx, 0x3B
    add     cl, 0x95
    movzx   ecx, cl
    imul    ecx, 0xB9
    mov     edx, eax
    and     edx, 0x0F
    xor     edx, 0x9C
    imul    edx, 0x47
    add     edx, ecx
    imul    ecx, edx, 0x0D
    add     cl, 0x91
    mov     [rdi+rax], cl
```    

We note that these assembly instructions are quite common to find within the Intel x86\_64 instruction set, and with a little effort it is possible to reduce the expression to a simplified expression through some framework, such as [Msynth](https://github.com/mrphrazer/msynth). These instructions are also classified historically as SISD: a single expression acting on a single data item (the sum between the `cl` and `0x95` registers has no side effects on the other registers).

Another type of logical arithmetic expression that I have currently found involves another type of instruction. The Intel x86\_64 instruction set uses a subsystem called MMX that is part of the SIMD (Single Instruction, Multiple Data) family of instructions and allows a single instruction to operate on multiple data. For example, a basic block of these instructions are:

    movdqu  	xmm0, xmmword ptr [rdi+rdx]
    pmovzxbw 	xmm3, xmm0
    punpckhbw 	xmm0, xmm0
    movdqa  	xmm1, cs:xmmword_100216450
    pmullw  	xmm0, xmm1
    pand    	xmm0, xmm6
    pmullw  	xmm3, xmm1
    pand    	xmm3, xmm6
    packuswb 	xmm3, xmm0
    paddb   	xmm3, cs:xmmword_100216470
    pshufd  	xmm0, xmm3, 0EEh
    pmovzxbd 	xmm8, xmm0
    pshufd  	xmm0, xmm3, 0FFh
    pmovzxbd 	xmm13, xmm0
    pshufd  	xmm0, xmm3, 55h ; 'U'
    pmovzxbd 	xmm9, xmm0
    pmovzxbd 	xmm10, xmm3
    movdqa  	xmm0, cs:xmmword_100216600
    pmulld  	xmm10, xmm0
    pmulld  	xmm9, xmm0
    pmulld  	xmm13, xmm0
    pmulld  	xmm8, xmm0
    movdqa  	xmm0, xmm14
    movdqa  	xmm4, cs:xmmword_100216480
    pand    	xmm0, xmm4
    

These kinds of operations are complex to translate into high-level code because they are dependent on the architecture and on whether one instruction can be converted into multiple high-level instructions (for example: paddb allows 128-bit addition between the xmm3 register and the data located at address xmmword\_100216470). IDA solves this problem by defining functions such as `mmu_addb(xmm3, xmmword_100216470)` but we do not know, however, what happens inside.

****Idea**: SIMD computation opens a number of opportunities for obfuscation based on MBA expressions: is it possible to use technology for [matrix computation](https://developer.apple.com/documentation/accelerate)/parallel to obfuscate a certain constant? The idea would be to have a series of MBA expressions to compute in parallel to find the result of a constant. This would lower the overhead given by introducing the complication of an expression (and since I have a lower overhead, I can increase the degree of illegibility of the expression). Plus it would be dependent on the Apple Silicon architecture, making it more difficult to emulate the computation. Possible problems with this approach include: finding equations that can be executed in parallel, checking how to interact with parallel cores (via software abstraction?), checking the semantics of expressions.**

The technique of obfuscation by mixed boolean arithmetic expressions has been widely used in the `fairplayd` binary. An excellent example of how this obfuscation has been used can be found in the function [sub\_1001F5D32](https://gist.github.com/seekbytes/696f71ec49b7e75728e94f5bea4dee46): the largest function within which Boolean arithmetic expressions are found. There are currently several methods for trying to summarize arithmetic Boolean expressions, but most rely on a single technique called **symbolic execution** and involve the use of Proof Solvers such as z3 to verify the semantics between the obfuscated expression and the synthesized expression.

The best tool you can currently use to solve arithmetic Boolean expressions is [goomba](https://github.com/HexRaysSA/goomba/), developed by HexRays and available by default in IDA Pro and IDA Teams versions 8.3. An excellent alternative is the [Msynth](https://github.com/mrphrazer/msynth) framework developed by [Tim Blazytko](https://synthesis.to), which succeeds well. Ultimately, while MBA obfuscation is a good alternative for making expressions more complicated, there are many tools for solving and rewriting expressions.

### Opaque Predicates

Opaque predicates are another very “cheap” technique for introducing obfuscation within instructions. This technique consists of introducing some always true or always false conditions that cause the decompiler to explore blocks of instructions with zero utility. The always true or always false conditions include a direct or indirect jump to basic blocks that will never be executed: they do not present additional functionality, they only add complexity to the functions being analyzed.

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

To our amazement we will find that the decompiler would struggle to reconstruct the original check, so it would miss some always true or always false checks. The defense given by these opaque predicates is proportional to the degree of illegibility and complexity given by the check made between the (high-level) ifs or the various algebraic arithmetic statements given before the `cmp` statement. Code within the ifs is presented as _dead code_ i.e., code that will never be executed: during decompilation, however, it is not possible to recognize between code that will run and dead code. Opaque predicates therefore add degrees of “confusion” to binary analysis.

A typical example of this transformation is the procedure `sub_100005FC0`: after the classical prologue, we can find a function call and then two different execution branches. The condition to jump into a branch or continue is given by the `test al, 2` instruction: at high level the condition would be equal to `if (al % 4 == 0)`, i.e. if the number stored inside register `al` is divisible by 4 continue with the execution, otherwise jump to `loc_10000505F`. The `test` instruction actually performs AND between the register `al` and the immediate value `0x2`: performing an AND with an immediate value means checking the remainder of the register division with the immediate value `al & 2 == al % 4`.

Visualizing the graph tree of the function we can see how the basic block just below the condition is a very large block.

![595b472a58f3a9ad505db85b6070a049.png](../../_resources/595b472a58f3a9ad505db85b6070a049.png)

Intuitively one would think that a very large basic block would be an integral part of the processing. If the program were not obfuscated, this is a very real assumption. However, we have to imagine that Apple’s developers have complicated the instructions precisely in order to challenge the automatic binary analysis tools. In this case, the largest basic block (TRUE branch of the condition `at % 4 == 0`) represents a clear example of **opaque predicate**. In fact, if we saw the most substantial elementary block, we could see a number of useless assembly operations.

![713e85c4fc7155e6540fc86a4f795337.png](../../_resources/713e85c4fc7155e6540fc86a4f795337.png)

Note that the screenshot has been cropped to avoid filling the entire article with the image of the basic block. We ask: Is it exactly a basic block that is useless for computation purposes? Is the condition `al % 4 == 0` always true or always false? The job of obfuscation would be to prevent any kind of inference within the condition. And it is: we cannot predict whether the instruction will always have a constant result or not.

What we can do, however, is to see the basic block present below the basic block of the TRUE condition `al % 4 == 0`. The basic block we are looking at then represents the FALSE condition (i.e. following `jnz loc_100005F2F`, we come to analyze the condition `loc_100005F2F`). A good way to check whether the largest basic block is actually useful or not is to see if there are any dependencies between the data used in the suspect basic block and the other basic blocks.

In this case, we can see that the basic block loads the address of xmmword\_1002B6460, which was previously used in the “suspect” basic block.

![c3127873658cc406b91481392654d38c.png](../../_resources/c3127873658cc406b91481392654d38c.png)

In the suspect basic block, the memory location `xmmword_1002B6460` is rewritten by an information present in another memory location. It is therefore difficult to verify the complex dependency between the instructions! However in this case, it is easy to check the dependency since the most “important” code for this function can be translated like this:
```C
    xmmword_1002B5460 = 0x0EC0C7C941423B0F77D59F9E25CFAC016;
    xmmword_1002142F0 = 0x6432B8A1D491C1746754EB00EF0F9478;
    
    // prologo
    [... omissis ...]
    // eax = 
    if (al % 4 == 0){
    	// dead code
    	xmmword_1002B5460 = xmmword_1002142F0;
    }
    memcpy(v6, &xmmword_1002B5460, 0x1000);
    
    // epilogo
```    

What is the value of the variable `xmmword_1002B5460`? Much depends on the condition of the if. However, even in uncertainty, the obfuscation given by the opaque predicate turns out to be not very powerful: we have two choices, this means that we can propagate most of the changes in a parallel way (verifying what happens if we enter the TRUE condition block or inside to block the FALSE condition). Other questions that might come to mind:

*   how are we sure that the assembly code of the largest basic block is composed of “dead code”? The instructions currently show that other data is overwritten, but that data is never used. The question remains open on other procedures that could use this data (even if the analysis of the references has not highlighted this possibility, we cannot trust 100% of the binary analysis tools).
    
*   how do we make sure that `xmmword_1002142F0` is not overwritten by other functions? As usual, we use the powerful XREF tool: there are no other procedures that read/write the memory of `xmmword_1002142F0`. But remember that XREF checks based on information gleaned from automated binary analysis tools. Even just one piece of information lost during the initial parsing can cause major headaches at the end of the parse (ie during reference scanning).
    
*   how can we be sure that the condition `al % 4 == 0` is not always satisfied? We could only answer this question when we have invented the crystal ball. What matters to us is not so much verifying whether the condition is true or false: we actually want to carry out an analysis of the data to understand the effects of the “suspect” basic block of being dead code. To verify this, we carried out an analysis on the basic block immediately following the suspected block. If we are in doubt, as in this case, we can also propagate the changes to the following blocks to see if in one way or another we can infer some other data.
    

To answer the question “so what does `sub_100005FC0` do in practice?”, just look at the basic block just after the `memcpy` function call. Let us remember that the signature (that is the declaration of the function) is the following: `sub_100005FC0(int64 a1, int a2)`.
```C
    for ( i = 0; i != a2; ++i ){
    	v4 = i + 15;
    	if ( i >= 0 )
    		v4 = i;
    
    	*(_BYTE *)(a1 + i) = -13 * v6[256 * (__int64)(int)(i - (v4 & 0xFFFFFFF0))
    	 	+ (unsigned __int8)(59 * *(_BYTE *)(a1 + i) - 107)] - 111;
    }
```

The `sub_100005FC0` procedure then proceeds to iterate over the v6 buffer to deobfuscate the contents of the word `xmmword_1002B5460`. The most expensive instruction, computationally speaking, is an MBA expression to be deobfuscated. Please refer to the MBA subsection to understand how to deobfuscate the expression and rewrite it. If you have problems with pointers, I can rewrite the expression as:
```C
    a1[i] = -13 * v6[256 * (i - (v4 & 0xFFFFFFF0)) + 59 * a1[i] - 107] - 111;
```

The obfuscation measure in this case is **not effective** and allows you to recover much of the original information. There are numerous cases of the tendency to include “useless” opaque predicates (like this one discussed) within the `fairplayd` program. The opaque predicates introduced by Apple don’t give you a headache: a good software analyst would be able to distinguish between dead branches and instructions that cannot be executed. Apple needs to review the construction of the opaque predicates to make the basic blocks even more complex.

****Idea**: To make obfuscation even more effective, you would need to make each elementary block more complex (perhaps trying to make the control flow more complicated). A clear example of how hard the decompiler’s job can be made is presented in the subsection **control flow flattening**.**

### Moving the stack

Before continuing with our discussion, let’s stick with the analysis of the `sub_100005FC0` function and display the prologue of the procedure before the `test al, 0x2` statement.

![277c91df9f2e40f8eca448282de03a73.png](../../_resources/277c91df9f2e40f8eca448282de03a73.png)

Do we notice anything strange? I confess that it was not easy to analyze, but after the analysis of IDA we understand another “obfuscation” technique that Apple has applied to protect `fairplayd`. Very subtly the stack is moved up (or down, depending on how you want to build the stack). Why would a program want to move the stack?

All instruction analysis software provides, among the many analysis steps, a particular type of insight called “stack analysis”. This technique allows you to recover how the stack and local variables are composed: it is essential to be able to correctly deduce how the stack is composed, otherwise the analysis software is no longer able to understand the flow control and the various function calls.

For this procedure, the type of error that IDA reports is `sp-analysis failed` i.e. it fails to trace how the stack pointer is modified. In our case, IDA fails to realize that the shift is done precisely to confuse the analysis. Indeed with the instruction `mov eax, 1010h` and `sub rsp, rax`[3](#fn:3), IDA is led to think that there is a local buffer called `var_1020` of size 0x1010 bytes. This is just one of many examples where IDA fails to rebuild the original stack.

Other examples can be found in functions with multiple statements: the procedure `sub_10000F620` has a parameter on the stack that occupies 0x2CE7AB73 bytes (753380211 bytes = about 753 MB). Such a large parameter on the stack we know cannot exist: the stack can be increased in size, but usually reaches **maximum** 65520kb (higher values cause errors in macOS). The value 753 MB is given by an basic block in which the `sub esp, 0x2CE7AB73` instruction is used, an unreachable basic block, therefore considered dead code. However, IDA does not have the tools to realize that such a value is too large to exist: IDA does not recognize that its analysis is inaccurate, on the contrary it refuses to decompile the function because the size of the stack frame is too large. With a single instruction, Apple manages to put a major barrier to anyone who wants to decompile the feature.

Is it so difficult to fix this situation? For a beginner, yes. For a motivated reverse engineering student, no. Fortunately, IDA allows the analyst to be able to modify part of the information that he has deduced. By redefining the stack, much of the 753MB of variable can be ignored. An alternative way is to report a certain path as dead code (just select the bytes of that block and not define it via the option called _Undefine_). In the case of moving the stack pointer up, when we try to make software analysis complex, we’re not really talking about obfuscation: we’re not affecting the readability of the code. Rather, let’s act on some limitations that these tools have to recover information. A good article grouping some techniques to confuse the analysis techniques applied by IDA was written by Markus Gaasedelen: [Dangers of the Decompiler](https://blog.ret2.io/2017/11/16/dangers-of-the-decompiler/).

### Control Flow Flattening

Control Flow Flattening is another obfuscation technique found within the FairPlay binary and is perhaps the most powerful used by Apple in the field of code protection for `fairplayd`. Flow control indicates how the program evolves over time in terms of instruction flow. What functions a procedure calls, what kind of jumps the check makes (if conditional or unconditional), the link between the various basic blocks, the conditions for which the execution moves and many other information are inferred from the flow control.

The high-level instructions of flow control are the typical instructions that allow you to divide the execution of a program into one or more cases: `if`, `if else`, `if else if else`, `switch`, `do while`, `for` and many others. Most of these conditions translate into machine language instructions which for Intel become `cmp`, `test`, `jmp`. If the high-level flow control change is dictated by “logical” instructions, the low-level flow control change is to change the address of the next instruction, or rather, jump to a certain instruction.

Among the information that we can deduce from the flow control, we can derive the various connections between the basic blocks and proceed to carry out the intra-procedural analysis, i.e. try to reconstruct the program execution flow between the various procedures. The flow of control also allows to recover important information (such as loops) to subsequently proceed to the translation from machine code to pseudo C code.

To more clearly show the power of obfuscation using control flow flattening, let’s take an example function: `sub_10003FE60`. From IDA, we select the example function, right click and choose the tree view. The tree view is a feature of IDA which allows you to understand: how the basic blocks are connected, the dependencies between the blocks and the execution branches. Often through this view it is possible to get a better idea of what the pseudo code translated by the decompiler will look like.

The goal of the control flow flattening technique is to flatten the control flow, i.e. to transform the control flow through some conditional and unconditional jumps. The technique was developed by [Chenxi Wang](https://www.darkreading.com/edge-articles/chenxi-wang-from-security-research-to-developing-the-next-generation-of-security-leaders) in his doctoral thesis entitled _A Security Architecture for Survivability Mechanisms_. Normally the control flow of a procedure is almost developed vertically, as you can see in the image below.

![2c406f7c7a7148352eb0dee98b46d96b.png](../../_resources/2c406f7c7a7148352eb0dee98b46d96b.png)

With the application of control flow flattening, the procedure graph is heavily modified by making it “more horizontal,” that is, flattened or _flatten_. Here is a typical example of how a function can be modified by applying (simple) control flow flattening:

![8853939d0cebb2bff884509e91f0fd05.png](../../_resources/8853939d0cebb2bff884509e91f0fd05.png)

We can then see how the basic blocks have all been brought to the same de facto level by horizontally extending the graph of basic blocks. The case of control flow flattening taken to the extreme drives the analyst crazy, such as this function depicted below:

<img src="../../_resources/05a2769d532278e2402601c2acd0d51d.png" width="800" />

As can be seen from the image just above, control flow flattening can be applied to construct even bogus basic blocks that can confuse the decompiler. The end result is a control flow that is extremely complex to analyze. At a high level, the transformation technique can be translated by a simple switch. Suppose we have a procedure:
```C
    int sum(int a, int b){
    	int s = 4;
    	int c;
    	if ( a > 500 ){
    		c = 2;
    	} else {
    		c = 0;
    	}
    	int result = s + a + b + c;
    	return result;
    }
```    

The control flow of this simple function is given by the following graph:

![79d766cb380c2d1975647e4a66793c78.png](../../_resources/79d766cb380c2d1975647e4a66793c78.png)

Now we want this vertical graph to be as flattened as possible! To do this we use another powerful construct of programming languages: the `switch` instruction. The `switch` instruction allows the execution to be divided into multiple branches, creating basic blocks arranged horizontally. To switch from one branch to another in the switch, however, we can use an auxiliary variable, called `state`, which stores the current state we are in. Does this sound familiar? This technique sounds a lot like the synthesis of a finite state machine! The auxiliary variable helps to change state without worrying about some side effects (or without the use of labels and goto). Let us then move on to the actual transformation! For now, let us assume that we do not add any unnecessary basic blocks.

The steps we perform are as follows:

1.  **Creation of the “start”** block: contains the initialization of the state variable and all the variables that will be used within the basic blocks of the switch. It is possible to move the declarations within the switch branches, however, we must pay attention to possible side effects of using local variables internal to a construct.
    
2.  **Dispatcher creation**: This basic block will check what state to jump to. It is a simple `while` with a switch statement inside: this is to avoid that after entering one of the switch branches, the jump out reaches the last statement of the function.
    
3.  **Move basic blocks**: I choose a new number for a new state. I insert the code within the identified basic block including the state change. The state variable should point to the next basic block I want to execute. I remind you that I can also create new elementary blocks by subdividing larger basic blocks or adding instructions that will never be executed. The limit for obfuscation is imagination :- )
    
4.  **Transformation check**: I check that indeed the function can terminate, paying attention to the last block that is executed. All transformations of obfuscations should actually be checked through some proof solvers and mathematical equations to prevent the original semantics from being changed during the obfuscation process.
    

Applying control flow flattening:
```C
    int sum(int a, int b){
    	int state = 0;
    	int s, c, result;
    	while(1){
    		switch(state){
    			case 2:
    				if (a > 500){
    					state = 4;
    				} else {
    					state = -1;
    				}
    				break;
    			case 0:
    				s = 4;
    				state = 2;
    				break;
    			case -1:
    				c = 0;
    				state = 5;
    				break;
    			case 5:
    				result = s + a + b + c;
    			case 0x54292639:
    				return result;
    			case 4:
    				c = 2;
    				state = 5;
    				break;
    		}
    	}
    	return -542926392;
    }
```   

The result:

![2b1ac03427dbb7f5b1703fe8f44c0081.png](../../_resources/2b1ac03427dbb7f5b1703fe8f44c0081.png)

Some notes we write per point:

*   the graph does not contain the while instruction to avoid adding too many arrows and making the graph more confusing. When a branch of a switch ends with the `break` instruction, execution resumes from the evaluation of the `state` variable. The graph does not contain the last instruction `return -542926392;`, it should be inserted with an arrow after dispatcher execution.
    
*   `basic block A` and `basic block B` are two basic blocks that were inserted to highlight the difference in the control flow whether the variable `a` results in greater than 500 or not. If indeed the condition is true, we change state by going to state 4, otherwise we go to state -1. Note that the number for each state is trivially a number chosen at random to identify each state with a label.
    
*   An attacker could still predict the instruction execution flow by following the evolution of the `state` variable. There are some gimmicks to obfuscate the value of the `state` variable such as using MBA expressions and aliasing. It is also possible to introduce unnecessary blocks, opaque predicates and dynamically construct the switch to divert the analyst or hinder the decompiler’s analysis.
    
*   the variable `state` is never assigned with the value `0x54292639`. So how do we ensure that the function terminates by returning `result`? In the previous code we used a small stratagem: we take advantage of not mentioning `break` within the switch branch. In this case, execution goes immediately from state `5` to state `0x54292639` because the program is executed sequentially (shown as “**C Magic**”). Sometimes it is possible that the compiler applies optimizations to not create the branch of 0x54292639, so be sure to disable the optimizations if you want the graph to be the same as the image above.
    

Just out of curiosity, let’s try compiling a program that uses both `sum` functions to see how effective this kind of technique is. The program in fact has two functions `sum_1`, the obfuscated function, and `sum_2`, the original function. The `sum_2` function is decompiled correctly:
```C
    __int64 __fastcall sum_2(int a1, int a2){
      int v3; // [rsp+4h] [rbp-10h]
    
      if ( a1 <= 500 )
        v3 = 0;
      else
        v3 = 2;
      return (unsigned int)(v3 + a2 + a1 + 4);
    }
```    

The function is very similar to the original. In addition, the control flow graph constructed by IDA is not suspicious and looks the same as the graph constructed by us manually. Different is the case for `sum_1`:
```C
    __int64 __fastcall sum_1(int a1, int a2){
      int v3; // [rsp+8h] [rbp-14h]
      int v4; // [rsp+Ch] [rbp-10h]
      int i; // [rsp+10h] [rbp-Ch]
    
      for ( i = 0; ; i = 5 )
      {
        while ( 1 )
        {
          while ( 1 )
          {
            while ( i == -1 )
            {
              v3 = 0;
              i = 5;
            }
            if ( i )
              break;
            v4 = 4;
            i = 2;
          }
          if ( i != 2 )
            break;
          if ( a1 <= 500 )
            i = -1;
          else
            i = 4;
        }
        if ( i != 4 )
          break;
        v3 = 2;
      }
      return (unsigned int)(v3 + a2 + a1 + v4);
    }
```

Meanwhile, we can say that we have managed to confuse the IDA decompiler. Indeed, we notice that a for is constructed, then 3 `while(1)` loops one inside the other. Although the result of the decompiler may not be exactly correct and is different from the original code, the program works. An attacker would then be able to pick up the program logic. This is a typical example of weak obfuscation: the attacker may have some difficulty reconstructing the original control flow, but there is a good chance that the initial logic can be recovered. To make it even more complex, we can use the other techniques: insertion of opaque predicates, transformations of constants, transformations of arithmetic Boolean expressions.

Returning to `fairplayd`, the control flow flattening technique is used heavily to try to obfuscate the control flow evolution that fairplay has during execution. We open with IDA the function `sub_10003FE60` and extract the logic of operation via the decompiler view. In particular, we can see that the decompiler can understand the use of a high-level switch-like structure:
```C
    switch ( (v7 == 0) + v2 ){
        case 0:
          JUMPOUT(0x100035E2CLL);
        case 1:
          JUMPOUT(0x10004359ELL);
        case 2:
          JUMPOUT(0x10005BECDLL);
        case 3:
          v46 = v3;
          v45 = a1;
          v8 = v2 ^ 6u;
          v9 = v8 - 5;
          v10 = ((_DWORD)v8 - 5) | 2u;
          ....
    }
```

However, by examining all cases within the switch, we will have nonsense instructions! This is because Apple’s developers have managed to heavily obfuscate the state variable, namely `v2` and `v7`. On the other hand, the decompiler recognizes the switch construct only by the presence of **jump tables**, particular tables whose entries consist of memory addresses to jump to. Suppose you have 5 branches of a switch, you can store 5 different addresses in memory, each associated with a different branch of the switch. When comparing, you would simply jump to the address pointed to the correct cell in the table. These tables are stored at specific locations in the binary, and with a little luck it is possible to restore them all.

To raise the bar of analysis difficulty even higher, the developers decided that for some functions the jump table would be recreated dynamically. Very briefly, the program allocates a memory space where addresses can be entered that will later be used as branches of a switch. The calculation of the addresses is done by the usual MBA expressions, and due to a number of technical limitations of static analysis it is not possible to determine the addresses exactly. This in a nutshell completely destroys any possibility for IDA and the analyst to recover the original program logic and structure.

Typical example of address construction comes from the last portion of code in each basic block (the fact that address construction occurs inline adds complexity to the code):
```
    lea     r12, jpt_10003FF12								; address where to fetch the next address
    movsxd  rax, ds:(jpt_10003FF12 - 100218890h)[r12+rax*4] ; move the address referred by jpt_10003FF12 - 100218890h
    lea     rcx, loc_10005A140 								; load the address of loc_10005A140 (base address of jump table)
    add     rcx, rax										; add the rax value
    mov     r14, [rbp+var_58]								; save previous variable on stack
    jmp     rcx												; jump to next switch case
```

The control flow flattening technique thus turns out to be a good obfuscation technique by which Apple is able to prevent static analysis by attackers. Dynamic table construction, combined with the set of techniques mentioned in the previous subsections (opaque predicates, MBA) are the key to fairplayd obfuscation.

Are further obfuscation measures possible?
------------------------------------------

Throughout this article we have tried to identify some common obfuscation patterns that can be found in `fairplayd`, a userspace daemon used by Apple to protect copyrighted content within macOS. Some measures have proved effective: they actually complicate any attempt to accurately reverse engineer the program. The measure that is most complicit in this complication is given by the control flow flattening which eliminates the chances for an attacker to recover the original structure of the program. Attackers are faced with dynamically constructed jump tables, unrecoverable switches, and dynamically generated code.

Other obfuscation techniques seem to be good ideas, but only in theory: opaque predicates are simple enough to parse, and through a more thorough check of the building blocks it is possible to distinguish between the original code and the modified code to introduce obfuscation.

Is there a perfect fit to be able to obfuscate the code? Is it possible to devise a technique capable of producing unassailable protection? No. It will always be an ongoing evolution between tools that try to deobfuscate hidden content and new techniques to be able to bypass these tools and apply heavier obfuscation. This is given by the limitation of digital content management: the information must ultimately be able to be visible and therefore deobfuscate, as opposed to other types of information that can be hidden forever inside the devices. It is likely that after yet another article about obfuscation within `fairplayd` the protection measures will be updated again, causing the article content to be out of date.

One possible avenue that Apple could pursue to apply more powerful obfuscation is to move much of the execution to an obfuscation technique called “virtualization”. This technique involves writing an interpreter and a completely ad-hoc instruction set architecture to perform the operations necessary to decrypt the content. At the moment the technique doesn’t seem to be used inside `fairplayd`, but there are some signs (like switches with a lot of branches) that seem to predict the use of data evaluation and dynamic code production. Honestly, I haven’t had time to analyze this part in detail yet.

As always, if you have any criticisms, suggestions or any other comments, you can write an email to [seekbytes@protonmail.com](mailto:seekbytes@protonmail.com)[4](#fn:4). I never want to be the voice of the truth, so reports of errors or inaccuracies are always welcome.

* * *

1.  It should be noted that the term “installer” means the IPA package that contains the executable files, the resources and the signed metadata to be able to start the application. [↩︎](#fnref:1)
    
2.  This technique is actually already used within the Apple world to encrypt and decrypt the firmware content. The key is called [GID Key](https://theapplewiki.com/wiki/GID_Key) and is a 256-bit AES key shared by all devices with the same processor. Part of this key can be used to dynamically generate your iCloud account key for verifying Apple Store purchases. [↩︎](#fnref:2)
    
3.  Looking at the original code, we can see that there is a call to `sub_100008B80`, however it doesn’t affect the original result. The procedure is a simple loading of the canary stack into the stack. [↩︎](#fnref:3)
    
4.  Special thanks go to the talk [_“Revealing the secrets in binaries using code detection strategies”_](https://www.youtube.com/watch?v=y95MNr2Xu-g&ab_channel=ReconConference) by [Tim Blazytko](https://synthesis.to) which made it possible to speed up a large part of the binary analysis using the information given by the length of the blocks, their arrangement within the control flow graph. [↩︎](#fnref:4)
    

Developed in some autumn afternoon – 2021-2023 – [RSS feed](/index.xml)