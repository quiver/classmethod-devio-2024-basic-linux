# まとめ

regreSSHion を切り口にLinuxの世界に足を踏み入れました。

- シグナルハンドラーのレースコンディション
- sshd(SSHサーバー)
- root権限
- glibc

これらキーワードへの理解は深まったでしょうか?

Qualys による regreSSHion ブログの脆弱性の概要を再掲します。

> The vulnerability, which is a **signal handler race condition** in **OpenSSH’s server (sshd)**, allows unauthenticated remote code execution (RCE) as **root** on **glibc-based** Linux systems; that presents a significant security risk. This **race condition** affects **sshd** in its default configuration.
>
> [regreSSHion: Remote Unauthenticated Code Execution Vulnerability in OpenSSH server \| Qualys Security Blog](https://blog.qualys.com/vulnerabilities-threat-research/2024/07/01/regresshion-remote-unauthenticated-code-execution-vulnerability-in-openssh-server)

`SIGTERM` などのシグナルをハンドリングするシグナル・ハンドラーの実装では、`printf()` 関数などは使えず、非同期シグナル安全な関数のみ呼び出せるなど、制約が多いです。

SSHプロトコルのサーバーサイドプログラムである **sshd(OpenSSH server)** では、クライアントの認証時に `SIGALRM` タイマーで試行時間を制御し、特に **glibc** を使うLinux環境のシグナルハンドラー内では、内部的に `malloc()`/`free()` を呼び出す `syslog()` が使われていました。

複数の SSH 接続を試み、認証のタイムアウトを発生させ、シグナルハンドラー内の `malloc()` 呼び出しに別の非同期シグナルをうまく割り込ませるなどして、ヒープを不正な状態にすることで(**レースコンディション**)、**リモートコード実行(RCE)** のリスクがあります。

認証できていないため、*unauthenticated* なRCEです。


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

このパッチを適用すると、OpenBSD(Linuxとは別のUNIX系OS)のように、シグナルハンドラー内で再入可能な`syslog`関数(`syslog_r`など)が使われている場合のみ、ログ出力します。このミニマルパッチを適用後も、OpenBSDでの挙動は変わらず、Linux + glibc のように non-reentrant な `syslog()`を読んでいる場合、ログ出力されなくなります。

2006年にも類似の脆弱性があり([CVE-2006-5051](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-5051))、その際は、安全な `syslog()` 関数がある場合のみログ出力する判定(`#ifdef DO_LOG_SAFE_IN_SIGHAND`) が追加されました。

その後、2020年のリファクタリングの際に、誤ってこの判定が除外されてしまい、CVE-2006-5051 のバグが再発するようになっていました(再帰バグ=regression)。

以上が [CVE-2024-6387(regreSSHion)](https://nvd.nist.gov/vuln/detail/CVE-2024-6387) のあらましです。

## 次のステップ

今回のワークショップのような、OSを強く意識した世界に興味をもったら、以下の書籍がオススメです。

- 武内 覚 著『［試して理解］Linuxのしくみ』(技術評論社, 2022, ISBN 978-4-297-13148-7)
    - サンプルコードはPython。コマンドからOSを学ぶ側面が強い。
- 青木 峰郎著『ふつうのLinuxプログラミング 第2版』(SBクリエイティブ, 2017, ISBN 978-4-7973-8647-9)
    - サンプルコードはC。アプリケーション開発からOSを学ぶ側面が強い。
- 河田 旺、小池 悠生、渡邉 慶一、佐伯 学哉、荒田 実樹 著『河田 旺、小池 悠生、渡邉 慶一、佐伯 学哉、荒田 実樹 著』(オライリー・ジャパン、2024, ISBN 978-4-8144-0085-0)
    - 低レイヤーと友達になりたい

ぜひ、本屋で手にとって見てください。
