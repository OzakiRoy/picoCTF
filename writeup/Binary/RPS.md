# picoCTF Writeup: RPS

ソースをしっかり読んでみよう問題

- ジャンル: Binary Exploitation
- 難易度: Medium

## Writeup
問題文
>Here's a program that plays rock, paper, scissors against you. I hear something good happens if you win 5 times in a row.
The program's source code with the flag redacted can be downloaded here.
Connect to the program with netcat:
$ nc saturn.picoctf.net 59835
じゃんけんプログラムがあります。5回連続で勝つといいことがあると聞きました。プログラムソースコード（flag公開される）はここからダウンロードできます。netcatでプログラムに接続してください。

さて、まずはソースコードを読んでいきましょうか。

`main`からですね。
```
int main () {
  char input[3] = {'\0'};
  int command;
  int r;

  puts("Welcome challenger to the game of Rock, Paper, Scissors");
  puts("For anyone that beats me 5 times in a row, I will offer up a flag I found");
  puts("Are you ready?");
  
  while (true) {
    puts("Type '1' to play a game");
    puts("Type '2' to exit the program");
    r = tgetinput(input, 3);
    // Timeout on user input
    if(r == -3)
    {
      printf("Goodbye!\n");
      exit(0);
    }
    
    if ((command = strtol(input, NULL, 10)) == 0) {
      puts("Please put in a valid number");
      
    } else if (command == 1) {
      printf("\n\n");
      if (play()) {
        wins++;
      } else {
        wins = 0;
      }

      if (wins >= 5) {
        puts("Congrats, here's the flag!");
        puts(flag);
      }
    } else if (command == 2) {
      return 0;
    } else {
      puts("Please type either 1 or 2");
    }
  }

  return 0;
}
```

`play()`の結果`true`を返せば`wins`がカウントアップされて、`wins`が5以上になればflagがとれますね。

では、`play()`をみてみます。
```
bool play () {
  char player_turn[100];
  srand(time(0));
  int r;

  printf("Please make your selection (rock/paper/scissors):\n");
  r = tgetinput(player_turn, 100);
  // Timeout on user input
  if(r == -3)
  {
    printf("Goodbye!\n");
    exit(0);
  }

  int computer_turn = rand() % 3;
  printf("You played: %s\n", player_turn);
  printf("The computer played: %s\n", hands[computer_turn]);

  if (strstr(player_turn, loses[computer_turn])) {
    puts("You win! Play again?");
    return true;
  } else {
    puts("Seems like you didn't win this time. Play again?");
    return false;
  }
}
```

`if (strstr(player_turn, loses[computer_turn])) {`
これで勝敗を判定しています。
`strstr()`は部分一致検索して一致があればその場所のポインタを返す関数とのことで、`player_turn`の中に`loses[computer_turn]`が入っていれば勝ちになるようです。

次に`player_turn`と`loses[computer_turn]`に何が入っているか見ていきます。

まず`player_turn`
```
  printf("Please make your selection (rock/paper/scissors):\n");
  r = tgetinput(player_turn, 100);
```
`player_turn`には、プレイヤーの入力文字列があ入るようです。

そして`loses[computer_turn]`
```
int computer_turn = rand() % 3;
  printf("You played: %s\n", player_turn);
  printf("The computer played: %s\n", hands[computer_turn]);

  if (strstr(player_turn, loses[computer_turn])) {
```
`computer_turn`にはランダム数値を3で割った余りが入ります。
`loses[computer_turn]`は
```
char* hands[3] = {"rock", "paper", "scissors"};
char* loses[3] = {"paper", "scissors", "rock"};
```
その余りの数値(0~2)に対応した手の文字列となるようです。

つまり、
`if (strstr(player_turn, loses[computer_turn])) {`
プレイヤーの入力文字列に"paper", "scissors", "rock"のすべてが含まれていれば必ず勝てることになります。

例えば、今回は入力文字列を`rockpaperscissors`にしてやってみます。
```
$ nc saturn.picoctf.net 62121
Welcome challenger to the game of Rock, Paper, Scissors
For anyone that beats me 5 times in a row, I will offer up a flag I found
Are you ready?
Type '1' to play a game
Type '2' to exit the program
1
1


Please make your selection (rock/paper/scissors):
rockpaperscissors
rockpaperscissors
You played: rockpaperscissors
The computer played: paper
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1
1


Please make your selection (rock/paper/scissors):
rockpaperscissors
rockpaperscissors
You played: rockpaperscissors
The computer played: rock
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1
1


Please make your selection (rock/paper/scissors):
rockpaperscissors
rockpaperscissors
You played: rockpaperscissors
The computer played: paper
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1
1


Please make your selection (rock/paper/scissors):
rockpaperscissors
rockpaperscissors
You played: rockpaperscissors
The computer played: scissors
You win! Play again?
Type '1' to play a game
Type '2' to exit the program
1
1


Please make your selection (rock/paper/scissors):
rockpaperscissors
rockpaperscissors
You played: rockpaperscissors
The computer played: paper
You win! Play again?
Congrats, here's the flag!
picoCTF{50M3_3X7R3M3_1UCK_B69E01B8}
Type '1' to play a game
Type '2' to exit the program
```
flagとれました。
`picoCTF{50M3_3X7R3M3_1UCK_B69E01B8}`

## 余談
勝利判定は`strcmp()`を使い、完全一致にした方が安全ですね。