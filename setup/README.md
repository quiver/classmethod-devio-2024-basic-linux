## 概要

今回のワークショップでは、ブラウザからカンタンにLinuxの開発環境を利用できるようにAWS Cloud9を利用し、AWS環境以外でもワークショップを実行できるように、ディストリビューションにはUbuntuを採用しています。

## 検証環境

- 検証環境 : AWS Cloud9
- CPU : x86_64
- OS : Linux
- ディストリビューション : Ubuntu 22.04

## AWS Cloud9とは?

[AWS Cloud9](https://aws.amazon.com/cloud9/)はブラウザから利用可能なIDE環境です。

クラウド(AWS)で実行されるVS Codeのようなものです。

## TODO:手順を追記

1． AWS Cloud9閉道
1. 東京リージョンを選択
1. 「環境を作成」※ 重要な設定項目
    - 環境タイプ: 新しい EC2 インスタンス
    - インスタンスタイプ:t3.small
    - プラットフォーム : Ubuntu Server 22.04 LTS


