# picoCTF Writeup: MacroHard WeakEdge

**pptm**ファイルって`unzip`できるんだぜ？問題

- ジャンル: Forensics
- 難易度: Medium

## Writeup

問題文
>I've hidden a flag in this file. Can you find it? Forensics is fun.pptm
私はflagをこのファイルに隠した。みつけられるかな？pptmファイル

まずは、pptmファイルをダウンロードします。
```
# curl -O https://mercury.picoctf.net/static/c00c449c3b08daaccacca6f9d5c55d49/Forensics%20is%20fun.pptm
```

`file`コマンドで確認します。
```
# file Forensics%20is%20fun.pptm
Forensics%20is%20fun.pptm: Microsoft PowerPoint 2007+
```
パワポのファイルですね。

`exiftool`で確認します。
```
# exiftool Forensics%20is%20fun.pptm                  
ExifTool Version Number         : 13.25
File Name                       : Forensics%20is%20fun.pptm
Directory                       : .
File Size                       : 100 kB
File Modification Date/Time     : 2025:12:17 06:51:48+09:00
File Access Date/Time           : 2025:12:17 08:05:54+09:00
File Inode Change Date/Time     : 2025:12:17 06:51:48+09:00
File Permissions                : -rw-r--r--
File Type                       : PPTM
File Type Extension             : pptm
MIME Type                       : application/vnd.ms-powerpoint.presentation.macroEnabled.12
Zip Required Version            : 20
Zip Bit Flag                    : 0x0006
Zip Compression                 : Deflated
Zip Modify Date                 : 1980:01:01 00:00:00
Zip CRC                         : 0xa0517e97
Zip Compressed Size             : 674
Zip Uncompressed Size           : 10660
Zip File Name                   : [Content_Types].xml
Preview Image                   : (Binary data 2278 bytes, use -b option to extract)
Title                           : Forensics is fun
Creator                         : John
Last Modified By                : John
Revision Number                 : 2
Create Date                     : 2020:10:23 18:21:24Z
Modify Date                     : 2020:10:23 18:35:27Z
Total Edit Time                 : 4 minutes
Words                           : 7
Application                     : Microsoft Office PowerPoint
Presentation Format             : Widescreen
Paragraphs                      : 2
Slides                          : 58
Notes                           : 0
Hidden Slides                   : 1
MM Clips                        : 0
Scale Crop                      : No
Heading Pairs                   : Fonts Used, 3, Theme, 1, Slide Titles, 58
Titles Of Parts                 : Arial, Calibri, Calibri Light, Office Theme, Forensics is fun, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation, PowerPoint Presentation
Links Up To Date                : No
Shared Doc                      : No
Hyperlinks Changed              : No
App Version                     : 16.0000
```
やけに多い`PowerPoint Presentation`が気になりましたが、
それよりも`Zip Compression : Deflated`が引っ掛かりました。
`pptm`って`zip`圧縮されてるんですね。知りませんでした。

`unzip`してみたいと思います。
```
# unzip Forensics%20is%20fun.pptm -d fun
出力略
# ls fun 
'[Content_Types].xml'   _rels   docProps   ppt
```
`fun`ディレクトリに展開できました。

では、flagを探していきます。
```
# cd fun                                               
   
# ls
'[Content_Types].xml'   _rels   docProps   ppt

# find . -name *flag*

# grep -r  pico .

```
`find`しても、`grep`しても、引っ掛かりません。。。

