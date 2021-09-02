---
title: WarGames Crackme Writeup
published: true
---
Today we are cracking **WarGames** from [crackmes.one](https://crackmes.one). I will use **gdb** as debugger for this binary.
# Having a look at main
So because this program actually has a defined main function, we dont have to spot it.
```asm
(gdb) disass main
Dump of assembler code for function main:
   0x0000000000401d35 <+0>:	endbr64
   0x0000000000401d39 <+4>:	push   rbp
   0x0000000000401d3a <+5>:	mov    rbp,rsp
   0x0000000000401d3d <+8>:	sub    rsp,0x40
   0x0000000000401d41 <+12>:	mov    DWORD PTR [rbp-0x34],edi
   0x0000000000401d44 <+15>:	mov    QWORD PTR [rbp-0x40],rsi
   0x0000000000401d48 <+19>:	mov    rax,QWORD PTR fs:0x28
   0x0000000000401d51 <+28>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401d55 <+32>:	xor    eax,eax
   0x0000000000401d57 <+34>:	cmp    DWORD PTR [rbp-0x34],0x2
   0x0000000000401d5b <+38>:	je     0x401d73 <main+62>
   0x0000000000401d5d <+40>:	lea    rdi,[rip+0x932a0]        # 0x495004
   0x0000000000401d64 <+47>:	call   0x412180 <puts>
   0x0000000000401d69 <+52>:	mov    eax,0x0
   0x0000000000401d6e <+57>:	jmp    0x401e95 <main+352>
   0x0000000000401d73 <+62>:	mov    rax,QWORD PTR [rbp-0x40]
   0x0000000000401d77 <+66>:	add    rax,0x8
   0x0000000000401d7b <+70>:	mov    rax,QWORD PTR [rax]
   0x0000000000401d7e <+73>:	mov    rdi,rax
   0x0000000000401d81 <+76>:	call   0x401180
   0x0000000000401d86 <+81>:	cmp    rax,0x9
   0x0000000000401d8a <+85>:	je     0x401da2 <main+109>
   0x0000000000401d8c <+87>:	lea    rdi,[rip+0x93285]        # 0x495018
   0x0000000000401d93 <+94>:	call   0x412180 <puts>
   0x0000000000401d98 <+99>:	mov    eax,0x0
   0x0000000000401d9d <+104>:	jmp    0x401e95 <main+352>
   0x0000000000401da2 <+109>:	movabs rax,0x6370742377737367
   0x0000000000401dac <+119>:	mov    QWORD PTR [rbp-0x11],rax
   0x0000000000401db0 <+123>:	mov    BYTE PTR [rbp-0x9],0x7a
   0x0000000000401db4 <+127>:	mov    DWORD PTR [rbp-0x24],0x0
   0x0000000000401dbb <+134>:	mov    DWORD PTR [rbp-0x28],0x0
   0x0000000000401dc2 <+141>:	mov    edi,0x7bf
   0x0000000000401dc7 <+146>:	call   0x410630 <srandom>
   0x0000000000401dcc <+151>:	mov    QWORD PTR [rbp-0x20],0x0
   0x0000000000401dd4 <+159>:	jmp    0x401e65 <main+304>
   0x0000000000401dd9 <+164>:	call   0x410d80 <rand>
   0x0000000000401dde <+169>:	mov    ecx,eax
   0x0000000000401de0 <+171>:	movsxd rax,ecx
   0x0000000000401de3 <+174>:	imul   rax,rax,0x66666667
   0x0000000000401dea <+181>:	shr    rax,0x20
   0x0000000000401dee <+185>:	mov    edx,eax
   0x0000000000401df0 <+187>:	sar    edx,1
   0x0000000000401df2 <+189>:	mov    eax,ecx
   0x0000000000401df4 <+191>:	sar    eax,0x1f
   0x0000000000401df7 <+194>:	sub    edx,eax
   0x0000000000401df9 <+196>:	mov    eax,edx
   0x0000000000401dfb <+198>:	shl    eax,0x2
   0x0000000000401dfe <+201>:	add    eax,edx
   0x0000000000401e00 <+203>:	sub    ecx,eax
   0x0000000000401e02 <+205>:	mov    edx,ecx
   0x0000000000401e04 <+207>:	lea    eax,[rdx+0x1]
   0x0000000000401e07 <+210>:	mov    DWORD PTR [rbp-0x24],eax
   0x0000000000401e0a <+213>:	lea    rdx,[rbp-0x11]
   0x0000000000401e0e <+217>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e12 <+221>:	add    rax,rdx
   0x0000000000401e15 <+224>:	movzx  eax,BYTE PTR [rax]
   0x0000000000401e18 <+227>:	mov    edx,eax
   0x0000000000401e1a <+229>:	mov    eax,DWORD PTR [rbp-0x24]
   0x0000000000401e1d <+232>:	sub    edx,eax
   0x0000000000401e1f <+234>:	mov    eax,edx
   0x0000000000401e21 <+236>:	mov    ecx,eax
   0x0000000000401e23 <+238>:	lea    rdx,[rbp-0x11]
   0x0000000000401e27 <+242>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e2b <+246>:	add    rax,rdx
   0x0000000000401e2e <+249>:	mov    BYTE PTR [rax],cl
   0x0000000000401e30 <+251>:	lea    rdx,[rbp-0x11]
   0x0000000000401e34 <+255>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e38 <+259>:	add    rax,rdx
   0x0000000000401e3b <+262>:	movzx  edx,BYTE PTR [rax]
   0x0000000000401e3e <+265>:	mov    rax,QWORD PTR [rbp-0x40]
   0x0000000000401e42 <+269>:	add    rax,0x8
   0x0000000000401e46 <+273>:	mov    rcx,QWORD PTR [rax]
   0x0000000000401e49 <+276>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e4d <+280>:	add    rax,rcx
   0x0000000000401e50 <+283>:	movzx  eax,BYTE PTR [rax]
   0x0000000000401e53 <+286>:	cmp    dl,al
   0x0000000000401e55 <+288>:	je     0x401e60 <main+299>
   0x0000000000401e57 <+290>:	mov    DWORD PTR [rbp-0x28],0x1
   0x0000000000401e5e <+297>:	jmp    0x401e70 <main+315>
   0x0000000000401e60 <+299>:	add    QWORD PTR [rbp-0x20],0x1
   0x0000000000401e65 <+304>:	cmp    QWORD PTR [rbp-0x20],0x8
   0x0000000000401e6a <+309>:	jbe    0x401dd9 <main+164>
   0x0000000000401e70 <+315>:	cmp    DWORD PTR [rbp-0x28],0x0
   0x0000000000401e74 <+319>:	je     0x401e84 <main+335>
   0x0000000000401e76 <+321>:	lea    rdi,[rip+0x9319b]        # 0x495018
   0x0000000000401e7d <+328>:	call   0x412180 <puts>
   0x0000000000401e82 <+333>:	jmp    0x401e90 <main+347>
   0x0000000000401e84 <+335>:	lea    rdi,[rip+0x931a0]        # 0x49502b
   0x0000000000401e8b <+342>:	call   0x412180 <puts>
   0x0000000000401e90 <+347>:	mov    eax,0x0
   0x0000000000401e95 <+352>:	mov    rsi,QWORD PTR [rbp-0x8]
   0x0000000000401e99 <+356>:	xor    rsi,QWORD PTR fs:0x28
   0x0000000000401ea2 <+365>:	je     0x401ea9 <main+372>
   0x0000000000401ea4 <+367>:	call   0x44c6e0 <__stack_chk_fail_local>
   0x0000000000401ea9 <+372>:	leave
   0x0000000000401eaa <+373>:	ret
End of assembler dump.
```
Having a look at the assembly code, we see **2 functions** with **name "random"**.
But we dont actually have a function to get the seed from, so it **posiblly isnt actually random**.
We also have a **non defined function**, lets see what it does.
```
(gdb) p /d $rax
4
```
As my input was "test" it gives me the **lenght**, and then **compares** it with **0x9**.
Now we know the **password** is **9 chars long**.
# Getting the password
Remember the random functions?
They actually give the **same output** over and over again, I think it is actually a **decoy**.
Looking again at the code I can see a **for loop**, with a **compare** in it.
```asm
   0x0000000000401dd9 <+164>:	call   0x410d80 <rand>
   0x0000000000401dde <+169>:	mov    ecx,eax
   0x0000000000401de0 <+171>:	movsxd rax,ecx
   0x0000000000401de3 <+174>:	imul   rax,rax,0x66666667
   0x0000000000401dea <+181>:	shr    rax,0x20
   0x0000000000401dee <+185>:	mov    edx,eax
   0x0000000000401df0 <+187>:	sar    edx,1
   0x0000000000401df2 <+189>:	mov    eax,ecx
   0x0000000000401df4 <+191>:	sar    eax,0x1f
   0x0000000000401df7 <+194>:	sub    edx,eax
   0x0000000000401df9 <+196>:	mov    eax,edx
   0x0000000000401dfb <+198>:	shl    eax,0x2
   0x0000000000401dfe <+201>:	add    eax,edx
   0x0000000000401e00 <+203>:	sub    ecx,eax
   0x0000000000401e02 <+205>:	mov    edx,ecx
   0x0000000000401e04 <+207>:	lea    eax,[rdx+0x1]
   0x0000000000401e07 <+210>:	mov    DWORD PTR [rbp-0x24],eax
   0x0000000000401e0a <+213>:	lea    rdx,[rbp-0x11]
   0x0000000000401e0e <+217>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e12 <+221>:	add    rax,rdx
   0x0000000000401e15 <+224>:	movzx  eax,BYTE PTR [rax]
   0x0000000000401e18 <+227>:	mov    edx,eax
   0x0000000000401e1a <+229>:	mov    eax,DWORD PTR [rbp-0x24]
   0x0000000000401e1d <+232>:	sub    edx,eax
   0x0000000000401e1f <+234>:	mov    eax,edx
   0x0000000000401e21 <+236>:	mov    ecx,eax
   0x0000000000401e23 <+238>:	lea    rdx,[rbp-0x11]
   0x0000000000401e27 <+242>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e2b <+246>:	add    rax,rdx
   0x0000000000401e2e <+249>:	mov    BYTE PTR [rax],cl
   0x0000000000401e30 <+251>:	lea    rdx,[rbp-0x11]
   0x0000000000401e34 <+255>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e38 <+259>:	add    rax,rdx
   0x0000000000401e3b <+262>:	movzx  edx,BYTE PTR [rax]
   0x0000000000401e3e <+265>:	mov    rax,QWORD PTR [rbp-0x40]
   0x0000000000401e42 <+269>:	add    rax,0x8
   0x0000000000401e46 <+273>:	mov    rcx,QWORD PTR [rax]
   0x0000000000401e49 <+276>:	mov    rax,QWORD PTR [rbp-0x20]
   0x0000000000401e4d <+280>:	add    rax,rcx
   0x0000000000401e50 <+283>:	movzx  eax,BYTE PTR [rax]
   0x0000000000401e53 <+286>:	cmp    dl,al
   0x0000000000401e55 <+288>:	je     0x401e60 <main+299>
   0x0000000000401e57 <+290>:	mov    DWORD PTR [rbp-0x28],0x1
   0x0000000000401e5e <+297>:	jmp    0x401e70 <main+315>
   0x0000000000401e60 <+299>:	add    QWORD PTR [rbp-0x20],0x1
   0x0000000000401e65 <+304>:	cmp    QWORD PTR [rbp-0x20],0x8
   0x0000000000401e6a <+309>:	jbe    0x401dd9 <main+164>
```
I can se an interesting **cmp** at **0x0000000000401e53**.
Lets set a **breakpoint** and see what they are comparing.
```
(gdb) p /c $al
$151 = 52 '4'
(gdb) p /c $dl
$152 = 100 'd'
```
It is actually comparing the first char I entered, which is "**4**", with "**d**".
Becuase theres **no encryption** in that cmp instruction, we can just set a breakpoint and let the **chars flow**!     
**Solution**: *"dont play"*    
If you liked this writeup dont forget to support KRDnet!
### Author
* [s0ck37](https://github.com/Kik449)
