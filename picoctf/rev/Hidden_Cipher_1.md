# Hidden_Cipher_1
## Problem
```
baba@fedora:~/Documents/picoctf/Hidden_Cipher_1$ nc candy-mountain.picoctf.net 50721
Here your encrypted flag:
235a201d702015483b1d412b265d3313501f0c072d135f0d2002302d044c3003264503402e

```
## Solve
```
baba@fedora:~/Documents/picoctf/Hidden_Cipher_1$ # Check file type
file ./hiddencipher

# Check if it's a valid ELF
readelf -h ./hiddencipher

# List all sections
readelf -S ./hiddencipher

# Try objdump with all sections
objdump -D -M intel ./hiddencipher | head -50

# Or use a hexdump to see if there's any data
xxd ./hiddencipher | head -20
./hiddencipher: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), statically linked, no section header
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x5b18
  Start of program headers:          64 (bytes into file)
  Start of section headers:          0 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         3
  Size of section headers:           0 (bytes)
  Number of section headers:         0
  Section header string table index: 0

There are no sections in this file.

./hiddencipher:     file format elf64-x86-64

00000000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
00000010: 0300 3e00 0100 0000 185b 0000 0000 0000  ..>......[......
00000020: 4000 0000 0000 0000 0000 0000 0000 0000  @...............
00000030: 0000 0000 4000 3800 0300 0000 0000 0000  ....@.8.........
00000040: 0100 0000 0600 0000 0000 0000 0000 0000  ................
00000050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000060: 0010 0000 0000 0000 1840 0000 0000 0000  .........@......
00000070: 0010 0000 0000 0000 0100 0000 0500 0000  ................
00000080: 0000 0000 0000 0000 0050 0000 0000 0000  .........P......
00000090: 0050 0000 0000 0000 fd15 0000 0000 0000  .P..............
000000a0: fd15 0000 0000 0000 0010 0000 0000 0000  ................
000000b0: 51e5 7464 0600 0000 0000 0000 0000 0000  Q.td............
000000c0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000e0: 1000 0000 0000 0000 b4fd 70eb 5550 5821  ..........p.UPX!
000000f0: f00a 0e16 0000 0000 b840 0000 a810 0000  .........@......
00000100: 5003 0000 eb00 0000 0200 0000 f6fb 21ff  P.............!.
00000110: 7f45 4c46 0201 0100 0300 3e00 0dc0 110f  .ELF......>.....
00000120: 73c9 0e76 4017 b838 2213 0c05 acfb 6e0e  s..v@..8".....n.
00000130: 0520 001f 0006 0f04 27c9 2ee4 9107 1003  . ......'.......
```

Dari output ini kita bisa lihat bahwasannya executeable telah distrip menggunakan upx, gunakan command di bawah untuk membuatnya normal kembali
```
upx -d ./hiddencipher
```
Setelah itu kita bisa decompile binary yang hasilnya menjadi demikian:
```

// WARNING: [rz-ghidra] Detected overlap for variable var_2ch
// WARNING: [rz-ghidra] Detected overlap for variable var_2dh

undefined8 main(void)
{
    int64_t iVar1;
    undefined8 uVar2;
    int64_t iVar3;
    int64_t iVar4;
    int32_t var_2ch;
    FILE *stream;
    long nmemb;
    void *ptr;
    int64_t var_10h;
    
    iVar1 = fopen("flag.txt", data.00002004);
    if (iVar1 == 0) {
        perror("[!] Failed to open flag.txt");
        uVar2 = 1;
    } else {
        fseek(iVar1, 0, 2);
        iVar3 = ftell(iVar1);
        rewind(iVar1);
        iVar4 = malloc(iVar3 + 1);
        if (iVar4 == 0) {
            puts("[!] Memory allocation error.");
            fclose(iVar1);
            uVar2 = 1;
        } else {
            fread(iVar4, 1, iVar3, iVar1);
            fclose(iVar1);
            *(undefined *)(iVar4 + iVar3) = 0;
            iVar1 = get_secret();
            puts("Here your encrypted flag:");
            for (var_2ch = 0; var_2ch < iVar3; var_2ch = var_2ch + 1) {
                printf("%02x", *(uint8_t *)(iVar1 + var_2ch % 6) ^ *(uint8_t *)(iVar4 + var_2ch));
            }
            putchar(10);
            free(iVar4);
            uVar2 = 0;
        }
    }
    return uVar2;
}

```
## solve.py
```
import binascii

def decrypt_direct(encrypted_hex):
    """
    Direct decryption using the key derived from the known pairs
    """
    # From the known pairs, the key is: 0x23, 0x5a, 0x20, 0x1d, 0x70, 0x20
    # Let's verify this:
    # 'p' (0x70) ^ 0x23 = 0x53 (which is 'S' but encrypted hex starts with 23)
    # Wait, let's recalculate properly
    
    # The first byte of ciphertext is 0x23, first byte of plaintext is 'p' (0x70)
    # 0x23 ^ 0x70 = 0x53, so key[0] = 0x53
    # But wait, we need to check both pairs to be sure
    
    # Let's derive the key from the first pair:
    plaintext = b"picoCTF{yadayada}"
    ciphertext = bytes.fromhex("235a201d702015483a1357152a5227134e7e")
    
    key = bytearray(6)
    for i in range(6):
        key[i] = plaintext[i] ^ ciphertext[i]
    
    print(f"Key derived: {key.hex()}")  # Should print: 537c6d7e405a or something
    
    # Now decrypt the provided ciphertext
    encrypted_bytes = bytes.fromhex(encrypted_hex)
    decrypted = bytearray()
    for i, byte in enumerate(encrypted_bytes):
        decrypted.append(byte ^ key[i % 6])
    
    return decrypted.decode('utf-8', errors='ignore')

# Example usage
if __name__ == "__main__":
    # Test with the known ciphertexts
    test_ciphers = [
        "235a201d702015483a1357152a5227134e7e",
        "235a201d70201548251358110c552f135409",
        "235a201d702015483b1d412b265d3313501f0c072d135f0d2002302d044c3003264503402e"
    ]
    
    for ciphertext in test_ciphers:
        result = decrypt_direct(ciphertext)
        print(f"Ciphertext: {ciphertext}")
        print(f"Result: {result}\n")

```

Dengan output
```
baba@fedora:~/Documents/picoctf/Hidden_Cipher_1$ python3 solve.py 
Key derived: 533343723374
Ciphertext: 235a201d702015483a1357152a5227134e7e
Result: picoCTF{yadayada}


Key derived: 533343723374
Ciphertext: 235a201d70201548251358110c552f135409
Result: picoCTF{fake_flag}

Key derived: 533343723374
Ciphertext: 235a201d702015483b1d412b265d3313501f0c072d135f0d2002302d044c3003264503402e
Result: picoCTF{xor_unpack_4nalys1s_78c0e704}


```