# セットアップ

## 概要

このワークショップでは、ブラウザからカンタンにLinuxの開発環境を利用できるように統合開発環境の[AWS Cloud9](https://aws.amazon.com/cloud9/)を利用し、AWS環境以外でもワークショップを実行できるように、ディストリビューションにはUbuntuを採用しています。

## 検証環境

- 検証環境 : AWS Cloud9
- CPU : x86_64
- OS : Linux
- ディストリビューション : Ubuntu 22.04

## AWS Cloud9とは?

[AWS Cloud9](https://aws.amazon.com/cloud9/)はブラウザから利用可能なIDE環境です。

クラウド(AWS)で実行されるVS Codeのようなものです。


## AWS コンソールログイン手順

1. 指定のURLでAWSコンソールにアクセス
2. ユーザー名、初期パスワードを入力し、新しいパスワードを設定

## Cloud9 構築手順

1. AWS Cloud9へ移動
1. 東京リージョンを選択
1. 「環境を作成」※ 重要な設定項目
    - 環境タイプ: 新しい EC2 インスタンス
    - インスタンスタイプ:t3.small
    - プラットフォーム : Ubuntu Server 22.04 LTS


## Cloud9 でワークショップコンテンツをGit Clone

```
~/environment $ git clone https://github.com/quiver/classmethod-devio-2024-basic-linux.git
...
~/environment $ ls
README.md  classmethod-devio-2024-basic-linux
```
