# picoCTF Writeup: two-sum

intオーバーフローを知ろう問題

- ジャンル: Binary Exploitation
- 難易度: Medium

## Writeup
問題文
>Can you solve this?
>What two positive numbers can make this possible: n1 > n1 + n2 OR n2 > n1 + n2
>Enter them here nc saturn.picoctf.net 60831. Source

`int`と宣言すると、32bit(4Bytes)で2の補数表現で扱われます。

`int`は最上位bitが符号bitなので、最大値と最小値は以下になります。
|最大値|最小値|
|-|-|
|2,147,483,647 ( = 2^31 - 1)|-2,147,483,648 ( = -2^31)|

最大値`2147483647`に`1`を足すと、最上位bitが`1`となり最小値に変化します。

この考え方で、サーバーに接続してみます。

```
$ nc saturn.picoctf.net 57832
n1 > n1 + n2 OR n2 > n1 + n2 
What two positive numbers can make this possible:
2147483647
1
You entered 2147483647 and 1
You have an integer overflow
YOUR FLAG IS: picoCTF{Tw0_Sum_Integer_Bu773R_0v3rfl0w_482d8fc4}
```
flagとれました。