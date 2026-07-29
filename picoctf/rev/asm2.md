# ASM2
## Problem
```
asm2:
	<+0>:	endbr32 
	<+4>:	push   ebp
	<+5>:	mov    ebp,esp
	<+7>:	sub    esp,0x10
	<+10>:	mov    eax,DWORD PTR [ebp+0xc]
	<+13>:	mov    DWORD PTR [ebp-0x4],eax
	<+16>:	mov    eax,DWORD PTR [ebp+0x8]
	<+19>:	mov    DWORD PTR [ebp-0x8],eax
	<+22>:	jmp    0x11d0 <asm2+35>
	<+24>:	add    DWORD PTR [ebp-0x4],0x1
	<+28>:	add    DWORD PTR [ebp-0x8],0xcb
	<+35>:	cmp    DWORD PTR [ebp-0x8],0xd72d
	<+42>:	jle    0x11c5 <asm2+24>
	<+44>:	mov    eax,DWORD PTR [ebp-0x4]
	<+47>:	leave  
	<+48>:	ret    

```
Hasil ret dengan cara pemanggilan sebagai berikut asm2(0xf,0x17)

## Memperjelas yang tidak jelas
Kode assembly bisa ditulis ulang menjadi sebagai berikut:
```
asm2:
	<+0>:	endbr32 
	<+4>:	push   ebp
	<+5>:	mov    ebp,esp
	<+7>:	sub    esp, 0x10

	<+10>:	mov    eax, PARAM2	;eax = 0x17
	<+13>:	mov    VAR1, eax	;VAR1= 0x17
	<+16>:	mov    eax, PARAM1	;eax = 0xf
	<+19>:	mov    VAR2, eax	;VAR2 = 0xf

	<+22>:	jmp    0x11d0 <asm2+35>

	<+24>:	add    VAR1, 0x1
	<+28>:	add    VAR2, 0xcb

	<+35>:	cmp    VAR2, 0xd72d	;
	<+42>:	jle    0x11c5 <asm2+24>
	<+44>:	mov    eax, VAR1
	<+47>:	leave  
	<+48>:	ret    
```
Atau jika kurang masuk bisa ditulis ulang ke kode C sebagai berikut:
```
int asm2(int param1, int param2){
	int var1, var2;

	var1 = param2;
	var2 = param1;

	while(var2 <= 0xd72d){
		var1 += 0x1;
		var2 += 0xcb;
	}

	return var1;
}
```
Intinya:
* parameter ke2 akan dicek dengan 0xd72d, jika lebih kecil maka parameter ke2 akan ditambah dengan 0xcb dan parameter ke1 akan ditambah dengan 0x1
* Jika sudah tidak lebih kecil atau sama dengan 0xd72d, maka fungsi akan mengembalikan nilai parameter ke1