# サーバーレスエンジニアのための体験型Linux入門

## このハンズオンについて

これは、クラスメソッド株式会社の [DevelopersIO 2024 Odyssey](https://classmethod.jp/m/odyssey/) の2024年7月20日に開催された次の Linux ワークショップのために作成されたものです。

## サーバーレスエンジニアのための体験型Linux入門

サーバーレス技術の普及に伴い、エンジニアがOSを意識する機会は大幅に減りました。とはいえ、そのようなエンジニアが技術領域を広げるために、サーバーレスの対極にあるサーバーフルな世界に足を踏み入れる選択肢があってもよいでしょう。

2024年7月に`sshd`に関する脆弱性[regreSSHion(CVE-2024-6387)](https://unit42.paloaltonetworks.com/threat-brief-cve-2024-6387-openssh/)が公開されました。

![](Q-regreSSHion.jpg)

脆弱性を発見した Qualys のエンジニアブログでは、regreSSHion について以下のような解説があります。

> The vulnerability, which is a **signal handler race condition** in **OpenSSH’s server (sshd)**, allows unauthenticated remote code execution (RCE) as **root** on **glibc-based** Linux systems; that presents a significant security risk. This **race condition** affects **sshd** in its default configuration.
>
> [regreSSHion: Remote Unauthenticated Code Execution Vulnerability in OpenSSH server \| Qualys Security Blog](https://blog.qualys.com/vulnerabilities-threat-research/2024/07/01/regresshion-remote-unauthenticated-code-execution-vulnerability-in-openssh-server)

脆弱性の解説記事に現れるこれらキーワードをサーバーレスエンジニア向けに体験してもらうのが、本ワークショップの狙いです。

この脆弱性を修正したエンジニアは、[脆弱性を回避するミニマルなパッチも提供しています](https://marc.info/?l=oss-security&m=171982317624594)

```
1) Critical race condition in sshd

diff --git a/log.c b/log.c
index 9fc1a2e2e..191ff4a5a 100644
--- a/log.c
+++ b/log.c
@@ -451,12 +451,14 @@ void
 sshsigdie(const char *file, const char *func, int line, int showfunc,
     LogLevel level, const char *suffix, const char *fmt, ...)
 {
+#ifdef SYSLOG_R_SAFE_IN_SIGHAND
 	va_list args;
 
 	va_start(args, fmt);
 	sshlogv(file, func, line, showfunc, SYSLOG_LEVEL_FATAL,
 	    suffix, fmt, args);
 	va_end(args);
+#endif
 	_exit(1);
 }
```

`syslog_r` 関数が定義されている場合のみ `sshlogv` というログ出力する関数を呼び出しています。

OpenSSHの開発母体とも言える、[OpenBSD](https://www.openbsd.org/)というセキュリティを強く意識したBSD系OSではこの `syslog_r` 関数が定義されています。`syslog_r` を[マニュアルを抜粋すると、](https://man.openbsd.org/syslog.3)以下のとおりです。

> The syslog_r() function is a reentrant version of the syslog() function. It takes a pointer to a syslog_data structure which is used to store information. This parameter must be initialized before syslog_r() is called. The SYSLOG_DATA_INIT constant is used for this purpose.
> 
> ...
> 
> **CAVEATS**
> 
> syslog_r() and the other reentrant functions should only be used where reentrancy is required (for instance, in a signal handler). syslog() being not reentrant, only syslog_r() should be used here. For more information about reentrancy and signal handlers, see signal(3).

シグナルハンドラー内では reentrant な syslog 関数が定義されている場合のみ、ログ出力するように修正しているわけです。

ワークショップを終えると、この意味が理解できるようになるはずです。

## 目次

- [環境セットアップ](setup/README.md)
- [regreSSHionとは](regresshion/README.md)
- [ルート権限](root/README.md)
- [ライブラリ](library/README.md)
- [プロセス](process/README.md)
- [スレッド](thread/README.md)
- [シグナル](signal/README.md)
- [まとめ](summary/README.md)
