## スレッドとは?

プロセスごとに固有の処理を割り当てることも可能ですが、プロセスの中で、並列化可能な単位に処理を分割し、複数の処理を行うことも可能です。この処理の単位をスレッドと呼びます。

MySQLのようなデータベースサーバーでは、クライアントからのSQL処理をマルチスレッドで処理したり、レプリケーションのような管理系処理を特定のスレッドで処理したりしています。

## Webサーバーのスケールアウト戦略

Webサーバーはたくさんのクライアントのリクエストを処理する必要があります。
同じ処理をスケールアウトするための手段として

- マルチプロセス
- マルチスレッド

の2通りがあります。

Apacheはプロセス方式(prework)とスレッド方式(worker/event)の両方に対応しています。
eventはApache 2.2から追加されたWorkerのバリエーションであり、Ubuntu 22.04 のデフォルトのApache2はevent方式で起動されています。

event 方式の機能概要をドキュメントから引用します。


> The event Multi-Processing Module (MPM) is designed to allow more requests to be served simultaneously by passing off some processing work to the listeners threads, freeing up the worker threads to serve new requests.
>
> [event - Apache HTTP Server Version 2.4](https://httpd.apache.org/docs/2.4/mod/event.html)

## event型Apacheの動作を確認

設定ファイルからわかるように、複数のプロセスを起動させ(`StartServers`)、各プロセスをマルチスレッド(`ThreadsPerChild`)で動作させています。

```
$ cat /etc/apache2/mods-enabled/mpm_event.conf
# event MPM
# StartServers: initial number of server processes to start
# MinSpareThreads: minimum number of worker threads which are kept spare
# MaxSpareThreads: maximum number of worker threads which are kept spare
# ThreadsPerChild: constant number of worker threads in each server process
# MaxRequestWorkers: maximum number of worker threads
# MaxConnectionsPerChild: maximum number of requests a server process serves
StartServers            2
MinSpareThreads         25
MaxSpareThreads         75
ThreadLimit             64
ThreadsPerChild         25
MaxRequestWorkers       150
MaxConnectionsPerChild  0
```

Apacheプロセスを確認しましょう。

```
$ ps -f -p $(pgrep apache2)
UID          PID    PPID  C STIME TTY      STAT   TIME CMD
root        1468       1  0 04:19 ?        Ss     0:01 /usr/sbin/apache2 -k start
www-data    1471    1468  0 04:19 ?        Sl     0:00 /usr/sbin/apache2 -k start
www-data    1472    1468  0 04:19 ?        Sl     0:00 /usr/sbin/apache2 -k start
```

`ps` に `-T` オプションを渡すと、`SPID` 列からスレッドIDも取得できます。

```
$ ps -T -p $(pgrep apache2)
    PID    SPID TTY      STAT   TIME COMMAND
   1468    1468 ?        Ss     0:01 /usr/sbin/apache2 -k start
   1471    1471 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1475 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1476 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1477 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1480 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1482 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1483 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1486 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1488 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1489 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1491 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1493 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1495 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1497 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1502 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1503 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1505 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1507 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1509 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1510 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1512 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1514 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1516 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1518 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1520 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1522 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1471    1524 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1472 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1478 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1479 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1481 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1484 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1485 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1487 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1490 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1492 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1494 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1496 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1498 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1499 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1500 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1501 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1504 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1506 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1508 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1511 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1513 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1515 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1517 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1519 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1521 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1523 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1525 ?        Sl     0:00 /usr/sbin/apache2 -k start
   1472    1526 ?        Sl     0:00 /usr/sbin/apache2 -k start
```


`/proc` ディレクトリ以下にはプロセスの情報がファイル管理されています。

`/proc/PID/task/` ディレクトリ以下には数字のディレクトリが並んでおり、各数字は先ほど`ps -T` で確認したスレッドIDを表します。

```
$ ls -F /proc/1471/task/
1471/  1475/  1476/  1477/  1480/  1482/  1483/  1486/  1488/  1489/  1491/  1493/  1495/  1497/  1502/  1503/  1505/  1507/  1509/  1510/  1512/  1514/  1516/  1518/  1520/  1522/  1524/
```

## (発展)CPUのスレッド

プロセスと関連したスレッドとは別に、｢ハイパースレッディング」というスレッドもよく耳にします。

CPUの物理的なプロセッサコアを複数の論理的なコアのようにみせかけることを一般にSimultaneous Multi-Threading(SMT)とよび、Intel社の技術を特別にハイパースレッディングと呼びます。

x86系のIntel/AMDのCPUはこの技術が備わっていて一般には有効化され、ARM系ではこの技術は採用されていません。

Amazon EC2のvCPU数は、論理コア数です。

そのため、Intel/AMDは物理コアの2倍であり、Arm(Graviton)は物理コア数と同じです。

```
$ egrep -E 'physical id|processor' /proc/cpuinfo
processor       : 0
physical id     : 0
processor       : 1
physical id     : 0
```

physical idは0しか存在しないため、物理コア数は1、processorは0と1が存在するため、論理コア数は2となります。

最新のCPU技術の進化に伴い、Intelは次世代CPU「Lunar Lake」から、一部のモデルでSMTを無効化すること発表しました。これは、SMTによるセキュリティ上の懸念や、電力効率の観点からの判断と言われています。

[Why Lunar Lake Won’t Use Hyper-Threading - YouTube](https://www.youtube.com/shorts/FZVUWvOdOdg)
