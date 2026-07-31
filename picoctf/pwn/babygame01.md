## babygame01
### Problem
Get the flag and reach the exit.
```
Player position: 4 4
End tile position: 29 89
Player has flag: 0
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
....@.....................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
..........................................................................................
.........................................................................................X
```

### Analisis File
Binary file ialah:
```
baba@fedora:~/Documents/picoctf/babygame01$ file ./game 
./game: ELF 32-bit LSB executable, Intel i386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=02a3bb43121b1f6fbc2ab9154ab38a9427e19149, for GNU/Linux 3.2.0, not stripped
```
Security file:
```
baba@fedora:~/Documents/picoctf/babygame01$ checksec --file=./game
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   61 Symbols	  No	0		2		./game
```

### Analisis Kode
Hasil decompile menggunakan ghidra
```
undefined4 main(int argc)
{
    char cVar1;
    undefined4 uVar2;
    int32_t unaff_EBX;
    int32_t in_GS_OFFSET;
    int32_t var_aach;
    int32_t var_aa8h;
    int32_t var_aa4h;
    int32_t var_aa0h;
    int32_t var_14h;
    int32_t var_10h;
    
    var_10h = (int32_t)&argc;
    __x86.get_pc_thunk.bx();
    var_14h = *(int32_t *)(in_GS_OFFSET + 0x14);
    init_player((int32_t)&var_aach);
    init_map((int32_t)&var_aa0h, (int32_t)&var_aach);
    print_map((int32_t)&var_aa0h, (int32_t)&var_aach);
    signal(2, unaff_EBX + -0x568);
    do {
        do {
            cVar1 = getchar();
            move_player((int32_t)&var_aach, (int32_t)cVar1, (int32_t)&var_aa0h);
            print_map((int32_t)&var_aa0h, (int32_t)&var_aach);
        } while (var_aach != 0x1d);
    } while (var_aa8h != 0x59);
    puts(unaff_EBX + 0x90a);
    if ((char)var_aa4h != '\0') {
        puts(unaff_EBX + 0x913);
        win();
        fflush(**(undefined4 **)(unaff_EBX + 0x287e));
    }
    uVar2 = 0;
    if (var_14h != *(int32_t *)(in_GS_OFFSET + 0x14)) {
        uVar2 = __stack_chk_fail_local();
    }
    return uVar2;
}
```

yang mana looping game akan terlepas jika var_aach bernilai 0x1d dan var_aa8h ialah 0x59, yang mana kedua itu adalah variable koordinat. Jika sudah sesuai akan mengkomparasikan var_aa4h dengan 0, jika bukan maka kita mendapatkan flagnya.
```
void move_player(int32_t arg_4h, int32_t arg_8h, int32_t arg_ch)
{
    char cVar1;
    undefined uVar2;
    int32_t unaff_EBX;
    unsigned long var_10h;
    int32_t var_ch;
    
    __x86.get_pc_thunk.bx();
    cVar1 = (char)arg_8h;
    if (cVar1 == 'l') {
        uVar2 = getchar();
        *(undefined *)(unaff_EBX + 0x2ad3) = uVar2;
    }
    if (cVar1 == 'p') {
        solve_round(arg_ch, arg_4h);
    }
    *(undefined *)(*(int32_t *)arg_4h * 0x5a + arg_ch + *(int32_t *)(arg_4h + 4)) = 0x2e;
    if (cVar1 == 'w') {
        *(int32_t *)arg_4h = *(int32_t *)arg_4h + -1;
    } else if (cVar1 == 's') {
        *(int32_t *)arg_4h = *(int32_t *)arg_4h + 1;
    } else if (cVar1 == 'a') {
        *(int32_t *)(arg_4h + 4) = *(int32_t *)(arg_4h + 4) + -1;
    } else if (cVar1 == 'd') {
        *(int32_t *)(arg_4h + 4) = *(int32_t *)(arg_4h + 4) + 1;
    }
    *(undefined *)(*(int32_t *)arg_4h * 0x5a + arg_ch + *(int32_t *)(arg_4h + 4)) = *(undefined *)(unaff_EBX + 0x2ad3);
    return;
}
```
Yang mana berarti kita memiliki kontrol akan gamenya, yaitu:
* Jika kita menekan w maka nilai *(arg_4h) akan naik
* Jika kita menekan s maka nilai *(arg_4h) akan turun
* Jika kita menekan d maka nilai *(arg_4h + 4) akan naik
* Jika kita menekan a maka nilai *(arg_4h + 4) akan turun
* dengan p akan otomatis solve
* dengan l maka akan mengubah karakter kita
Ada satu hal yang perlu diperhatikan:
```
*(undefined *)(*(int32_t *)arg_4h * 0x5a + arg_ch + *(int32_t *)(arg_4h + 4)) = *(undefined *)(unaff_EBX + 0x2ad3);
```
Atau jika dibuat lebih sederhana akan menjadi:
```
arrMap[(row * 0x5a) + col] = player_tile;
```
Sehingga di sini kita memiliki arbitary write pada array yang tidak dijaga.\
Kurang lebih stack akan menjadi demikian:
||
|:--:|
|  y |
|  x |
| var_aa4h|
| arrMap |

Maka dengan begitu, player harus mundur ke (-4, 0), jangan terlalu jauh juga karena bisa merusak memory lain dan menyebabkan segfault.