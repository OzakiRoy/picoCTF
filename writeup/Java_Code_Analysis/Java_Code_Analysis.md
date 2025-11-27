JWTを操作してAdmin権限を取得する

1. 問題で与えられているuser, userでlogin
1. JWTを確認
1. decodeしてpayloadをadmin用に修正
![jwt](Java_Code_Analysis_01.png)
1. keyは1234(source code), user_idは2(guess)
1. 作成したJWTでブラウザリロード
1. Admin権限でログインできているのでFlag bookを開く
![admin](Java_Code_Analysis_admin.png)
1. Flag
![flag](Java_Code_Analysis_flag.png)

Best Youtube
https://www.youtube.com/watch?v=y6g7Q63LQPU