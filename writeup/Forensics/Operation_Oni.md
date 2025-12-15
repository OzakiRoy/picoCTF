# picoCTF Writeup: Operation Oni

ディスクイメージの構造や制御技術を知ろう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文はこんな感じです。
>Download this disk image, find the key and log into the remote machine.
Note: if you are using the webshell, download and extract the disk image into /tmp not your home directory.
Download disk image
Remote machine: ssh -i key_file -p 51272 ctf-player@saturn.picoctf.net
>ディスクイメージをダウンロードして、鍵を見つけてリモートマシンにログインしてね。
>注意：webshell使う場合、ディスクイメージを/tmpにダウンロードして取り出してね。あなたのhomeディレクトリじゃないよ。
> ディスクイメージをダウンロードして。
>リモートマシン：sshコマンド


早速ダウンロードしていきます。
```
$ curl -O https://artifacts.picoctf.net/c/69/disk.img.gz
```
`file`コマンドで確認します。
```
$ file disk.img.gz   
disk.img.gz: gzip compressed data, was "disk.img", last modified: Wed Oct  6 14:32:01 2021, from Unix, original size modulo 2^32 241172480
```
`gzip`解凍します。
```
$ gzip -d disk.img.gz
$ ls
disk.img
```
`file`、`fdisk -l`でディスクイメージを見ていきます。
```
$ file disk.img   
disk.img: DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS (0x0,32,33), end-CHS (0xc,223,19), startsector 2048, 204800 sectors; partition 2 : ID=0x83, start-CHS (0xc,223,20), end-CHS (0x1d,81,52), startsector 206848, 264192 sectors
$ fdisk -l disk.img 
Disk disk.img: 230 MiB, 241172480 bytes, 471040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0b0051d0

Device     Boot  Start    End Sectors  Size Id Type
disk.img1  *      2048 206847  204800  100M 83 Linux
disk.img2       206848 471039  264192  129M 83 Linux
```
2つイメージがありますね。一応両方`dd`で取り出します。
```
$ dd if=disk.img of=part1.img bs=512 skip=2048 count=204800
204800+0 records in
204800+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.335462 s, 313 MB/s
                                                                                        
$ dd if=disk.img of=part2.img bs=512 skip=206848 count=264192
264192+0 records in
264192+0 records out
135266304 bytes (135 MB, 129 MiB) copied, 1.214 s, 111 MB/s
```
それぞれマウントしていきます。
```
$ sudo mkdir /mnt/oni-p1

$ sudo mkdir /mnt/oni-p2

$ sudo mount -o loop part1.img /mnt/oni-p1

$ sudo mount -o loop part2.img /mnt/oni-p2
```                                                             

中を軽くチェック。
```
┌──(venv)─(ozaki㉿kali)-[/tmp/oni]
└─$ sudo ls -l /mnt/oni-p1                  
合計 15752
-rw-r--r-- 1 root root 3632749  8月 27  2021 System.map-virt
lrwxrwxrwx 1 root root       1 10月  6  2021 boot -> .
-rw-r--r-- 1 root root  123781  8月 27  2021 config-virt
-rw-r--r-- 1 root root     398 10月  6  2021 extlinux.conf
-rw------- 1 root root 5706882 10月  6  2021 initramfs-virt
-r--r--r-- 1 root root  116112 10月  6  2021 ldlinux.c32
-r--r--r-- 1 root root   69632 10月  6  2021 ldlinux.sys
-rw-r--r-- 1 root root  179720 10月  6  2021 libcom32.c32
-rw-r--r-- 1 root root   23544 10月  6  2021 libutil.c32
drwx------ 2 root root   12288 10月  6  2021 lost+found
-rw-r--r-- 1 root root   11712 10月  6  2021 mboot.c32
-rw-r--r-- 1 root root   26568 10月  6  2021 menu.c32
-rw-r--r-- 1 root root   27020 10月  6  2021 vesamenu.c32
-rw-r--r-- 1 root root 6194624  8月 27  2021 vmlinuz-virt
                                                                                        
┌──(venv)─(ozaki㉿kali)-[/tmp/oni]
└─$ sudo ls -l /mnt/oni-p2
合計 38
drwxr-xr-x  2 root root  3072 10月  6  2021 bin
drwxr-xr-x  2 root root  1024 10月  6  2021 boot
drwxr-xr-x  2 root root  1024 10月  6  2021 dev
drwxr-xr-x 27 root root  3072 10月  6  2021 etc
drwxr-xr-x  2 root root  1024 10月  6  2021 home
drwxr-xr-x  9 root root  1024 10月  6  2021 lib
drwx------  2 root root 12288 10月  6  2021 lost+found
drwxr-xr-x  5 root root  1024 10月  6  2021 media
drwxr-xr-x  2 root root  1024 10月  6  2021 mnt
drwxr-xr-x  2 root root  1024 10月  6  2021 opt
drwxr-xr-x  2 root root  1024 10月  6  2021 proc
drwx------  3 root root  1024 10月  6  2021 root
drwxr-xr-x  2 root root  1024 10月  6  2021 run
drwxr-xr-x  2 root root  5120 10月  6  2021 sbin
drwxr-xr-x  2 root root  1024 10月  6  2021 srv
drwxr-xr-x  2 root root  1024 10月  6  2021 sys
drwxrwxrwt  4 root root  1024 10月  6  2021 tmp
drwxr-xr-x  8 root root  1024 10月  6  2021 usr
drwxr-xr-x 11 root root  1024 10月  6  2021 var
```
oni-p1のイメージにはカーネル、initramfs、ブートローダが
oni-p2のイメージにはrootファイルシステムがありますね。

