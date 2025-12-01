# picoCTF Writeup: DISKO 2

ディスクイメージの構造や制御技術を知ろう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文はこんな感じです。
>このディスクイメージからflagを見つけられるかな？
> 正しいのはLinuxです。
> 一歩間違えばすべてが消え去ります。

問題文からは、特にわかることはないですね。

では、問題のファイルをダウンロードします。
```
ozaki@NucBoxM5PLUS:/tmp$ curl -O https://artifacts.picoctf.net/c/541/disko-2.dd.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 15.8M  100 15.8M    0     0  4401k      0  0:00:03  0:00:03 --:--:-- 4400k
ozaki@NucBoxM5PLUS:/tmp$ ls -l | grep disko-2.dd.gz 
-rw-r--r-- 1 ozaki ozaki 16619340 Dec  1 13:40 disko-2.dd.gz
ozaki@NucBoxM5PLUS:/tmp$ gzip -d disko-2.dd.gz 
ozaki@NucBoxM5PLUS:/tmp$ ls -l | grep disko-2.dd
-rw-r--r-- 1 ozaki ozaki 104857600 Dec  1 13:40 disko-2.dd
```

fileコマンドでなんのファイルか確認します。
```
ozaki@NucBoxM5PLUS:/tmp$ file disko-2.dd 
disko-2.dd: DOS/MBR boot sector; partition 1 : ID=0x83, start-CHS (0x0,32,33), end-CHS (0x3,80,13), startsector 2048, 51200 sectors; partition 2 : ID=0xb, start-CHS (0x3,80,14), end-CHS (0x7,100,29), startsector 53248, 65536 sectors
```

ddファイルのツールって、どんなのがあるかググります。
`hexdump`を使うと中身を見れるらしいです。
```
ozaki@NucBoxM5PLUS:/tmp$ hexdump -C disko-2.dd | head -n20
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001b0  00 00 00 00 00 00 00 00  ee ea f8 8e 00 00 00 20  |............... |
000001c0  21 00 83 50 0d 03 00 08  00 00 00 c8 00 00 00 50  |!..P...........P|
000001d0  0e 03 0b 64 1d 07 00 d0  00 00 00 00 01 00 00 00  |...d............|
000001e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 55 aa  |..............U.|
00000200  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00002c00  70 69 63 6f 43 54 46 7b  34 5f 50 34 52 74 5f 31  |picoCTF{4_P4Rt_1|
00002c10  74 5f 69 35 5f 35 64 37  30 64 35 31 35 7d 00 00  |t_i5_5d70d515}..|
00002c20  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```
あれ？flag見えてない？
`picoCTF{4_P4Rt_1t_i5_5d70d515}`
を提出してみましたが、残念ながら不正解。

このディスクイメージをもう少し調べてみましょうか。
```
ozaki@NucBoxM5PLUS:/tmp$ fdisk -l disko-2.dd 
Disk disko-2.dd: 100 MiB, 104857600 bytes, 204800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8ef8eaee

Device      Boot Start    End Sectors Size Id Type
disko-2.dd1       2048  53247   51200  25M 83 Linux
disko-2.dd2      53248 118783   65536  32M  b W95 FAT32
```

disko-2.dd1(**Linux**)の方をマウントして中身を見ればflagが現れるって感じかな？

<span style="color: gray;">*ここでマウントにこだわった私はほぼ半日溶かしました・・・*</span>

ヒントに書いてある通り
>How can you extract/isolate a partition?

Linuxのイメージだけ取り出せばよさそうです。

```
ozaki@NucBoxM5PLUS:/tmp$ dd if=disko-2.dd of=part1.dd bs=512 skip=2048 count=51200
51200+0 records in
51200+0 records out
26214400 bytes (26 MB, 25 MiB) copied, 0.172645 s, 152 MB/s
```

|command引数|意味|
|-|-|
|if=disko-2.dd|input file|
|of=part1.dd|output file|
|bs=512|読み取り単位 512Bytes/回|
|skip=2048|読み取り開始位置|
|count=51200|5120セクタ分読み取る|


```
ozaki@NucBoxM5PLUS:/tmp$ file part1.dd 
part1.dd: Linux rev 1.0 ext4 filesystem data, UUID=9e5121c2-9728-4271-a07c-d74aabdc093b (errors) (extents) (64bit) (large files) (huge files)
```

これでLinuxのイメージだけ取り出せました。

イメージファイルにstringsコマンドを実行してみます。
flag出てきました。(一応flagはマスクしておきます。)

```
ozaki@NucBoxM5PLUS:/tmp$ strings part1.dd | grep 'picoCTF{' | sort -u
picoCTF{4_P4Rt_1t_i5_XXXXXXXX}
```

もちろん、hexdumpでもflag出ますね。
```
ozaki@NucBoxM5PLUS:/tmp$ hexdump -C part1.dd | grep -A1 picoCTF
00039ab0  70 69 63 6f 43 54 46 7b  34 5f 50 34 52 74 5f 31  |picoCTF{4_P4Rt_1|
00039ac0  74 5f 69 35 5f 30 35 35  64 64 31 37 35 7d 00 00  |t_i5_XXXXXXXX}..|
--
0102ceb0  70 69 63 6f 43 54 46 7b  34 5f 50 34 52 74 5f 31  |picoCTF{4_P4Rt_1|
0102cec0  74 5f 69 35 5f 30 35 35  64 64 31 37 35 7d 00 00  |t_i5_XXXXXXXX}..|
```
