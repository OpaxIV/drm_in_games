Secproj Fragen 16.10.2023

- Beispiel und Definition von Opaque Predicate
--> Ein klares Ausgehen einer Bedingung, die aber trotzdem zu überprüfen ist.
--> Grundsätzliche Idee ist daher, dass durch das Einfügen immer mehr "unnötiger" Branches der Kontrollfluss an Volumen zunimmt.




-  "Is hides the original code in a sequence of bytes, which are then interpreted at runtime."
--> Was genau ist "bytecode"?
	-->  Anderer Begriff für "machine code"
	--> repräsentiert instruktionen als machinen code
	--> Im Falle einer VM gibt es virtuelle Instruktionen (bspw. virtuelle addition)
	--> Eine VM hat eine eigenständige ISA, die je nach VM also unterschiedlich sein kann.
	--> Der originale Code wird in der VM verschleiert, diese übersetzt den Originalcode in VM assembler / byte repräsentation
	--> Der byte code ist nichts anderes als ein array: es wird eine Funktion nach der anderen (bspw. `d5` an der Adresse  0x104060 ist die erste)
	--> Durch Inkremetierung werden dann im array die nächste(n) Instruktionen geholt, zeigt dann auf nächste intruktion.


- Unterschiedliche Längen der Instruktionen aus dem Bytearray?
	--> Jede Instruktion ist anders und kann daher auch einer anderen bytelänge entsprechen
	--> Beispiel wäre an 0x101262 


- RDX: instruction pointer, wird addiert zur grösse der jetzigen instruction --> ist der wert (ADD rbx 0x5) daher irrelevent? bsp. 001011e1
	--> Der Virtual Intruction Pointer hat nichts mit dem nativen Instruction Pointer zu tun.
	--> Ich habe mich immer gefragt, wieso der Instructionpointer nur manchmal addiert und manchmal nicht. Diesem sollte ja eigentlich immer etwas dazuaddiert werden, sodass er auch auf die neuste Instruktion zeigt.
	--> Da dieser jedoch unabhängig ist, ist alles umso klarer. Hier ein Beispiel:
	--> Wie im Decompiler zu sehen, werden Instruktionen trotzdem weiter ausgeführt, auch wenn der VIP gar nicht angerührt wird:
	```
            virtual_IP = virtual_IP + 1;
            *(uint *)(ppiVar2 + -1) =
                 (uint)(*(uint *)(ppiVar2 + -1) < *(uint *)ppiVar2 ||
                       *(uint *)(ppiVar2 + -1) == *(uint *)ppiVar2);
            ppiVar2 = ppiVar2 + -1;
	```		
			
			
- 001012b9: korrekt, dass der nicht virtuelle stack wiederhergestellt wird?
	--> korrekt, hier geht es um den "nativen" stack.
	--> ergebnis wird in eax geschrieben 
	--> Stack aufgeräumt

- Wie erkennt man virtuelle und native Instruktionen?
	--> bsp: 
	--> Im Grunde ist jede der Instruktionen eine Instruktion, die auf der nativen CPU ausgeführt wird.
	--> Virtuelle Instruktionen, als die auf einer VM ausführt werden, bestehen quasi aus einer Reihe von nativen Instruktionen.
	--> Im folgenden Beispiel (basic block), von 10129e bis 1012b4, sind acht x86-64 instruktionen zu sehen, die alle zusammen in Kombination eine entsprechende virtuelle Instruktion bilden (High-Level)
	```
	                             LAB_0010129e                                    XREF[1]:     0010120c(j)  
	        0010129e 48 83 c2 01     ADD        virtual_IP,0x1
	        001012a2 8b 01           MOV        EAX,dword ptr [RCX]=>local_128
	        001012a4 39 41 f8        CMP        dword ptr [RCX + local_130],EAX
	        001012a7 0f 96 c0        SETBE      AL
	        001012aa 0f b6 c0        MOVZX      EAX,AL
	        001012ad 89 41 f8        MOV        dword ptr [RCX + local_130],EAX
	        001012b0 48 83 e9 08     SUB        RCX,0x8
	        001012b4 e9 4e ff        JMP        LAB_00101207
	                 ff ff
	
	```


- Wie würde eine register based architecure aussehen?
	-->register wie x86

- wie weitermachen?
	--> anstelle analysieren von tigress opaque predicate: https://github.com/mrphrazer/r2con2020_deobfuscation/blob/master/samples/ac3e087e43be67bdc674747c665b46c2 an Addresse: 0x491aa0

