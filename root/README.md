# root権限

> The vulnerability, which is a signal handler race condition in OpenSSH’s server (sshd), allows unauthenticated remote code execution (RCE) as **root** on glibc-based Linux systems; that presents a significant security risk. This race condition affects sshd in its default configuration.

## Linuxの権限

Windows や Macと同じくLinuxの権限もシステム管理権限と一般権限に分かれます。

通常の操作は一般権限であれば十分であり、特定の操作のみシステム権限が求められます。

この強いシステム管理権限を持ったユーザーを root ユーザー(スーパーユーザー)とよび、ユーザーID=0が割り振られています。

```
$ id root
uid=0(root) gid=0(root) groups=0(root)
```

一般ユーザーの `ubuntu` と比較してみましょう

```
$ echo $USER  # ログインユーザー
ubuntu
$ id $USER
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),119(netdev),120(lxd),999(docker)
```

出力から `ubuntu` ユーザーは、以下の機能を使えるグループに属している事がわかります。

- 27(sudo)
- 999(docker)

## su/sudo コマンドで強い権限で操作

サーバーを停止したり、HTTP通信用の80番ポートでウェブサーバーを起動するなど、サーバー全体に影響をあたえるような操作を行う場合、システム管理者権限が必要です。

このような場合に `sudo` コマンドや `su` コマンドを使います。

### 別ユーザー権限でコマンド実行するsudo

`sudo` コマンドを利用すると、別ユーザーの権限でコマンドを実行できます。特に、root ユーザー権限でコマンド実行したい時に利用されます。

`man` には "execute a command as another user" とあります。

Ubuntuでは `systemd` というプログラムでプロセス管理されています。

一般ユーザー権限で、この systemd 経由で [WebサーバーのApache](https://httpd.apache.org/)(nginxの仲間)の再起動を試みると、権限不足のエラーが発生します。

```
$ systemctl restart apache2
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'apache2.service'.
Authenticating as: Ubuntu (ubuntu)
Password:
```

`sudo` を接頭すると、root ユーザーとして実行されるため、再起動が成功します。
```
$ sudo systemctl restart apache2
$
```

一般には `sudo` コマンド実行時には、ユーザーのパスワードが必要ですが、EC2 Ubuntuの場合は、`ubuntu` ユーザーはパスワード無しに `sudo` をよびだせる設定になっています。

```
$ sudo grep ubuntu /etc/sudoers
ubuntu    ALL=(ALL) NOPASSWD: /sbin/poweroff, /sbin/reboot, /sbin/shutdown
```

### 別ユーザーに変わる su

コマンド単位で別ユーザーの権限で実行するかわりに、別ユーザーにかわって、コマンドをまとめて実行したい場合に `su` コマンドを使います。

`man` には " run a command with substitute user and group ID" とあります。

```
$ su -       # root ユーザーになる。`-` をつけることで、ログインシェルとしてログイン(=`~/.profile` などの初期化ファイルを読み込む)
Password:

$ su - sato  # sato ユーザーになる
Password:
```

`su` 実行時には、切り替わる先のユーザーのパスワード入力が必要です。

EC2 Ubuntuの場合は、`root` パスワードは初期設定されていないため、一度rootユーザーになってパスワードを設定する、あるいは、次のようにして `sudo` 経由で `su` を呼び出すと、`root` パスワードは不要です。

```
$ sudo su -
root@ip-172-31-34-13:~#
root@ip-172-31-34-13:~# echo $USER
root
```

## Apache プロセスの場合

80番ポートで起動しているApache のプロセスを確認します

```
$ sudo lsof -i :80
COMMAND  PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
apache2 2529     root    4u  IPv6  23786      0t0  TCP *:http (LISTEN)
apache2 2531 www-data    4u  IPv6  23786      0t0  TCP *:http (LISTEN)
apache2 2532 www-data    4u  IPv6  23786      0t0  TCP *:http (LISTEN)
apache2 2533 www-data    4u  IPv6  23786      0t0  TCP *:http (LISTEN)
apache2 2534 www-data    4u  IPv6  23786      0t0  TCP *:http (LISTEN)
apache2 2535 www-data    4u  IPv6  23786      0t0  TCP *:http (LISTEN)
```

1~1024のポートを利用するネットワーク・サーバーの起動にはシステム管理者権限が必要です。

rootユーザーが80番ポートでApacheを起動し(PID=2529)、そこからの `fork()` で、80番ポート番号のサーバーの起動、及び、 `setuid` で実行ユーザーを root から `www-data` に変更しています。

対応する `pstree` 結果も共有します。

```
           ├─apache2(2529)─┬─apache2(2531)
           │               ├─apache2(2532)
           │               ├─apache2(2533)
           │               ├─apache2(2534)
           │               └─apache2(2535)
```

Python の `http` ライブラリを用い、`-p` オプションでポートを指定して起動してみましょう。

一般ユーザーで80番ポートのプロセスを起動しようとすると、エラーが発生します

```
$ echo $USER
ubuntu
$ python3 -m http.server 80
Traceback (most recent call last):
  File "/usr/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/usr/lib/python3.10/http/server.py", line 1307, in <module>
    test(
  File "/usr/lib/python3.10/http/server.py", line 1258, in test
    with ServerClass(addr, HandlerClass) as httpd:
  File "/usr/lib/python3.10/socketserver.py", line 452, in __init__
    self.server_bind()
  File "/usr/lib/python3.10/http/server.py", line 1301, in server_bind
    return super().server_bind()
  File "/usr/lib/python3.10/http/server.py", line 137, in server_bind
    socketserver.TCPServer.server_bind(self)
  File "/usr/lib/python3.10/socketserver.py", line 466, in server_bind
    self.socket.bind(self.server_address)
PermissionError: [Errno 13] Permission denied
```

`sudo` でシステム管理者権限で起動したり、1024より大きなポート番号で起動する必要があります。

```
$ sudo python3 -m http.server 80
Serving HTTP on :: port 80 (http://[::]:80/) ...

$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

## オプション:sudo の利用制限

Amazon EC2のAmazon LinuxやUbuntuでは、初期ユーザーは自由度高く `sudo` を呼び出せて、ルート権限で色々操作できます。

実運用では、どのユーザーに `sudo` を許可するか、どのコマンドを `sudo` で実行できるか、といったことを詰める必要も出てきます。

## 発展:コンテナの実行ユーザー

Dockerコンテナではデフォルトでrootユーザーで実行されています。

このようなコンテナは、ホストから見ると、コンテナプロセスがrootで動作することになり、ホスト側にエスケープされた場合にホストのroot権限が取得されてしまうことになります。

そのため、UIDマッピングやRootlessモードの使用が推奨されます。

詳細は森田 浩平著『基礎から学ぶコンテナセキュリティ』の「4.3セキュアなコンテナイメージを作る」や「5.5コンテナ実行ユーザーの変更と権限昇格の防止」を参照ください。
