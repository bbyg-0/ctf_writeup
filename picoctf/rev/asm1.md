# ASM1
## Problem
```
asm1:
	<+0>:	endbr32 
	<+4>:	push   ebp
	<+5>:	mov    ebp,esp
	<+7>:	cmp    DWORD PTR [ebp+0x8],0x2a7
	<+14>:	jg     0x11d3 <asm1+38>
	<+16>:	cmp    DWORD PTR [ebp+0x8],0x28
	<+20>:	jne    0x11cb <asm1+30>
	<+22>:	mov    eax,DWORD PTR [ebp+0x8]
	<+25>:	add    eax,0x15
	<+28>:	jmp    0x11ea <asm1+61>
	<+30>:	mov    eax,DWORD PTR [ebp+0x8]
	<+33>:	sub    eax,0x15
	<+36>:	jmp    0x11ea <asm1+61>
	<+38>:	cmp    DWORD PTR [ebp+0x8],0x48b
	<+45>:	jne    0x11e4 <asm1+55>
	<+47>:	mov    eax,DWORD PTR [ebp+0x8]
	<+50>:	sub    eax,0x15
	<+53>:	jmp    0x11ea <asm1+61>
	<+55>:	mov    eax,DWORD PTR [ebp+0x8]
	<+58>:	add    eax,0x15
	<+61>:	pop    ebp
	<+62>:	ret    
```
## Tracing
Tracing dengan parameter 0x3fa:
* asm1+7: komparasi antara parameter dengan 0x2a7
* asm1+14: Jika parameter lebih besar maka loncat ke 0x11d4 (TRUE)
* asm1+38: komparasi parameter dengan 0x48b
* asm1+45: jika beda maka loncar ke 0x11e4 (TRUE)
* asm1+55: Menyalin nilai parameter ke eax
* asm1+58: menambah nilai eax dengan 0x15 (0x3fa + 0x15 = 0x40F)
* pemulihan stack dan return nilai eax

Hasilnya adalah 0x40f