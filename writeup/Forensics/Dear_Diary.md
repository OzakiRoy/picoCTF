# picoCTF Writeup: Dear Diary

ディスクイメージから変更を読み取ろう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題
>If you can find the flag on this disk image, we can close the case for good!
>Download the disk image here.
>このディスク イメージでフラグが見つかった場合は、ケースを完全に閉じることができます。
>ここからディスクイメージをダのではなドしてください。

ダウンロードします。
```
$ curl -O https://artifacts.picoctf.net/c_titan/63/disk.flag.img.gz
```

`file`コマンドで確認します。
```
$ file disk.flag.img.gz 
disk.flag.img.gz: gzip compressed data, was "disk.flag.img", last modified: Sat Feb 17 22:59:04 2024, from Unix, original size modulo 2^32 1073741824
```

`gzip -d`で展開します。
```
$ gzip -d disk.flag.img.gz
```

`file`コマンドと`fdisl -l`コマンドで確認します。
```
$ file disk.flag.img    
disk.flag.img: DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS (0x0,32,33), end-CHS (0x26,94,56), startsector 2048, 614400 sectors; partition 2 : ID=0x82, start-CHS (0x26,94,57), end-CHS (0x47,1,58), startsector 616448, 524288 sectors; partition 3 : ID=0x83, start-CHS (0x47,1,59), end-CHS (0x82,138,8), startsector 1140736, 956416 sectors
```
```
$ fdisk -l disk.flag.img 
Disk disk.flag.img: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x6062d30a

Device         Boot   Start     End Sectors  Size Id Type
disk.flag.img1 *       2048  616447  614400  300M 83 Linux
disk.flag.img2       616448 1140735  524288  256M 82 Linux swap / Solaris
disk.flag.img3      1140736 2097151  956416  467M 83 Linux
```

3番目のディスクイメージだけ取り出します。
（1番目はVM関連、2番目はswap領域、3番目がrootファイルシステム）
```
$ dd if=disk.flag.img of=part3.img bs=512 skip=1140736 count=956416
956416+0 records in
956416+0 records out
489684992 bytes (490 MB, 467 MiB) copied, 4.37481 s, 112 MB/s
```
取り出せました。

さて、`mnt`してみます。
```
$ sudo mount -o loop part3.img /mnt/img3
mount: /mnt/img3: cannot mount; probably corrupted filesystem on /dev/loop0.
       dmesg(1) may have more information after failed mount system call.
```

あれ？できない。`probably corrupted filesystem`と言われています。

`file`コマンドでみてみます。
```
$ file part3.img    
part3.img: Linux rev 1.0 ext4 filesystem data, UUID=5f123377-8fc2-4bdc-a5e4-de3e08835caf (needs journal recovery) (extents) (64bit) (large files) (huge files)
```
よく見ると`needs journal recovery`とありますね。
つまり、**直前の書き込みが正常に完了しなかった**恐れがあります。

一旦、`journal`を無視して読み取り専用で`mount -o loop,ro,noload`してみます。
```
$ sudo mount -o loop,ro,noload part3.img /mnt/img3
```
```
$ sudo ls -la /mnt/img3     
合計 45
drwxr-xr-x 22 root root  1024 12月  9  2023 .
drwxr-xr-x  4 root root  4096 12月 23 09:52 ..
drwxr-xr-x  2 root root  3072 12月  9  2023 bin
drwxr-xr-x  2 root root  1024 12月  9  2023 boot
drwxr-xr-x  2 root root  1024 12月  9  2023 dev
drwxr-xr-x 31 root root  3072  2月 18  2024 etc
drwxr-xr-x  2 root root  1024 12月  9  2023 home
drwxr-xr-x 10 root root  1024 12月  9  2023 lib
drwx------  2 root root 12288 12月  9  2023 lost+found
drwxr-xr-x  5 root root  1024 12月  9  2023 media
drwxr-xr-x  2 root root  1024 12月  9  2023 mnt
drwxr-xr-x  2 root root  1024 12月  9  2023 opt
drwxr-xr-x  2 root root  1024 12月  9  2023 proc
drwx------  3 root root  1024  2月 18  2024 root
drwxr-xr-x  2 root root  1024 12月  9  2023 run
drwxr-xr-x  2 root root  6144 12月  9  2023 sbin
drwxr-xr-x  2 root root  1024 12月  9  2023 srv
drwxr-xr-x  2 root root  1024 12月  9  2023 swap
drwxr-xr-x  2 root root  1024 12月  9  2023 sys
drwxrwxrwt  2 root root  1024 12月  9  2023 tmp
drwxr-xr-x  8 root root  1024 12月  9  2023 usr
drwxr-xr-x 11 root root  1024 12月  9  2023 var

```
`mnt`できました。

