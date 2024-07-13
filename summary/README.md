## TODO:まとめ

regreSSHion を切り口にLinuxの用語を振り返る

> The vulnerability, which is a **signal handler race condition** in **OpenSSH’s server (sshd)**, allows unauthenticated remote code execution (RCE) as **root** on **glibc-based** Linux systems; that presents a significant security risk. This **race condition** affects **sshd** in its default configuration.

キーワード回収できた?

## 参考

- 武内 覚 著『［試して理解］Linuxのしくみ』(技術評論社, 2022, ISBN 978-4-297-13148-7)
    - サンプルコードはPython主体です
- 青木 峰郎著『ふつうのLinuxプログラミング 第2版』(SBクリエイティブ, 2017, ISBN 978-4-7973-8647-9)
    - サンプルコードはCです
- 森田 浩平著『基礎から学ぶコンテナセキュリティ』(技術評論社, 2023, ISBN 978-4-297-13635-2)
    - コンテナ x セキュリティ x 低レイヤーのコンボです
- [The Twelve-Factor App](https://12factor.net/)