`tree`でディレクトリ構造を確認します。
```
# tree                     
.
├── [Content_Types].xml
├── _rels
├── docProps
│   ├── app.xml
│   ├── core.xml
│   └── thumbnail.jpeg
└── ppt
    ├── _rels
    │   └── presentation.xml.rels
    ├── presProps.xml
    ├── presentation.xml
    ├── slideLayouts
    │   ├── _rels
    │   │   ├── slideLayout1.xml.rels
    │   │   ├── slideLayout10.xml.rels
    │   │   ├── slideLayout11.xml.rels
    │   │   ├── slideLayout2.xml.rels
    │   │   ├── slideLayout3.xml.rels
    │   │   ├── slideLayout4.xml.rels
    │   │   ├── slideLayout5.xml.rels
    │   │   ├── slideLayout6.xml.rels
    │   │   ├── slideLayout7.xml.rels
    │   │   ├── slideLayout8.xml.rels
    │   │   └── slideLayout9.xml.rels
    │   ├── slideLayout1.xml
    │   ├── slideLayout10.xml
    │   ├── slideLayout11.xml
    │   ├── slideLayout2.xml
    │   ├── slideLayout3.xml
    │   ├── slideLayout4.xml
    │   ├── slideLayout5.xml
    │   ├── slideLayout6.xml
    │   ├── slideLayout7.xml
    │   ├── slideLayout8.xml
    │   └── slideLayout9.xml
    ├── slideMasters
    │   ├── _rels
    │   │   └── slideMaster1.xml.rels
    │   ├── hidden
    │   └── slideMaster1.xml
    ├── slides
    │   ├── _rels
    │   │   ├── slide1.xml.rels
    │   │   ├── slide10.xml.rels
    │   │   ├── slide11.xml.rels
    │   │   ├── slide12.xml.rels
    │   │   ├── slide13.xml.rels
    │   │   ├── slide14.xml.rels
    │   │   ├── slide15.xml.rels
    │   │   ├── slide16.xml.rels
    │   │   ├── slide17.xml.rels
    │   │   ├── slide18.xml.rels
    │   │   ├── slide19.xml.rels
    │   │   ├── slide2.xml.rels
    │   │   ├── slide20.xml.rels
    │   │   ├── slide21.xml.rels
    │   │   ├── slide22.xml.rels
    │   │   ├── slide23.xml.rels
    │   │   ├── slide24.xml.rels
    │   │   ├── slide25.xml.rels
    │   │   ├── slide26.xml.rels
    │   │   ├── slide27.xml.rels
    │   │   ├── slide28.xml.rels
    │   │   ├── slide29.xml.rels
    │   │   ├── slide3.xml.rels
    │   │   ├── slide30.xml.rels
    │   │   ├── slide31.xml.rels
    │   │   ├── slide32.xml.rels
    │   │   ├── slide33.xml.rels
    │   │   ├── slide34.xml.rels
    │   │   ├── slide35.xml.rels
    │   │   ├── slide36.xml.rels
    │   │   ├── slide37.xml.rels
    │   │   ├── slide38.xml.rels
    │   │   ├── slide39.xml.rels
    │   │   ├── slide4.xml.rels
    │   │   ├── slide40.xml.rels
    │   │   ├── slide41.xml.rels
    │   │   ├── slide42.xml.rels
    │   │   ├── slide43.xml.rels
    │   │   ├── slide44.xml.rels
    │   │   ├── slide45.xml.rels
    │   │   ├── slide46.xml.rels
    │   │   ├── slide47.xml.rels
    │   │   ├── slide48.xml.rels
    │   │   ├── slide49.xml.rels
    │   │   ├── slide5.xml.rels
    │   │   ├── slide50.xml.rels
    │   │   ├── slide51.xml.rels
    │   │   ├── slide52.xml.rels
    │   │   ├── slide53.xml.rels
    │   │   ├── slide54.xml.rels
    │   │   ├── slide55.xml.rels
    │   │   ├── slide56.xml.rels
    │   │   ├── slide57.xml.rels
    │   │   ├── slide58.xml.rels
    │   │   ├── slide6.xml.rels
    │   │   ├── slide7.xml.rels
    │   │   ├── slide8.xml.rels
    │   │   └── slide9.xml.rels
    │   ├── slide1.xml
    │   ├── slide10.xml
    │   ├── slide11.xml
    │   ├── slide12.xml
    │   ├── slide13.xml
    │   ├── slide14.xml
    │   ├── slide15.xml
    │   ├── slide16.xml
    │   ├── slide17.xml
    │   ├── slide18.xml
    │   ├── slide19.xml
    │   ├── slide2.xml
    │   ├── slide20.xml
    │   ├── slide21.xml
    │   ├── slide22.xml
    │   ├── slide23.xml
    │   ├── slide24.xml
    │   ├── slide25.xml
    │   ├── slide26.xml
    │   ├── slide27.xml
    │   ├── slide28.xml
    │   ├── slide29.xml
    │   ├── slide3.xml
    │   ├── slide30.xml
    │   ├── slide31.xml
    │   ├── slide32.xml
    │   ├── slide33.xml
    │   ├── slide34.xml
    │   ├── slide35.xml
    │   ├── slide36.xml
    │   ├── slide37.xml
    │   ├── slide38.xml
    │   ├── slide39.xml
    │   ├── slide4.xml
    │   ├── slide40.xml
    │   ├── slide41.xml
    │   ├── slide42.xml
    │   ├── slide43.xml
    │   ├── slide44.xml
    │   ├── slide45.xml
    │   ├── slide46.xml
    │   ├── slide47.xml
    │   ├── slide48.xml
    │   ├── slide49.xml
    │   ├── slide5.xml
    │   ├── slide50.xml
    │   ├── slide51.xml
    │   ├── slide52.xml
    │   ├── slide53.xml
    │   ├── slide54.xml
    │   ├── slide55.xml
    │   ├── slide56.xml
    │   ├── slide57.xml
    │   ├── slide58.xml
    │   ├── slide6.xml
    │   ├── slide7.xml
    │   ├── slide8.xml
    │   └── slide9.xml
    ├── tableStyles.xml
    ├── theme
    │   └── theme1.xml
    ├── vbaProject.bin
    └── viewProps.xml

12 directories, 152 files
```
おわかりいただけただろうか。。。
怪しいファイルがあったことに。。。
`hidden`という怪しい名前のファイルがあります。

```
# cat ppt/slideMasters/hidden 
Z m x h Z z o g c G l j b 0 N U R n t E M W R f d V 9 r b j B 3 X 3 B w d H N f c l 9 6 M X A 1 f Q
```
`tr`で空白を抜きます。
```
# cat ppt/slideMasters/hidden | tr -d ' '
ZmxhZzogcGljb0NURntEMWRfdV9rbjB3X3BwdHNfcl96MXA1fQ
```
**A-Z, a-z, 0-9**が出てるので、`base64`っぽいですね。

`base64` でデコードしてみる。
```
# cat ppt/slideMasters/hidden | tr -d ' ' | base64 -d
flag: picoCTF{D1d_u_kn0w_ppts_r_XXXX}
```
flagとれました。（flagはマスクしています。）