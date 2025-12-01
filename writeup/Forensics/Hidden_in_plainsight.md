# picoCTF Writeup: Hidden in plainsight

**ステガノグラフィ**というデータ隠ぺい技術を知ろう問題

- ジャンル: Forensics
- 難易度: Easy

## Writeup

まずはダウンロードしたファイルをfileコマンドで確認する。
```
ozaki@NucBoxM5PLUS:~/project/picoCTF/writeup/Forensics$ file /tmp/img.jpg
/tmp/img.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, comment: "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9", baseline, precision 8, 640x640, components 3
```
なんかコメントに入っている。

base64デコードしてみる。
```
ozaki@NucBoxM5PLUS:~/project/picoCTF/writeup/Forensics$ echo -n "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9" | base64 -d
steghide:cEF6endvcmQ=
```
`steghide` って何？
`cEF6endvcmQ=` はまだでコードできそう。

```
ozaki@NucBoxM5PLUS:~/project/picoCTF/writeup/Forensics$ echo -n "cEF6endvcmQ=" | base64 -d
pAzzword
```

`steghide:pAzzword`
なんか、それっぽいんだけど、だから何？って感じですね。

`steghide`をググる。
https://steghide.sourceforge.net/
>Steghideは、様々な種類の画像ファイルや音声ファイルにデータを隠すことができるステガノグラフィプログラムです。


`steghide`を自分の環境にインストールして、flagを探すのかと思いました。
でも、`steghide`をインストールしたくなかったからググる。

こんなオンラインのデコーダーがありました。（ありがたや）
https://futureboy.us/stegano/decinput.html

問題の画像ファイルとパスワードを入力して、ペイロードをデコードするらしい。
パスワードには`pAzzword`を使えばいいんだな。

flagが表示されました。

## 余談

`ステガノグラフィ`って、リアルではどういったときに使われるの？と思いました。
>日本におけるデジタルデータの隠ぺい方法は、アンダーグラウンドでの需要と相まって、独自の発展を遂げました。

受け渡しをしたいデータを隠ぺいする方法の一つということですね。
普通に生きていれば使うことはないと思う。
社内での発行した認証情報の受け渡しなんかに使うとオシャレかもです。

参考にさせていただいたリンクです。ありがとうございます。
https://digitaltravesia.jp/usamimihurricane/webhelp/_RESOURCE/MenuItem/another/anotherAboutSteganography.html#history