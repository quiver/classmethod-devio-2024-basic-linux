# regreSSHionについて

## 概要

OpenSSH sshdというほぼすべてのLinuxサーバーにインストールされ、頻繁に使用されるSSHのサーバープログラム(`sshd`)に脆弱性(CVE-2024-6387)が発見されました。

[OpenSSH Vulnerability: CVE-2024-6387 FAQs and Resources | Qualys](https://www.qualys.com/regresshion-cve-2024-6387/)

2006年に修正された同じ sshd に関するCVE-2006-5051の回帰(regression)バグであることから、この脆弱性には "regression" と同じ発音で綴りに "SSH" を含めた regreSSHion という名前がつけられました。

[シグナルハンドラの競合状態(CWE-364)](https://cwe.mitre.org/data/definitions/364.html)に分類される脆弱性です。

regreSSHion の詳細は、次の日本語の解説記事をご確認ください

[OpenSSHの脆弱性 CVE-2024-6387についてまとめてみた - piyolog](https://piyolog.hatenadiary.jp/entry/2024/07/02/032122)

### CVEとは?

CVE(Common Vulnerabilities and Exposures)は脆弱性を一意な名前・IDで管理。

> The mission of the CVE® Program is to identify, define, and catalog publicly disclosed cybersecurity vulnerabilities. 
> https://www.cve.org/About/Overview

今回見つかったOpen sshdの脆弱性の場合、regreSSHion という名前と CVE-2024-6387 という管理番号が割り振られています。

CVEの取り組みにより、sshdの開発者もsshdを配布するベンダーもサーバーの運用者もCVE IDで会話できます。

## SSH とは?

セキュアシェル(SSH)はネットワーク経由で安全にシェルアクセスするプロトコルです。

SSH 実装として OpenSSHプロジェクトによるクライアント向けの `ssh` とサーバー向けの `sshd` が多くの環境で利用されています。

Windows環境で利用される `putty` は SSH プロトコルを実装した SSH クライアントプログラムの一つです(`putty` にははSSH以外にも様々なプロトコルが実装されています)。

![](ssh.png)

SSHの認証方式として以下があります。

- パスワード認証
- 公開鍵/秘密鍵ペアの公開鍵認証
    - [Amazon EC2が利用](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)
- クライアント認証
    - [Netflixの利用例](https://zenn.dev/quiver/articles/32ec71c3eedb2b)

GitHub.com のレポジトリでもSSH認証が利用されています。

[About SSH - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)

## コンテナへのシェルアクセス

SSHはサーバーの運用やトラブルシュートなどのために大昔から利用されています。

コンテナでは、コンテナランタイムがDockerの場合は `docker exec`、Container Runtime Interface (CRI)互換なランタイムの場合は `crictl exec` でシェルアクセス可能です。

```
$ crictl exec -i -t 12345 ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
```

AWSが提供するコンテナオーケストレーターのECSの場合、[ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html) という同等の機能が提供されています。

```
$ aws ecs execute-command --cluster cluster-name \
    --task task-id \
    --container container-name \
    --interactive \
    --command "/bin/sh"
```

## 参考

- [OpenSSHの脆弱性 CVE-2024-6387についてまとめてみた - piyolog](https://piyolog.hatenadiary.jp/entry/2024/07/02/032122)