---
title: Revv Crackme Writeup
published: true
---
Today we are cracking **revv** from [crackmes.one](https://crackmes.one).
I will use **gdb** as debugger for this binary.
# Spotting the main function
As you can see below theres not actualy a main function, so we actually have to **find** it.
```
(gdb) disass main
No symbol table is loaded.  Use the "file" command.
```
The first thing I will do is spotting a **defined function**.  
We run the code and as we can se there is actually a message before getting the input.
```
(gdb) run
Starting program: /home/s0ck37/RE/revv
Enter the Password: testpassword
Length mismatch! :(
[Inferior 1 (process 291355) exited normally]
```
We know the main function calls **printf@plt** in the main function.  
Then we just set a break point in **printf@plt**...
```
(gdb) break printf@plt
Breakpoint 1 at 0x5555555550c0
```
Now we run the program and wait for breakpoint 1 message.
```
(gdb) run
Starting program: /home/s0ck37/RE/revv

Breakpoint 1, 0x00005555555550c0 in printf@plt ()
```
We just "**ni**" to go instruction by instruction until we are in "**??**" function, this means it is not defined.
```
(gdb) ni
0x00005555555553e9 in ?? ()
```
And now we have one instruction of the main function, I will disassembly a range of instructions with "disass **starting**,**ending**"  
I will do for example "disass 0x00005555555553e9-70,0x00005555555553e9+70"
```asm
(gdb) disass 0x00005555555553e9-70,0x00005555555553e9+70
Dump of assembler code from 0x5555555553a3 to 0x55555555542f:
   0x00005555555553a3:	add    al,ch
   0x00005555555553a5:	out    0xfc,eax
   0x00005555555553a7:	(bad)
   0x00005555555553a8:	(bad)
   0x00005555555553a9:	jmp    0x5555555553b7
   0x00005555555553ab:	lea    rdi,[rip+0xc8b]        # 0x55555555603d
   0x00005555555553b2:	call   0x555555555090 <puts@plt>
   0x00005555555553b7:	nop
   0x00005555555553b8:	leave
   0x00005555555553b9:	ret
   0x00005555555553ba:	endbr64
   0x00005555555553be:	push   rbp
   0x00005555555553bf:	mov    rbp,rsp
   0x00005555555553c2:	sub    rsp,0x3f0
   0x00005555555553c9:	mov    rax,QWORD PTR fs:0x28
   0x00005555555553d2:	mov    QWORD PTR [rbp-0x8],rax
   0x00005555555553d6:	xor    eax,eax
   0x00005555555553d8:	lea    rdi,[rip+0xc76]        # 0x555555556055
   0x00005555555553df:	mov    eax,0x0
   0x00005555555553e4:	call   0x5555555550c0 <printf@plt>
   0x00005555555553e9:	lea    rax,[rbp-0x3f0]
=> 0x00005555555553f0:	mov    rsi,rax
   0x00005555555553f3:	lea    rdi,[rip+0xc70]        # 0x55555555606a
   0x00005555555553fa:	mov    eax,0x0
   0x00005555555553ff:	call   0x5555555550d0 <__isoc99_scanf@plt>
   0x0000555555555404:	lea    rax,[rbp-0x3f0]
   0x000055555555540b:	mov    rdi,rax
   0x000055555555540e:	call   0x5555555551c9
   0x0000555555555413:	mov    eax,0x0
   0x0000555555555418:	mov    rdx,QWORD PTR [rbp-0x8]
   0x000055555555541c:	xor    rdx,QWORD PTR fs:0x28
   0x0000555555555425:	je     0x55555555542c
   0x0000555555555427:	call   0x5555555550b0 <__stack_chk_fail@plt>
   0x000055555555542c:	leave
   0x000055555555542d:	ret
   0x000055555555542e:	xchg   ax,ax
End of assembler dump.
```
And there we have, the function starts in **0x00005555555553ba** and ends in **0x000055555555542d**.
# Checking function
Looking at the assembly code we cant see nothing, but we have an interesting **call** in **0x000055555555540e**  
This function isn't defined, lets see what it does.
```asm
   0x00005555555551c9:	endbr64
   0x00005555555551cd:	push   rbp
   0x00005555555551ce:	mov    rbp,rsp
   0x00005555555551d1:	sub    rsp,0x10
   0x00005555555551d5:	mov    QWORD PTR [rbp-0x8],rdi
   0x00005555555551d9:	mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555551dd:	mov    rdi,rax
   0x00005555555551e0:	call   0x5555555550a0 <strlen@plt>
   0x00005555555551e5:	cmp    rax,0x19
   0x00005555555551e9:	jbe    0x5555555551fc
   0x00005555555551eb:	lea    rdi,[rip+0xe12]        # 0x555555556004
   0x00005555555551f2:	call   0x555555555090 <puts@plt>
   0x00005555555551f7:	jmp    0x5555555553b8
   0x00005555555551fc:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555200:	mov    rdi,rax
   0x0000555555555203:	call   0x5555555550a0 <strlen@plt>
   0x0000555555555208:	cmp    rax,0x15
   0x000055555555520c:	je     0x55555555521f
```
So first we can see it compares the length of the input with **0x15**, and if isn't equal it just exits.  
Now that we now the input must be **21 chars long** lets see what it does to check my input.
```asm
   0x000055555555520e:	lea    rdi,[rip+0xe07]        # 0x55555555601c
   0x0000555555555215:	call   0x555555555090 <puts@plt>
   0x000055555555521a:	jmp    0x5555555553b8
   0x000055555555521f:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555223:	add    rax,0x11
   0x0000555555555227:	movzx  eax,BYTE PTR [rax]
   0x000055555555522a:	cmp    al,0x31
   0x000055555555522c:	jne    0x5555555553ab
   0x0000555555555232:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555236:	movzx  eax,BYTE PTR [rax]
   0x0000555555555239:	cmp    al,0x41
   0x000055555555523b:	jne    0x5555555553ab
   0x0000555555555241:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555245:	add    rax,0x1
   0x0000555555555249:	movzx  eax,BYTE PTR [rax]
   0x000055555555524c:	cmp    al,0x43
   0x000055555555524e:	jne    0x5555555553ab
   0x0000555555555254:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555258:	add    rax,0x2
   0x000055555555525c:	movzx  eax,BYTE PTR [rax]
   0x000055555555525f:	cmp    al,0x54
   0x0000555555555261:	jne    0x5555555553ab
   0x0000555555555267:	mov    rax,QWORD PTR [rbp-0x8]
   0x000055555555526b:	add    rax,0x8
   0x000055555555526f:	movzx  eax,BYTE PTR [rax]
   0x0000555555555272:	cmp    al,0x63
   0x0000555555555274:	jne    0x5555555553ab
   0x000055555555527a:	mov    rax,QWORD PTR [rbp-0x8]
   0x000055555555527e:	add    rax,0x3
   0x0000555555555282:	movzx  eax,BYTE PTR [rax]
   0x0000555555555285:	cmp    al,0x46
   0x0000555555555287:	jne    0x5555555553ab
   0x000055555555528d:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555291:	add    rax,0xd
   0x0000555555555295:	movzx  eax,BYTE PTR [rax]
   0x0000555555555298:	cmp    al,0x76
   0x000055555555529a:	jne    0x5555555553ab
   0x00005555555552a0:	mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555552a4:	add    rax,0x5
   0x00005555555552a8:	movzx  eax,BYTE PTR [rax]
   0x00005555555552ab:	cmp    al,0x4e
   0x00005555555552ad:	jne    0x5555555553ab
   0x00005555555552b3:	mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555552b7:	add    rax,0x6
   0x00005555555552bb:	movzx  eax,BYTE PTR [rax]
   0x00005555555552be:	cmp    al,0x30
   0x00005555555552c0:	jne    0x5555555553ab
   0x00005555555552c6:	mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555552ca:	add    rax,0xb
   0x00005555555552ce:	movzx  eax,BYTE PTR [rax]
   0x00005555555552d1:	cmp    al,0x52
   0x00005555555552d3:	jne    0x5555555553ab
   0x00005555555552d9:	mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555552dd:	add    rax,0x7
   0x00005555555552e1:	movzx  eax,BYTE PTR [rax]
   0x00005555555552e4:	cmp    al,0x31
   0x00005555555552e6:	jne    0x5555555553ab
   0x00005555555552ec:	mov    rax,QWORD PTR [rbp-0x8]
   0x00005555555552f0:	add    rax,0xf
   0x00005555555552f4:	movzx  eax,BYTE PTR [rax]
   0x00005555555552f7:	cmp    al,0x72
   0x00005555555552f9:	jne    0x5555555553ab
   0x00005555555552ff:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555303:	add    rax,0x9
   0x0000555555555307:	movzx  eax,BYTE PTR [rax]
   0x000055555555530a:	cmp    al,0x65
   0x000055555555530c:	jne    0x5555555553ab
   0x0000555555555312:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555316:	add    rax,0x4
   0x000055555555531a:	movzx  eax,BYTE PTR [rax]
   0x000055555555531d:	cmp    al,0x7b
   0x000055555555531f:	jne    0x5555555553ab
   0x0000555555555325:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555329:	add    rax,0xa
   0x000055555555532d:	movzx  eax,BYTE PTR [rax]
   0x0000555555555330:	cmp    al,0x5f
   0x0000555555555332:	jne    0x5555555553ab
   0x0000555555555334:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555338:	add    rax,0xc
   0x000055555555533c:	movzx  eax,BYTE PTR [rax]
   0x000055555555533f:	cmp    al,0x33
   0x0000555555555341:	jne    0x5555555553ab
   0x0000555555555343:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555347:	add    rax,0x2
   0x000055555555534b:	movzx  eax,BYTE PTR [rax]
   0x000055555555534e:	cmp    al,0x54
   0x0000555555555350:	jne    0x5555555553ab
   0x0000555555555352:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555356:	add    rax,0xe
   0x000055555555535a:	movzx  eax,BYTE PTR [rax]
   0x000055555555535d:	cmp    al,0x33
   0x000055555555535f:	jne    0x5555555553ab
   0x0000555555555361:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555365:	add    rax,0x10
   0x0000555555555369:	movzx  eax,BYTE PTR [rax]
   0x000055555555536c:	cmp    al,0x35
   0x000055555555536e:	jne    0x5555555553ab
   0x0000555555555370:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555374:	add    rax,0x12
   0x0000555555555378:	movzx  eax,BYTE PTR [rax]
   0x000055555555537b:	cmp    al,0x5e
   0x000055555555537d:	jne    0x5555555553ab
   0x000055555555537f:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555383:	add    rax,0x13
   0x0000555555555387:	movzx  eax,BYTE PTR [rax]
   0x000055555555538a:	cmp    al,0x67
   0x000055555555538c:	jne    0x5555555553ab
   0x000055555555538e:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555555392:	add    rax,0x14
   0x0000555555555396:	movzx  eax,BYTE PTR [rax]
   0x0000555555555399:	cmp    al,0x7d
   0x000055555555539b:	jne    0x5555555553ab
   0x000055555555539d:	lea    rdi,[rip+0xc8c]        # 0x555555556030
   0x00005555555553a4:	call   0x555555555090 <puts@plt>
   0x00005555555553a9:	jmp    0x5555555553b7
   0x00005555555553ab:	lea    rdi,[rip+0xc8b]        # 0x55555555603d
   0x00005555555553b2:	call   0x555555555090 <puts@plt>
   0x00005555555553b7:	nop
   0x00005555555553b8:	leave
   0x00005555555553b9:	ret
```
So looking at that copy paste code I can see that its picking chars in a random order and **comparing** them **one by one**.  
Lets build the string!
```asm
   0x0000555555555258:	add    rax,0x2
   0x000055555555525c:	movzx  eax,BYTE PTR [rax]
   0x000055555555525f:	cmp    al,0x54
```
So the program follows this, the add is selecting the char **0x2** (2), and comparing with **0x54** ("T")  
So now we know the second char in the flag is T.  
**Remember**: the first char is **0** not **1**
```
xxTxxxxxxxxxxxxxxxxxx
```
And you just have to build it like that.     
Solution: **ACTF{N01ce_R3v3r51^g}**      
If you liked this writeup dont forget to support KRDnet!
### Author
* [s0ck37](https://github.com/Kik449) 
