# picoCTF Writeup: Bitlocker-1

**Hash craking**、**BitLocker**を知ろう問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文
>Jacky is not very knowledgable about the best security passwords and used a simple password to encrypt their BitLocker drive. See if you can break through the encryption!
> Download the disk image here
>ジャッキーは知識があまりなくて、BitLockerドライブの暗号化に簡単なパスワードを使ってしまった。暗号化を解けるかみてみてね。ディスクイメージはここからダウンロード。


ダウンロードしたディスクイメージ(`bitlocker-1.dd`)について確認します。

BitLockerの場合は、`FIVE-FS`という固定識別子が含まれているとのこと。
```
ozaki@NucBoxM5PLUS:~/project/picoCTF$ strings /tmp/bitlocker-1.dd | grep -i fve-fs
-FVE-FS-
-FVE-FS-9
-FVE-FS-9
z-FVE-FS-9
```
BitLockerイメージであることは確認できました。
なお、`FVE` = Full Volume Encryption / `FS` = File System の略とのこと。
（最初`fdisk`を使ったのですが、BitLockerはメタデータもパーティションテーブルも暗号化されているため、`fdisk`では内容情報を確認できませんでした。）


BitLocker問題について少しググると、`bitlocker2john`というツールがあるとのこと。

今までwsl2のUbuntuを使っていたのですが、johnをビルドしなきゃなんないし、wordlistもないし、今後のことも考えまして、ViurtualBoxでKaliLinux VMを起動しました。

kaliを起動して`bitlocker2john`を使って、内部のハッシュ情報をファイルに吐き出します。
```
┌──(ozaki㉿kali)-[~]
└─$ bitlocker2john -i Downloads/bitlocker-1.dd > /tmp/hash.txt

Signature found at 0x3
Version: 8 
Invalid version, looking for a signature with valid version...

Signature found at 0x2195000
Version: 2 (Windows 7 or later)

VMK entry found at 0x21950c5

VMK encrypted with Recovery Password found at 0x21950e6
Searching AES-CCM from 0x2195102
Trying offset 0x2195195....
VMK encrypted with AES-CCM!!

VMK entry found at 0x2195241

VMK encrypted with User Password found at 2195262
VMK encrypted with AES-CCM

Signature found at 0x2c1d000
Version: 2 (Windows 7 or later)

VMK entry found at 0x2c1d0c5

VMK entry found at 0x2c1d241

Signature found at 0x373a000
Version: 2 (Windows 7 or later)

VMK entry found at 0x373a0c5

VMK entry found at 0x373a241
```
ハッシュ情報が手に入ったので、`john`とパスワードリストを使って、ジャッキーのパスワードを見つけていきます。
```
┌──(ozaki㉿kali)-[~]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/hash.txt 
Note: This format may emit false positives, so it will keep trying even after finding a possible candidate.
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (BitLocker, BitLocker [SHA-256 AES 32/64])
Cost 1 (iteration count) is 1048576 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:02:47 0.01% (ETA: 2025-12-22 17:58) 0g/s 11.21p/s 22.43c/s 22.43C/s warriors..microsoft
jacqueline       (?)     
jacqueline       (?)
```
出ました。`jacqueline`がパスワードらしいです。

では、次にパスワードを使ってBitLocker暗号化を解きます。
`dislocker`というパッケージを使うと複合できるとのこと。
`sudo apt install dislocker`でインストールしました。

複合したボリュームファイルを保管するディレクトリを作って、`dislocker`で複合します。

```
┌──(ozaki㉿kali)-[~]
└─$ sudo mkdir -p /mnt/bitlocker

┌──(ozaki㉿kali)-[~]
└─$ sudo dislocker -V ~/Downloads/bitlocker-1.dd -u"jacqueline" -- /mnt/bitlocker

┌──(ozaki㉿kali)-[~]
└─$ sudo ls -l /mnt/bitlocker
合計 0
-rw-rw-rw- 1 root root 104857600  1月  1  1970 dislocker-file
```

BitLockerボリュームをマウントします。
```
┌──(ozaki㉿kali)-[~]
└─$ sudo mount -o ro,loop /mnt/bitlocker/dislocker-file /mnt/bitlocker_mount
```
マウントできたので、中身をみていきます。

```  
┌──(ozaki㉿kali)-[~]
└─$ ls -l /mnt/bitlocker_mount 
合計 5
drwxrwxrwx 1 root root    0  7月 16  2024 '$RECYCLE.BIN'
drwxrwxrwx 1 root root 4096  7月 16  2024 'System Volume Information'
-rwxrwxrwx 1 root root   43  7月 16  2024  flag.txt
                                                                                        
┌──(ozaki㉿kali)-[~]
└─$ cat /mnt/bitlocker_mount/flag.txt
picoCTF{us3_b3tt3r_p4ssw0rd5_pl5!_XXXXXXXX} 
```
flag取れました。（flagはマスクしています。）

## BitLockerって安全じゃないの？
って思ったので、もう少し調べてみます。
一般的にBitLocker暗号化は、**TPM**と**回復キー**を使って運用されるようです。
- **TPM**(Trusted Platform Module)：
PCのマザーボード上に搭載されるセキュリティチップ、外部に取り出せない。
- **回復キー**：
48桁の鍵、PCが破損した場合、別PCへマウントして複合するために使う。ハッシュ化された形ではイメージ内に入っていない。

今回の問題では、ユーザーパスワードが簡単だったため突破できましたが、回復キーの場合は、回復キーはイメージ内に含まれないので不正に復号できないことになります。

どんなツールも正しく使うことが大切ですね。