さて、いつも通り`root`ディレクトリからみていきます。
```
$ sudo ls -la /mnt/img3/root
合計 4
drwx------  3 root root 1024  2月 18  2024 .
drwxr-xr-x 22 root root 1024 12月  9  2023 ..
-rw-------  1 root root   27  2月 18  2024 .ash_history
drwxr-xr-x  2 root root 1024  2月 18  2024 secret-secrets
```
なにやら怪しいファイルとディレクトリがあります。
```
$ sudo cat /mnt/img3/root/.ash_history
ls -al ..
./force-wait.sh

$ sudo ls -la /mnt/img3/root/secret-secrets
合計 3
drwxr-xr-x 2 root root 1024  2月 18  2024 .
drwx------ 3 root root 1024  2月 18  2024 ..
-rwxr-xr-x 1 root root   21  2月 18  2024 force-wait.sh
-rw-r--r-- 1 root root    0  2月 18  2024 innocuous-file.txt
-rw-r--r-- 1 root root    0  2月 18  2024 its-all-in-the-name

$ sudo cat /mnt/img3/root/secret-secrets/force-wait.sh
#!/bin/ash

sleep 10

$ sudo cat /mnt/img3/root/secret-secrets/innocuous-file.txt

$ sudo cat /mnt/img3/root/secret-secrets/its-all-in-the-name

```
色々怪しいのですが、flagにつながりそうな点は特にありません。

他のディレクトリも`find`、`grep`で調べましたが、何も出ません。
```
$ sudo grep -r pico /mnt/img3/

$ sudo find /mnt/img3/ -name *flag*

```

ここでしばらく悩みましたが、
`mount`時のエラーであったように、**直前の書き込みが失敗している**ところにflagがあるのではないかと思いました。

現在の状態を`mnt`してみるのでなく、ディスクイメージ側から履歴的なものが見れないかと。

`fls`というコマンドがあります。
`fls - List file and directory names in a disk image.`
ディスクイメージのファイルやディレクトリ名を列挙するコマンドです。`inode`が確認できます。
```
$ fls -i raw -f ext4 part3.img              
d/d 32513:      home
d/d 11: lost+found
d/d 32385:      boot
d/d 64769:      etc
d/d 32386:      proc
d/d 13: dev
d/d 32387:      tmp
d/d 14: lib
d/d 32388:      var
d/d 21: usr
d/d 32393:      bin
d/d 32395:      sbin
d/d 32539:      media
d/d 203:        mnt
d/d 32543:      opt
d/d 204:        root
d/d 32544:      run
d/d 205:        srv
d/d 32545:      sys
d/d 32530:      swap
V/V 119417:     $OrphanFiles
```
上記コマンドでは、ext4ファイルシステムのルートディレクトリ配下を列挙していますが、`inode`に**8**を指定(詳細は[リンク](https://www.infradead.org/~mchehab/rst_conversion/filesystems/ext4/overview.html#special-inodes))することで、`journal`を指定できるので、リネームされてしまった履歴も見ることができます。（出力長いので`tail`）
```
$ fls -i raw -f ext4 part3.img 8 | tail -30
-/- * 0:        �^�e�^�e
r/r 1845:       original-filename
r/r 1845:       pic
-/- * 0:        �^�e�^�e
r/r 1845:       oCT
-/- * 0:        �^�e�^�e
-/- 18327:      d.��d.���?Yѱzse����
r/r 1845:       F{1
-/- * 0:        ^^�e�^�e
-/- 46451:      $?��$?��4q�<q^�e,���
r/r 1845:       _53
-/- * 0:        D^�e�^�e
r/r 1845:       3_n
-/- * 0:        ]^�e�^�e
-/- 13793:      ��w6��w6�?Yѱzse����
-/- 3215:       |���|�����w6q^�e,���
r/r 1845:       4m3
-/- * 0:        v^�e�^�e
r/r 1845:       5_8
-/- * 0:        �^�e�^�e
r/r 1845:       0d2
-/- * 0:        �^�e�^�e
r/r 1845:       4b3
-/- * 0:        �^�e�^�e
r/r 1845:       0}
-/- * 0:        �^�e�^�e
r/r 1845:       its-all-in-the-name
-/- * 0:        ^^�e�^�e
-/- 3016:       h*.
-/- 61306:      ������������*^�e�
```
おわかりいただけただろうか？
`original-filename`という`inode: 1845`のファイルが、いくつかの変更を経て、`its-all-in-the-name`になっていることに。

`1845`で`grep`してみますね。
```
$ fls -i raw -f ext4 part3.img 8 | grep 1845                                
r/r 1845:       original-filename
r/r 1845:       pic
r/r 1845:       oCT
r/r 1845:       F{1
r/r 1845:       _53
r/r 1845:       3_n
r/r 1845:       4m3
r/r 1845:       5_8
r/r 1845:       0d2
r/r 1845:       4b3
r/r 1845:       0}
r/r 1845:       its-all-in-the-name
grep: (標準入力): binary file matches
```
もうおわかりでしょう。
```
$ fls -i raw -f ext4 part3.img 8 | grep 1845 | awk '{print $3}' | tr -d '\n'   
grep: (標準入力): binary file matches
original-filenamepicoCTF{1_533_n4m35_80d24b30}its-all-in-the-name
```
flagとれました。

## 学び
`ext4` では `inode 8` が `journal inode` として予約されている。
https://www.infradead.org/~mchehab/rst_conversion/filesystems/ext4/overview.html#special-inodes
`fls` で `inode 8` を指定することで、ファイルシステムのジャーナル領域に残存するディレクトリエントリ断片を確認できた。