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



## 次のステップ

今回のワークショップのような、OSを強く意識した世界に興味をもったなら、以下の書籍がオススメです。

- 武内 覚 著『［試して理解］Linuxのしくみ』(技術評論社, 2022, ISBN 978-4-297-13148-7)
    - サンプルコードはPython。コマンドからOSを学ぶ側面が強い。
- 青木 峰郎著『ふつうのLinuxプログラミング 第2版』(SBクリエイティブ, 2017, ISBN 978-4-7973-8647-9)
    - サンプルコードはC。アプリケーション開発からOSを学ぶ側面が強い。
- 河田 旺、小池 悠生、渡邉 慶一、佐伯 学哉、荒田 実樹 著『河田 旺、小池 悠生、渡邉 慶一、佐伯 学哉、荒田 実樹 著』(オライリー・ジャパン、2024, ISBN 978-4-8144-0085-0)
    - 低レイヤーと友達になりたい

本屋で手にとって見てください。
