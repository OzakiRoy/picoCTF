$ curl -O https://challenge-files.picoctf.net/c_fickle_tempest/fea53d4b5a95f9e78fc25c77dd5332d9ef4aa71d2e64ea96bbe171e0300741b2/pico_img.png
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  106k  100  106k    0     0   125k      0 --:--:-- --:--:-- --:--:--  125k
                                                                                        
┌──(venv)─(ozaki㉿kali)-[/tmp/someta]
└─$ ls          
pico_img.png
                                                                                        
┌──(venv)─(ozaki㉿kali)-[/tmp/someta]
└─$ file pico_img.png    
pico_img.png: PNG image data, 600 x 600, 8-bit/color RGB, non-interlaced
                                                                                        
┌──(venv)─(ozaki㉿kali)-[/tmp/someta]
└─$ exiftool pico_img.png                                           
ExifTool Version Number         : 13.25
File Name                       : pico_img.png
Directory                       : .
File Size                       : 109 kB
File Modification Date/Time     : 2025:12:13 07:56:31+09:00
File Access Date/Time           : 2025:12:13 07:56:35+09:00
File Inode Change Date/Time     : 2025:12:13 07:56:31+09:00
File Permissions                : -rw-rw-r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 600
Image Height                    : 600
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Software                        : Adobe ImageReady
XMP Toolkit                     : Adobe XMP Core 5.3-c011 66.145661, 2012/02/06-14:56:27
Creator Tool                    : Adobe Photoshop CS6 (Windows)
Instance ID                     : xmp.iid:A5566E73B2B811E8BC7F9A4303DF1F9B
Document ID                     : xmp.did:A5566E74B2B811E8BC7F9A4303DF1F9B
Derived From Instance ID        : xmp.iid:A5566E71B2B811E8BC7F9A4303DF1F9B
Derived From Document ID        : xmp.did:A5566E72B2B811E8BC7F9A4303DF1F9B
Artist                          : picoCTF{s0_m3ta_bc056477}
Image Size                      : 600x600
Megapixels                      : 0.360
