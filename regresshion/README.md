## regreSSHion(CVE-2024-6387)について

OpenSSH sshdというほぼすべてのLinuxサーバーにインストールされ、頻繁に使用されるプログラムに脆弱性がQualysのエンジニアによって発表されました。

[OpenSSH Vulnerability: CVE-2024-6387 FAQs and Resources | Qualys](https://www.qualys.com/regresshion-cve-2024-6387/)

2006年に修正された同じ sshd に関するCVE-2006-5051の回帰(regression)バグであることから、この脆弱性には "regression" と同じ発音で綴りに "SSH" を含めた regreSSHion という名前がつけられました。

[シグナルハンドラの競合状態(CWE-364)](https://cwe.mitre.org/data/definitions/364.html)に分類される脆弱性です。

regreSSHion の詳細は、次の日本語の解説記事をご確認ください

[OpenSSHの脆弱性 CVE-2024-6387についてまとめてみた - piyolog](https://piyolog.hatenadiary.jp/entry/2024/07/02/032122)

## SSH とは?

セキュアシェル(SSH)はネットワーク経由で安全にシェルアクセスするプロトコルです。

SSH 実装として OpenSSHプロジェクトによるクライアント向けの `ssh` とサーバー向けの `sshd` が多くの環境で利用されています。

Windows環境で利用される `putty` は SSH プロトコルを実装した SSH クライアントプログラムの一つです(`putty` にははSSH以外にも様々なプロトコルが実装されています)。

![](ssh.png)

## SSH の認証方式

SSHの認証方式として以下があります。

- パスワード認証
- 公開鍵/秘密鍵ペアの公開鍵認証
- クライアント認証

GitHub.com のレポジトリでもSSH認証が利用されています。

https://docs.github.com/ja/authentication/connecting-to-github-with-ssh/about-ssh

TODO:ここにレポジトリのキャプチャ

## TODO:コンテナへのシェルアクセス

コンテナにSSHは不要

docker execを利用しよう。

AWS ECSだと[ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html)

## 参考

- [OpenSSHの脆弱性 CVE-2024-6387についてまとめてみた - piyolog](https://piyolog.hatenadiary.jp/entry/2024/07/02/032122)