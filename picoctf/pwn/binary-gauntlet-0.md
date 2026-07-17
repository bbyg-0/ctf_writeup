# binary-gauntlet-0
## Problem
main function:
```
Dump of assembler code for function main:
   0x000000000040090d <+0>:	push   rbp
   0x000000000040090e <+1>:	mov    rbp,rsp
   0x0000000000400911 <+4>:	sub    rsp,0x90
   0x0000000000400918 <+11>:	mov    DWORD PTR [rbp-0x84],edi
   0x000000000040091e <+17>:	mov    QWORD PTR [rbp-0x90],rsi
   0x0000000000400925 <+24>:	mov    edi,0x3e8
   0x000000000040092a <+29>:	call   0x400790 <malloc@plt>
   0x000000000040092f <+34>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000400933 <+38>:	lea    rsi,[rip+0x192]        # 0x400acc
   0x000000000040093a <+45>:	lea    rdi,[rip+0x18d]        # 0x400ace
   0x0000000000400941 <+52>:	call   0x4007c0 <fopen@plt>
   0x0000000000400946 <+57>:	mov    QWORD PTR [rbp-0x10],rax
   0x000000000040094a <+61>:	cmp    QWORD PTR [rbp-0x10],0x0
   0x000000000040094f <+66>:	jne    0x400967 <main+90>
   0x0000000000400951 <+68>:	lea    rdi,[rip+0x180]        # 0x400ad8
   0x0000000000400958 <+75>:	call   0x400730 <puts@plt>
   0x000000000040095d <+80>:	mov    edi,0x0
   0x0000000000400962 <+85>:	call   0x4007d0 <exit@plt>
   0x0000000000400967 <+90>:	mov    rax,QWORD PTR [rbp-0x10]
   0x000000000040096b <+94>:	mov    rdx,rax
   0x000000000040096e <+97>:	mov    esi,0x40
   0x0000000000400973 <+102>:	lea    rdi,[rip+0x200766]        # 0x6010e0 <flag>
   0x000000000040097a <+109>:	call   0x400760 <fgets@plt>
   0x000000000040097f <+114>:	lea    rsi,[rip+0xffffffffffffff41]        # 0x4008c7 <sigsegv_handler>
   0x0000000000400986 <+121>:	mov    edi,0xb
   0x000000000040098b <+126>:	call   0x400770 <signal@plt>
   0x0000000000400990 <+131>:	mov    eax,0x0
   0x0000000000400995 <+136>:	call   0x4007b0 <getegid@plt>
   0x000000000040099a <+141>:	mov    DWORD PTR [rbp-0x14],eax
   0x000000000040099d <+144>:	mov    edx,DWORD PTR [rbp-0x14]
   0x00000000004009a0 <+147>:	mov    ecx,DWORD PTR [rbp-0x14]
   0x00000000004009a3 <+150>:	mov    eax,DWORD PTR [rbp-0x14]
   0x00000000004009a6 <+153>:	mov    esi,ecx
   0x00000000004009a8 <+155>:	mov    edi,eax
   0x00000000004009aa <+157>:	mov    eax,0x0
   0x00000000004009af <+162>:	call   0x400740 <setresgid@plt>
   0x00000000004009b4 <+167>:	mov    rdx,QWORD PTR [rip+0x2006f5]        # 0x6010b0 <stdin@@GLIBC_2.2.5>
   0x00000000004009bb <+174>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004009bf <+178>:	mov    esi,0x3e8
   0x00000000004009c4 <+183>:	mov    rdi,rax
   0x00000000004009c7 <+186>:	call   0x400760 <fgets@plt>
   0x00000000004009cc <+191>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004009d0 <+195>:	add    rax,0x3e7
   0x00000000004009d6 <+201>:	mov    BYTE PTR [rax],0x0
   0x00000000004009d9 <+204>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004009dd <+208>:	mov    rdi,rax
   0x00000000004009e0 <+211>:	mov    eax,0x0
   0x00000000004009e5 <+216>:	call   0x400750 <printf@plt>
   0x00000000004009ea <+221>:	mov    rax,QWORD PTR [rip+0x2006af]        # 0x6010a0 <stdout@@GLIBC_2.2.5>
   0x00000000004009f1 <+228>:	mov    rdi,rax
   0x00000000004009f4 <+231>:	call   0x4007a0 <fflush@plt>
   0x00000000004009f9 <+236>:	mov    rdx,QWORD PTR [rip+0x2006b0]        # 0x6010b0 <stdin@@GLIBC_2.2.5>
   0x0000000000400a00 <+243>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400a04 <+247>:	mov    esi,0x3e8
   0x0000000000400a09 <+252>:	mov    rdi,rax
   0x0000000000400a0c <+255>:	call   0x400760 <fgets@plt>
   0x0000000000400a11 <+260>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400a15 <+264>:	add    rax,0x3e7
   0x0000000000400a1b <+270>:	mov    BYTE PTR [rax],0x0
   0x0000000000400a1e <+273>:	mov    rdx,QWORD PTR [rbp-0x8]
   0x0000000000400a22 <+277>:	lea    rax,[rbp-0x80]
   0x0000000000400a26 <+281>:	mov    rsi,rdx
   0x0000000000400a29 <+284>:	mov    rdi,rax
   0x0000000000400a2c <+287>:	call   0x400720 <strcpy@plt>
   0x0000000000400a31 <+292>:	mov    eax,0x0
   0x0000000000400a36 <+297>:	leave
   0x0000000000400a37 <+298>:	ret
End of assembler dump.

```

sigsegv_handler function: 
```
Dump of assembler code for function sigsegv_handler:
   0x00000000004008c7 <+0>:	push   rbp
   0x00000000004008c8 <+1>:	mov    rbp,rsp
   0x00000000004008cb <+4>:	sub    rsp,0x10
   0x00000000004008cf <+8>:	mov    DWORD PTR [rbp-0x4],edi
   0x00000000004008d2 <+11>:	mov    rax,QWORD PTR [rip+0x2007e7]        # 0x6010c0 <stderr@@GLIBC_2.2.5>
   0x00000000004008d9 <+18>:	lea    rdx,[rip+0x200800]        # 0x6010e0 <flag>
   0x00000000004008e0 <+25>:	lea    rsi,[rip+0x1e1]        # 0x400ac8
   0x00000000004008e7 <+32>:	mov    rdi,rax
   0x00000000004008ea <+35>:	mov    eax,0x0
   0x00000000004008ef <+40>:	call   0x400780 <fprintf@plt>
   0x00000000004008f4 <+45>:	mov    rax,QWORD PTR [rip+0x2007c5]        # 0x6010c0 <stderr@@GLIBC_2.2.5>
   0x00000000004008fb <+52>:	mov    rdi,rax
   0x00000000004008fe <+55>:	call   0x4007a0 <fflush@plt>
   0x0000000000400903 <+60>:	mov    edi,0x1
   0x0000000000400908 <+65>:	call   0x4007d0 <exit@plt>

```

## Solusi
sigsegv_handler menuliskan flagnya di stderr, jadi sengaja segmentation fault saja.