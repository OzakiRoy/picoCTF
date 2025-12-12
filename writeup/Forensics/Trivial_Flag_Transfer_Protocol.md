# picoCTF Writeup: Trivial Flag Transfer Protocol

この問題はpcap解析問題と見せかけた**ステガノグラフィ**問題です。

- ジャンル: Forensics
- 難易度: Medium

## Writeup
問題文
>Figure out how they moved the flag.
>彼らがflagをどうやって送ったかわかるかな？

まず、pcapファイルをダウンロードします。
```
$ curl -O https://mercury.picoctf.net/static/ed308d382ae6bcc37a5ebc701a1cc4f4/tftp.pcapng 
```
`wireshark`でpcapファイルを開いて一通り見ます。

picoで文字列検索してもヒットしません。
うーん、どうしましょうか。

`tftp`で何のファイルを転送してたのでしょうか。
ファイルをpcapから取り出してみようと思います。

`wireshark`からでも、`tshark`(whiresharkのCLI版)もできるようです。

今回は`tshark`でオブジェクトを取り出します。
```
$ tshark -q -r tftp.pcapng --export-objects tftp,./extracted
```
```
$ ls                            
extracted  tftp.pcapng
$ ls extracted 
instructions.txt  picture1.bmp  picture2.bmp  picture3.bmp  plan  program.deb
```
いくつかファイルが出てきましたね。
まずは、`instructions.txt`を見てみます。
```
$ cd extracted 
$ cat instructions.txt                            
GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA
```
んー、なんじゃこりや？

すべて大文字英字で、記号や数字が含まれていないので、
**ROT13**かもしれない。
```
$ cat instructions.txt| tr 'A-Za-z' 'N-ZA-Mn-za-m'
TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN
```
おー、英語になってますね。
```
TFTP DOESNT ENCRYPT OUR TRAFFIC SO WE MUST DISGUISE OUR FLAG TRANSFER. FIGURE OUT A WAY TO HIDE THE FLAG AND I WILL CHECK BACK FOR THE PLAN
TFTPは我々の通信トラフィックを暗号化しないから、我々はflag転送を偽装しないといけない。flagを隠す方法を見つけて、あとで計画を確認します。
```
ちょっと最後の文章の意味がよくわかりませんが、flagを隠す方法は計画=`plan`に書いてあるんでしょうね。

`plan`の方も見てみましょう。
```
$ cat plan                                        
VHFRQGURCEBTENZNAQUVQVGJVGU-QHRQVYVTRAPR.PURPXBHGGURCUBGBF
```
これも**ROT13**ぽいです。
```
$ cat plan | tr 'A-Za-z' 'N-ZA-Mn-za-m'
IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
```
```
I USED THE PROGRAM AND HID IT WITH-DUEDILIGENCE.CHECK OUT THE PHOTOS
プログラムを使って、DUEDILIGENCEと一緒にflagを隠した。写真を確認してください。
```
ここまでで、なにかのプログラムを使って写真の`picture1.bmp  picture2.bmp  picture3.bmp`にflagを隠したっぽいことがわかりました。

どんなプログラムをつかったのでしょうか？
`program.deb`これですね。
展開してみてみます。
```
$ dpkg -x program.deb ./prog
$ tree prog                                                 
prog
└── usr
    ├── bin
    │   └── steghide
    └── share
        ├── doc
        │   └── steghide
        │       ├── ABOUT-NLS.gz
        │       ├── BUGS
        │       ├── CREDITS
        │       ├── HISTORY
        │       ├── LEAME.gz
        │       ├── README.gz
        │       ├── TODO
        │       ├── changelog.Debian.amd64.gz
        │       ├── changelog.Debian.gz
        │       ├── changelog.gz
        │       └── copyright
        ├── locale
        │   ├── de
        │   │   └── LC_MESSAGES
        │   │       └── steghide.mo
        │   ├── es
        │   │   └── LC_MESSAGES
        │   │       └── steghide.mo
        │   ├── fr
        │   │   └── LC_MESSAGES
        │   │       └── steghide.mo
        │   └── ro
        │       └── LC_MESSAGES
        │           └── steghide.mo
        └── man
            └── man1
                └── steghide.1.gz

17 directories, 17 files
```
`steghide`というプログラムを使ってflagを画像に埋め込んだということですね。

それでは`steghide`を使ってflagを取り出してみます。

`steghide`ではflag埋め込み時に利用したパスワードが必要になります。
改めて`plan`を見てみると
```
I USED THE PROGRAM AND HID IT WITH-DUEDILIGENCE.CHECK OUT THE PHOTOS
```
`DUEDILIGENCE`で隠したと書いてありました。

ちょっと自信ないですが、パスワードは`DUEDILIGENCE`であろうという予想で進めます。

```
$ steghide extract -sf picture1.bmp -p "DUEDILIGENCE" 

steghide: could not extract any data with that passphrase!
```                                                         
`picture1.bmp`はダメ

```                            
$ steghide extract -sf picture2.bmp -p "DUEDILIGENCE"

steghide: could not extract any data with that passphrase!
```                                                         
`picture2.bmp`もダメ

```                            
$ steghide extract -sf picture3.bmp -p "DUEDILIGENCE"

wrote extracted data to "flag.txt".
```
`picture3.bmp`に`flag.txt`ありました。

```
$ cat flag.txt                        
picoCTF{h1dd3n_1n_pLa1n_51GHT_XXXXXXXX}
```
flagとれました。（flagはマスクしています。）

## まとめ
今回は色々出てきましたね。
tsharkでobject export, ROT13, steghideと。
他にもbmpファイルが出てきたのでLSBを疑ってzstegも試しましたが、今回ははまらなかったですね。
画像ファイルに対して攻め口の引き出しが増えると面白いですね。
その内、攻めパターンを整理したいと思います。