# picoCTF Writeup: DISKO 3

ディスクイメージの操作を学ぼう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文はこんな感じです。
>Can you find the flag in this disk image? This time, its not as plain as you think it is!
>このディスクイメージからflagを見つけられるかな？
>今回は君が考えるほどシンプルじゃないよ!

さて、ダウンロードしていきますか。
```
$ curl -O https://artifacts.picoctf.net/c/542/disko-3.dd.gz
```

`file`コマンドで確認します。
```
$ file disko-3.dd.gz 
disko-3.dd.gz: gzip compressed data, was "disko-3.dd", last modified: Thu Jul 17 15:06:36 2025, from Unix, original size modulo 2^32 104857600
```
`gzip`圧縮されているので、解凍します。
```
$ gzip -d disko-3.dd.gz

$ ls
disko-3.dd
```

```
$ file disko-3.dd   
disko-3.dd: DOS/MBR boot sector, code offset 0x58+2, OEM-ID "mkfs.fat", Media descriptor 0xf8, sectors/track 32, heads 8, sectors 204800 (volumes > 32 MB), FAT (32 bit), sectors/FAT 1576, serial number 0x49838d0b, unlabeled

$ fdisk -l disko-3.dd   
Disk disko-3.dd: 100 MiB, 104857600 bytes, 204800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000
```
`FAT`形式のイメージファイルですね。
`loop`デバイスとして、そのままマウントしてみます。
```
$ sudo mkdir /mnt/disk3

$ sudo mount -o loop disko-3.dd /mnt/disk3
```
OK、マウントできました。

中身をみていきます。
```
$ ll /mnt/disk3    
合計 4
drwxr-xr-x 11 root root 3584  4月  1  2025 log

$ cd /mnt/disk3/log
```
`log`ディレクトリ配下は結構多数のファイルやディレクトリがあったので、`find`で探してみます。
```
$ find . -name flag.*
./flag.gz
```
ありましたね。

```
$ file flag.gz       
flag.gz: gzip compressed data, was "flag", last modified: Thu Jul 17 15:06:36 2025, from Unix, original size modulo 2^32 53
```
こちらのファイルも`gzip -d`で解凍します。

```
$ sudo gzip -d flag.gz

$ file flag   
flag: ASCII text
```
解凍できました。テキストファイルのようです。
中をみてみます。

```
$ cat flag         
Here is your flag
picoCTF{n3v3r_z1p_2_h1d3_XXXXXXXX}
```
flagとれました。（flagはマスクしています。）
