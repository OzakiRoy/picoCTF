# picoCTF Writeup: Operation Orchid

**ディスクイメージ**や**復号**の技術を知ろう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文
>Download this disk image and find the flag.
ディスクイメージをダウンロードしてflagを見つけて。

ダウンロードします。
```
$ curl -O https://artifacts.picoctf.net/c/212/disk.flag.img.gz
```

`file`コマンドで確認します。
```
$ file disk.flag.img.gz                                            
disk.flag.img.gz: gzip compressed data, was "disk.flag.img", last modified: Thu Mar 16 01:56:49 2023, from Unix, original size modulo 2^32 419430400
```
`gzip -d`で解凍します。
```
$ gzip -d disk.flag.img.gz

$ ls
disk.flag.img

$ file disk.flag.img   
disk.flag.img: DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS (0x0,32,33), end-CHS (0xc,223,19), startsector 2048, 204800 sectors; partition 2 : ID=0x82, start-CHS (0xc,223,20), end-CHS (0x19,159,6), startsector 206848, 204800 sectors; partition 3 : ID=0x83, start-CHS (0x19,159,7), end-CHS (0x32,253,11), startsector 411648, 407552 sectors
```
ディスクイメージですね。

`fdisk -l`でもう少し詳しくみます。
```
$ fdisk -l disk.flag.img 
Disk disk.flag.img: 400 MiB, 419430400 bytes, 819200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xb11a86e3

Device         Boot  Start    End Sectors  Size Id Type
disk.flag.img1 *      2048 206847  204800  100M 83 Linux
disk.flag.img2      206848 411647  204800  100M 82 Linux swap / Solaris
disk.flag.img3      411648 819199  407552  199M 83 Linux
```

`dd`で3つ目のディスクイメージを取り出します。
```
$ dd if=disk.flag.img of=part3.dd bs=512 skip=411648 count=407552
407552+0 records in
407552+0 records out
208666624 bytes (209 MB, 199 MiB) copied, 1.86791 s, 112 MB/s
                              
$ ls
disk.flag.img  part3.dd

$ file part3.dd         
part3.dd: Linux rev 1.0 ext4 filesystem data, UUID=7b688671-b1b5-4e64-830d-36ebc2d4259e (extents) (64bit) (large files) (huge files)
```

マウントします。
```
$ sudo mkdir /mnt/orchid

$ sudo mount -o loop part3.dd /mnt/orchid

$ ls -l /mnt/orchid            
合計 39
drwxr-xr-x  2 root root  3072 10月  7  2021 bin
drwxr-xr-x  2 root root  1024 10月  7  2021 boot
drwxr-xr-x  2 root root  1024 10月  7  2021 dev
drwxr-xr-x 27 root root  3072 10月  7  2021 etc
drwxr-xr-x  2 root root  1024 10月  7  2021 home
drwxr-xr-x  9 root root  1024 10月  7  2021 lib
drwx------  2 root root 12288 10月  7  2021 lost+found
drwxr-xr-x  5 root root  1024 10月  7  2021 media
drwxr-xr-x  2 root root  1024 10月  7  2021 mnt
drwxr-xr-x  2 root root  1024 10月  7  2021 opt
drwxr-xr-x  2 root root  1024 10月  7  2021 proc
drwx------  2 root root  1024 10月  7  2021 root
drwxr-xr-x  2 root root  1024 10月  7  2021 run
drwxr-xr-x  2 root root  5120 10月  7  2021 sbin
drwxr-xr-x  2 root root  1024 10月  7  2021 srv
drwxr-xr-x  2 root root  1024 10月  7  2021 swap
drwxr-xr-x  2 root root  1024 10月  7  2021 sys
drwxrwxrwt  4 root root  1024 10月  7  2021 tmp
drwxr-xr-x  8 root root  1024 10月  7  2021 usr
drwxr-xr-x 11 root root  1024 10月  7  2021 var
```
OK。マウントできました。

とりあえず、`root`ディレクトリを探してみます。
```
$ sudo ls -la /mnt/orchid/root
合計 4
drwx------  2 root root 1024 10月  7  2021 .
drwxr-xr-x 22 root root 1024 10月  7  2021 ..
-rw-------  1 root root  202 10月  7  2021 .ash_history
-rw-r--r--  1 root root   64 10月  7  2021 flag.txt.enc
```
ビンゴですね。

`flag.txt.enc`を確認します。
```
$ sudo cat /mnt/orchid/root/flag.txt.enc
Salted__0��!�-6V����0��U��l��&�:�pj_1�0�|�h
                                           �Ȥ7� ���؎$�'% 
```
ありゃ？暗号化されていますね。

なんか怪しい、`.ash_history`もみてみます。
```
$ sudo cat /mnt/orchid/root/.ash_history
touch flag.txt
nano flag.txt 
apk get nano
apk --help
apk add nano
nano flag.txt 
openssl
openssl aes256 -salt -in flag.txt -out flag.txt.enc -k unbreakablepassword1234567
shred -u flag.txt
ls -al
halt
```
なるほど。
`openssl aes256 -salt -in flag.txt -out flag.txt.enc -k unbreakablepassword1234567`このコマンドで暗号化したようです。

復号コマンドを調べます。
```
$ sudo openssl aes256 -d -in /mnt/orchid/root/flag.txt.enc -out flag.txt -k unbreakablepassword1234567
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bad decrypt
4087E71C047F0000:error:1C800064:Provider routines:ossl_cipher_unpadblock:bad decrypt:../providers/implementations/ciphers/ciphercommon_block.c:107:

$ ls                     
disk.flag.img  flag.txt  part3.dd
```

色々エラーは出ていますが、`flag.txt`は復元できたみたいです。

```               
$ cat flag.txt
picoCTF{h4un71ng_p457_XXXXXXXX}
```
flagとれました。（flagはマスクしています。）