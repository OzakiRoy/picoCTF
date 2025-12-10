# picoCTF Writeup: Sleuthkit Apprentice

**Linuxディスクイメージ**からファイルをサルベージする問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文
>Download this disk image and find the flag.
ディスクイメージをダウンロードしてflagを見つけてね。

ディスクイメージをダウンロードします。

解凍します。
```$ gzip -d disk.flag.img.gz```

fileコマンドでどんなファイルか確認します。
```
$ file disk.flag.img 
disk.flag.img: DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS (0x0,32,33), end-CHS (0xc,223,19), startsector 2048, 204800 sectors; partition 2 : ID=0x82, start-CHS (0xc,223,20), end-CHS (0x16,111,25), startsector 206848, 153600 sectors; partition 3 : ID=0x83, start-CHS (0x16,111,26), end-CHS (0x26,62,24), startsector 360448, 253952 sectors
```
`ID=0x83`, `ID=0x82` このパーティションIDはLinuxファイルシステムです。
`0x83`がext系、`0x82`がswapです。

次に`fdisk`コマンドを確認する。
```
$ fdisk -l disk.flag.img 
Disk disk.flag.img: 300 MiB, 314572800 bytes, 614400 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x7389e82d

Device         Boot  Start    End Sectors  Size Id Type
disk.flag.img1 *      2048 206847  204800  100M 83 Linux
disk.flag.img2      206848 360447  153600   75M 82 Linux swap / Solaris
disk.flag.img3      360448 614399  253952  124M 83 Linux
```
`fdisk` が教えてくれた "startsector" に合わせて
各パーティションを dd で切り出し、個別のファイルとして mount できます。

`disk.flag.img1`, `disk.flag.img3`をパーティション分離してマウントしてみます。

```
$ dd if=disk.flag.img of=part1.img bs=512 skip=2048 count=204800
$ dd if=disk.flag.img of=part3.img bs=512 skip=360448 count=253952
$ sudo mkdir /mnt/p1
$ sudo mkdir /mnt/p3
$ sudo mount -o loop part1.img /mnt/p1
$ sudo mount -o loop part3.img /mnt/p3
```
さて、みていきますか。
```
$ sudo ls -l /mnt/p3/root/
total 1
drwxr-xr-x 2 root root 1024 Sep 30  2021 my_folder
```
`my_folder`。。。怪しいな

```
$ sudo ls -l /mnt/p3/root/my_folder
total 1
-rw-r--r-- 1 root root 60 Sep 30  2021 flag.uni.txt
```
やはり
```
$ sudo cat /mnt/p3/root/my_folder/flag.uni.txt
picoCTF{by73_5urf3r_XXXXXXXX}
```
flagとれました。（flagはマスクしています。）
