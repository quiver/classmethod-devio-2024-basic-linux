## 学んだことを振りかえる

regreSSHion を切り口にLinuxの世界に足を踏み入れました。

regreSSHion ブログの脆弱性の概要を再掲します。

> The vulnerability, which is a **signal handler race condition** in **OpenSSH’s server (sshd)**, allows unauthenticated remote code execution (RCE) as **root** on **glibc-based** Linux systems; that presents a significant security risk. This **race condition** affects **sshd** in its default configuration.
>
> [regreSSHion: Remote Unauthenticated Code Execution Vulnerability in OpenSSH server \| Qualys Security Blog](https://blog.qualys.com/vulnerabilities-threat-research/2024/07/01/regresshion-remote-unauthenticated-code-execution-vulnerability-in-openssh-server)

- シグナルハンドラーのレースコンディション
- sshd(SSHサーバー)
- root権限
- glibc

これらキーワードへの理解は深まったでしょうか?

## 脆弱性のミニマルパッチを振り返る

この脆弱性を修正したエンジニアは、[脆弱性を回避するミニマルなパッチを提供しています](https://marc.info/?l=oss-security&m=171982317624594)

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

OpenBSD(Linuxとは別のUNIX系OS)のように、シグナルハンドラー内で再入可能な`syslog` 関数(`syslog_r`)が使われている場合のみ、ログ出力しています。このミニマルパッチを適用後も、OpenBSDでの挙動は変わりません。

一方で、 `glibc` (を使っているLinux OS)ではそのような関数は定義されておらず、再入可能でない `syslog` 関数がシグナルハンドラー内で呼ばれて脆弱性の温床になっていたため、そのような関数を呼ばなくしているのがミニマルパッチです。

※ 細かく言うと、`syslog_r` は ISOが定めるCの標準ライブラリには含まれていません

## 次のステップ

今回のワークショップのような、OSを強く意識した世界に興味をもったら、以下の書籍がオススメです。

- 武内 覚 著『［試して理解］Linuxのしくみ』(技術評論社, 2022, ISBN 978-4-297-13148-7)
    - サンプルコードはPython。コマンドからOSを学ぶ側面が強い。
- 青木 峰郎著『ふつうのLinuxプログラミング 第2版』(SBクリエイティブ, 2017, ISBN 978-4-7973-8647-9)
    - サンプルコードはC。アプリケーション開発からOSを学ぶ側面が強い。
- 河田 旺、小池 悠生、渡邉 慶一、佐伯 学哉、荒田 実樹 著『河田 旺、小池 悠生、渡邉 慶一、佐伯 学哉、荒田 実樹 著』(オライリー・ジャパン、2024, ISBN 978-4-8144-0085-0)
    - 低レイヤーと友達になりたい

ぜひ、本屋で手にとって見てください。
