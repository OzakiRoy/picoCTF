# picoCTF Writeup: Sleuthkit Intro

**mmls**コマンドを使ってみよう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文
>Download the disk image and use mmls on it to find the size of the Linux partition. Connect to the remote checker service to check your answer and get the flag.
Note: if you are using the webshell, download and extract the disk image into /tmp not your home directory.
Download disk image
Access checker program: nc saturn.picoctf.net 51397
>ディスクイメージをダウンロードしてmmlsを使ってLinuxパーティションのサイズを確認して。リモートチェッカーサービスに接続して答えを確認してflagを取ってね。

まずはディスクイメージをダウンロードします。
```
$ curl -O https://artifacts.picoctf.net/c/164/disk.img.gz
```

`file`コマンドで確認します。
```
$ file disk.img.gz                                                          
disk.img.gz: gzip compressed data, was "disk.img", last modified: Tue Sep 21 19:34:53 2021, from Unix, original size modulo 2^32 104857600
```

`gzip -d`で展開します。
```
$ gzip -d disk.img.gz
```

`file`、`fdisk -l`でディスクイメージを確認します。
```
$ file disk.img   
disk.img: DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS (0x0,32,33), end-CHS (0xc,190,50), startsector 2048, 202752 sectors
```
```
$ fdisk -l disk.img                                                         
Disk disk.img: 100 MiB, 104857600 bytes, 204800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5d8b75fc

Device     Boot Start    End Sectors Size Id Type
disk.img1  *     2048 204799  202752  99M 83 Linux
```

問題文通り`mmls`を使ってみます。

`mmls`とは？
>mmls - Display the partition layout of a volume system  (partition tables)
パーティションがどうなってるか見れるツール

```
$ mmls disk.img 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000204799   0000202752   Linux (0x83)
```
パーティション情報が確認できました。

では、リモートチェッカーサービスに接続してみますか。
```
$ nc saturn.picoctf.net 51397
What is the size of the Linux partition in the given disk image?
Length in sectors:
```

`Linux`パーティションのサイズは`mmls`結果の`Length`に書いてありますね。
```
$ nc saturn.picoctf.net 51397
What is the size of the Linux partition in the given disk image?
Length in sectors: 202752
202752
Great work!
picoCTF{mm15_f7w!}
```
flag取れました。