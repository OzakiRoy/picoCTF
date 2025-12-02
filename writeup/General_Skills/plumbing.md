# picoCTF Writeup: plumbing

**出力を制御する技術**を知ろう問題

- ジャンル: General Skills
- 難易度: Medium

## Writeup

問題文
>Sometimes you need to handle process data outside of a file. Can you find a way to keep the output from this program and search for the flag?
Connect to fickle-tempest.picoctf.net 55636.

[訳]
>ときどき、あなたはファイルの外のデータを取り扱う必要があります。プログラムのアウトプットを維持しながら、flagを探す方法がわかるかな？fickle-tempest.picoctf.net 55636 に接続してください。

ncで接続します。
```
▶ nc fickle-tempest.picoctf.net 55636 | head 
I don't think this is a flag either
Not a flag either
Again, I really don't think this is a flag
I don't think this is a flag either
Again, I really don't think this is a flag
I don't think this is a flag either
Not a flag either
I don't think this is a flag either
I don't think this is a flag either
Again, I really don't think this is a flag
...
```
ビャーっと出力が出てきました。
```
▶ nc fickle-tempest.picoctf.net 55636 | wc -l  
   10002
```
10002行です。
`picoCTF`で`grep`してみます。
```
▶ nc fickle-tempest.picoctf.net 55636 | grep picoCTF
picoCTF{digital_plumb3r_XXXXXXXX}
```
あら？あっさりflag取れました。(flagはマスクしています。)

