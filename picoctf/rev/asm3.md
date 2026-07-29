# ASM3
## Problem
```
asm3:
	<+0>:	endbr32 
	<+4>:	push   ebp
	<+5>:	mov    ebp,esp
	<+7>:	xor    eax,eax
	<+9>:	mov    ah,BYTE PTR [ebp+0x9]
	<+12>:	shl    ax,0x10
	<+16>:	sub    al,BYTE PTR [ebp+0xd]
	<+19>:	add    ah,BYTE PTR [ebp+0xf]
	<+22>:	xor    ax,WORD PTR [ebp+0x10]
	<+26>:	nop
	<+27>:	pop    ebp
	<+28>:	ret    
```
dengan parameter sebagai berikut: asm3(0xdaba8a6c,0xfa8a55d0,0xd7771dae)

## Parameter
```
DWORD PTR [ebp + 0x8] : 0xdaba8a6c
DWORD PTR [ebp + 0xc] : 0xfa8a55d0
DWORD PTR [ebp + 0x10] : 0xd7771dae
```
Jangan lupa sama little-endian

## Anatomi register
|Nama|Cakupan|
|:---|:---|
|al|0-7|
|ah|8-15|
|ax|0-15|
|eax|0-31|
|rax|0-63|

## Tracing
Dimulai dari +16 soalnya semua yang sebelum itu dah dihapus sama +12:
* ah = 0xAB
* al = 0xFA
* ax = 0xFAAB ^ 1DAE = 0xE705