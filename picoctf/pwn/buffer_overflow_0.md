# Buffer overflow 0
## Probset
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }
  
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);


  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1); 
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}

```

## Diketahui
Ada beberapa hal yang bisa kita dapatkan dari membaca file ini:
* Flag diprint di fungsi sigsegv_handler
* sisegv_handler digunakan sebagai signal SISGSEGV handler
* terdapat gets tanpa proteksi untuk buf1 yang berukuran 100
* terdapat vuln dengan inputan buf1 yang isinya strcpy ke buf2 yang berukuran 16

## Solusi
Kita bisa sengaja overflow baik itu buf1 maupun buf2 agar memantik sisegv_handler

payloadnya karakter apapun yang lebih besar dari buf2