今回探すのは`ssh`鍵ファイルなので、oni-p2をメインに探っていきたいと思います。
まずは`home`を見てみます。

```
$ sudo ls -la home
合計 2
drwxr-xr-x  2 root root 1024 10月  6  2021 .
drwxr-xr-x 21 root root 1024 10月  6  2021 ..
```
homeディレクトリにsshするユーザー(ctf-player)の鍵があるかな？と思ったのですが、違うみたいですね。

どんなユーザーがいるか確認します。
```
$ cat etc/passwd 
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
chrony:x:100:101:chrony:/var/log/chrony:/sbin/nologin
```
ほとんど`nologin`ですね。

`root`ディレクトリ配下を探してみますか。

```
$ sudo su                                            

# ls root 

# ls -la root
合計 4
drwx------  3 root root 1024 10月  6  2021 .
drwxr-xr-x 21 root root 1024 10月  6  2021 ..
-rw-------  1 root root   36 10月  6  2021 .ash_history
drwx------  2 root root 1024 10月  6  2021 .ssh

# ls -la root/.ssh 
合計 4
drwx------ 2 root root 1024 10月  6  2021 .
drwx------ 3 root root 1024 10月  6  2021 ..
-rw------- 1 root root  411 10月  6  2021 id_ed25519
-rw-r--r-- 1 root root   96 10月  6  2021 id_ed25519.pub
```
`id_ed25519`が鍵でしょうね。たぶん。

問題文のコマンドに従って、`ssh`してみましょう。
```
# ssh -i root/.ssh/id_ed25519 -p51272 ctf-player@saturn.picoctf.net
The authenticity of host '[saturn.picoctf.net]:51272 ([13.59.203.175]:51272)' can't be established.
ED25519 key fingerprint is SHA256:XBSvB1lk28EctsAVdKJtsl0A7C5bonqPrvHCYH8aEy4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[saturn.picoctf.net]:51272' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 6.8.0-1041-aws x86_64)

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

ctf-player@challenge:~$
```
おー、`ssh`成功ですね。

また、ここからflagを探すのか。
```
ctf-player@challenge:~$ ls -la
total 4
drwxr-xr-x 1 ctf-player ctf-player 20 Dec 15 20:23 .
drwxr-xr-x 1 root       root       24 Aug  4  2023 ..
drwx------ 2 ctf-player ctf-player 34 Dec 15 20:23 .cache
drwxr-xr-x 2 ctf-player ctf-player 29 Aug  4  2023 .ssh
-rw-r--r-- 1 root       root       28 Aug  4  2023 flag.txt
```
あ、あった。
```
ctf-player@challenge:~$ cat flag.txt 
picoCTF{k3y_5l3u7h_XXXXXXXX}ctf-player@challenge:~$ 
```
flagとれました。（flagはマスクしています。）