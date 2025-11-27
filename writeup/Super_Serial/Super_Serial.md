## Unserializeは危険

serializeはオブジェクトを保存・送信可能な文字列に変換すること
（オブジェクト　→　serialize() →　文字列）
unserializeは受け取った文字列からオブジェクトを生成する
攻撃者の文字列をオブジェクト化して実行させてしまうから、unserializeは危険である

以下はflag取得までの流れ

robots.txtを確認
```
ozaki@NucBoxM5PLUS:~/project/picoCTF$ curl http://mercury.picoctf.net:25395/robots.txt
User-agent: *
Disallow: /admin.phps
```
phpsファイルはphpソースが見れる
index.phpsから見ていく
cookie.php, authentication.phpの存在を確認

```
ozaki@NucBoxM5PLUS:~/project/picoCTF$ curl http://mercury.picoctf.net:25395/index.phps
<?php
require_once("cookie.php");

if(isset($_POST["user"]) && isset($_POST["pass"])){
        $con = new SQLite3("../users.db");
        $username = $_POST["user"];
        $password = $_POST["pass"];
        $perm_res = new permissions($username, $password);
        if ($perm_res->is_guest() || $perm_res->is_admin()) {
                setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + (86400 * 30), "/");
                header("Location: authentication.php");
                die();
        } else {
                $msg = '<h6 class="text-center" style="color:red">Invalid Login.</h6>';
        }
}
?>
```

cookie.phps
```
if(isset($_COOKIE["login"])){
        try{
                $perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
                $g = $perm->is_guest();
                $a = $perm->is_admin();
        }
        catch(Error $e){
                die("Deserialization error. ".$perm);
        }
}
```
login cookieをurl　decodeしてbase64 decodeして、unserializeしている

authentication.phps
```
ozaki@NucBoxM5PLUS:~/project/picoCTF$ curl -c guest-cookie http://mercury.picoctf.net:25395/authentication.phps
<?php

class access_log
{
        public $log_file;

        function __construct($lf) {
                $this->log_file = $lf;
        }

        function __toString() {
                return $this->read_log();
        }

        function append_to_log($data) {
                file_put_contents($this->log_file, $data, FILE_APPEND);
        }

        function read_log() {
                return file_get_contents($this->log_file);
        }
}

require_once("cookie.php");
if(isset($perm) && $perm->is_admin()){
        $msg = "Welcome admin";
        $log = new access_log("access.log");
        $log->append_to_log("Logged in at ".date("Y-m-d")."\n");
} else {
        $msg = "Welcome guest";
}
?>
```
$log_fileに../flagを渡せば、read_log()でflag中身が見れそう


[3v4l.org](https://3v4l.org/)
![alt text](Super_Serial_img.png)

serializeした文字列をcookie loginに持たせてauthentication.phpをリロード
![flag](Super_Serial_flag.png)