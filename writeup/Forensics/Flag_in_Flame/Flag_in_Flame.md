# picoCTF Writeup: Flag in Flame

ファイルを識別してデコードする問題

- ジャンル: Forensics
- 難易度: Easy

## Writeup

問題文はこんな感じです。
>The SOC team discovered a suspiciously large log file after a recent breach. When they opened it, they found an enormous block of encoded text instead of typical logs. Could there be something hidden within? Your mission is to inspect the resulting file and reveal the real purpose of it. The team is relying on your skills to uncover any concealed information within this unusual log.
>Download the encoded data here: Logs Data. Be prepared—the file is large, and examining it thoroughly is crucial .
>SOCチームは、最近の侵害の後、不審なほど巨大なログファイルを発見しました。ファイルを開くと、通常のログではなく、膨大な量のエンコードされたテキストが見つかりました。何か隠されているのでしょうか？あなたの使命は、生成されたファイルを調べ、その真の目的を明らかにすることです。チームは、この異常なログに隠された情報を発見するために、あなたのスキルに期待しています。 エンコードされたデータは、こちらからダウンロードできます： ログデータ 。ファイルはサイズが大きいため、徹底的に調査することが重要です。 

では、ログファイルをダウンロードしていきます。　
```
# curl -O https://challenge-files.picoctf.net/c_amiable_citadel/1139fe6db834706f6db733413ff652f41951589576e1c4ddf082b8ecccce1454/logs.txt

# file logs.txt 
logs.txt: ASCII text, with very long lines (65536), with no line terminators

# du -h logs.txt 
1.6M logs.txt
```
ちょっと中身みてみよう。

```
# cat logs.txt           
iVBORw0KGgoAAAANSUhEUgAAA4AAAASACAIAAAAh8bSOAAEAAElEQVR4nOz919MsyZUniP3OcY+IFJ+6ouqWBqoautDYRmN6emfaZmlc0mi2+7xPNJJ/GJ/4RvJtH5dmQ1vbHu4Mp9kzjQYGohuoAkrh1tWfysyIcPdz+HAiIj3Vp66o
以下省略、大量に出るので気を付けてください。
```
**A-Z, a-z, 0-9, +, /**が出てるので、`base64`っぽいですね。

`base64` でデコードしてみる。

```
# cat logs.txt| base64 -d | head -5
�PNG
▒
IHDR�!���IDATx�����,ɕ'���q��������▒���Fczzg�fi\�h���O4�▒��F�m�fC[��
                                                                   ��3��J��՟�p�s�p""=է��jq��w3##<\-���/HA(�*�P�
                        "0T5j"u�9"UE"f&�""TՏ�����L��89;y��YU2Q�����CR��fm]�u���������|>o����7�������Ÿ�B��_����'
�P9vQ�q1��~��N����������'w���a��o����鴼��'�������af�����Ǟ��O>���_0�O��/�������?��OZ�'�KU����?������o������γ��c䋂RcogfU"bfU,?
                                    3E����7o��Ƴ/����~&�O��
ⵞ1Vغ%"���@T��D���*���t��,��������������o~�?���t��_�b��/Ӫj�v�{���}�G�4��o�[j����Z���W��_���۶��������U<SU��
```
あ、マジックナンバーが`PNG`ですね。
つまり、画像ファイルということか。

```
$ cat logs.txt| base64 -d > logs.png
```
`logs.png`を開いてみます。

![image](Flag_in_Flame.png)

なにやら下の方に文字がありますね。
`70`から始まっています。
これはASCIIコードでいう`p`なので、flagの可能性が高そうですね。

私の`kali`に`OCR`がなかったので、入れていきます。

```
# apt install tesseract-ocr
# tesseract --version
tesseract 5.5.0
 leptonica-1.86.0
  libgif 5.2.2 : libjpeg 6b (libjpeg-turbo 2.1.5) : libpng 1.6.52 : libtiff 4.7.1 : zlib 1.3.1 : libwebp 1.5.0 : libopenjp2 2.5.3
 Found SSE4.1
 Found OpenMP 201511
 Found libarchive 3.7.4 zlib/1.3.1 liblzma/5.8.1 bz2lib/1.0.8 liblz4/1.10.0 libzstd/1.5.7
 Found libcurl/8.13.0 OpenSSL/3.5.0 zlib/1.3.1 brotli/1.1.0 zstd/1.5.7 libidn2/2.3.8 libpsl/0.21.2 libssh2/1.11.1 nghttp2/1.64.0 nghttp3/1.8.0 librtmp/2.3 OpenLDAP/2.6.9
```

では、画像からテキストを読み取っていきます。

```
# tesseract logs.png output
Estimating resolution as 207
                          
# cat output.txt                    
~.
vbw
7

ge

ry
PNW

7069636F4354467B666F72656E736963735F 616E616C797369735F69735F61 6D617A696E675F32346431363839357D

KS

“
i
```

では、ASCII文字に変換します。

```
# echo '7069636F4354467B666F72656E736963735F 616E616C797369735F69735F61 6D617A696E675F32346431363839357D' | xxd -r -p
picoCTF{forensics_analysis_is_amazing_XXXXXXXX}
```
flagとれました。（flagはマスクしています。）