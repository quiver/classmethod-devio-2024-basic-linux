# プロセス

学ぶこと

- プロセス
- コンテナにおけるプロセス

### プロセスとは？

実行中のプログラムのことをプロセスと呼びます。

Linuxでは `ps` コマンド(ps=process status)からプロセス情報を確認できます。

現在ログインしているシェル(=実行中のプログラム)とそのプロセスID(PID)を取得します

```
$ echo $SHELL
/bin/bash
$ echo $$
1483
```

ログインユーザー(`ubuntu`)権限で実行されているプロセス一覧を確認します。

```
 $ ps ux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
ubuntu      1025  0.0  0.4  17052  9600 ?        Ss   06:22   0:00 /lib/systemd/systemd --user
...
ubuntu      1483  0.0  0.5  14960 10880 pts/1    Ss   06:22   0:00 bash -l
...
ubuntu      3283  0.0  0.1  10464  3200 pts/1    R+   06:40   0:00 ps ux
```

プロセスIDが1483の `bash` を確認できます。

## Apache版

`pstree` コマンドから、この上下関係を確認します。

```
$ pstree -p
systemd(1)─┬─acpid(367)
...
           ├─apache2(492)─┬─apache2(566)
           │              ├─apache2(568)
           │              ├─apache2(569)
           │              ├─apache2(578)
           │              └─apache2(580)...
```

プロセス名に続くカッコの数字はプロセスID(PID)です。

ブートローダーがLinuxカーネルを起動すると、LinuxカーネルがinitシステムをPID=1で起動し、このシステムを起点に、様々なプログラムが起動されます。
Ubuntu 22.04 では、このinitシステムに `systemd` が採用されています。

`systemd` の概要を公式ページから引用します。

> systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system.
>
> https://systemd.io/


プロセスは、プロセスを管理する側と管理される側が存在します。

`pstree` で左側が管理する側、右側が管理される側です。

systemd(1) と apache2(492) の場合、systemdが管理する側、apache2(492)が管理される側です。

systemctl でプロセスのステータスを確認

```
$ systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-07-10 06:22:22 UTC; 21min ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 368 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 492 (apache2)
      Tasks: 6 (limit: 2262)
     Memory: 25.9M
        CPU: 199ms
     CGroup: /system.slice/apache2.service
             ├─492 /usr/sbin/apache2 -k start
             ├─566 /usr/sbin/apache2 -k start
             ├─568 /usr/sbin/apache2 -k start
             ├─569 /usr/sbin/apache2 -k start
             ├─578 /usr/sbin/apache2 -k start
             └─580 /usr/sbin/apache2 -k start

Jul 10 06:22:22 ip-172-31-34-13 systemd[1]: Starting The Apache HTTP Server...
Jul 10 06:22:22 ip-172-31-34-13 systemd[1]: Started The Apache HTTP Server.
```

中央に `Main PID: 492 (apache2)` とあります。

プロセスIDの 492, 566, 568, ..., 580 はフラットではなく、

492が管理する側、566から580が管理される側です。

`pstree` のツリー表示と一致します。

## コンテナでのプロセスの考え方

アプリケーションをコンテナ化する上で、アプリケーションコンテナとシステムコンテナという2つの考え方があります。

![](container.jpeg)

※ [LXD vs Docker | Ubuntu](https://ubuntu.com/blog/lxd-vs-docker) から

現在主流のアプリケーションコンテナは、APIサーバーなどアプリケーション単位でコンテナ化します。
コンテナ内で、プロセス管理サービスを起動し、複数プロセスを管理するのはアンチパターンとされています。

> コンテナ内で単一のアプリケーションプロセスを実行します。
>
> コンテナの有効期間は、アプリケーションプロセスが実行されている期間です。Amazon ECS は、クラッシュしたプロセスを置き換えると共に、置き換え後のプロセスをどこで起動するかを決定します。適切に完成されたイメージは、デプロイ全体の耐障害性を高めます。
> [Best practices for Amazon ECS container images - Amazon Elastic Container Service](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/container-considerations.html)

Twelve-Factor App の VI. プロセスでも "アプリケーションを1つもしくは複数のステートレスなプロセスとして実行する" とあります。

[The Twelve-Factor App （日本語訳）](https://12factor.net/ja/processes)

システムコンテナはLXDなどで採用され、名前の通り、複数のアプリケーションからなるシステム全体を一つのコンテナに閉じ込めます。

## (発展)プロセスの作り方

Apacheサーバーを起動というように、新規にプロセスを作成するとき、fork & exec という仕組みが使われます。

プロセスを2つに分裂させ(`fork()`)、片方の中身を起動したいプログラムで書き換えます(`exec()`)。

詳細は『Linuxのしくみ』 2章を参照してください。

