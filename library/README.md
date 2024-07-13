## ライブラリとは?

プログラミング言語は、文字列処理やファイル操作やネットワーク通信のような頻出操作をかんたnに行うためのライブラリを提供

## C言語とglibcの関係

C言語の場合、ISOで標準ライブラリが定められています。

glibc はGNUプロジェクトによる標準ライブラリの実装であり、Linux環境のデファクトスタンダードです。

Linuxでは、 `man` コマンドでシステムリファレンスマニュアルを読めます。
コマンドから `$ man glibc` を実行しましょう

```
$ man glibc  # あるいは $ man 7 libc


libc(7)                                            Miscellaneous Information Manual                                           libc(7)

NAME
       libc - overview of standard C libraries on Linux

DESCRIPTION
       The  term “libc” is commonly used as a shorthand for the “standard C library” a library of standard functions that can be used
       by all C programs (and sometimes by programs in other languages).  Because of some history (see below), use of the term “libc”
       to refer to the standard C library is somewhat ambiguous on Linux.

   glibc
       By far the most widely used C library on Linux is the GNU C Library, often referred to as glibc.  This is the C  library  that
       is  nowadays  used  in  all  major Linux distributions.  It is also the C library whose details are documented in the relevant
       pages of the man-pages project (primarily in Section 3 of the manual).  Documentation of glibc is also available in the  glibc
       manual,  available  via  the command info libc.  Release 1.0 of glibc was made in September 1992.  (There were earlier 0.x re‐
       leases.)  The next major release of glibc was 2.0, at the beginning of 1997.

       The pathname /lib/libc.so.6 (or something similar) is normally a symbolic link that points to the location of  the  glibc  li‐
       brary, and executing this pathname will cause glibc to display various information about the version installed on your system.
```

man ページは `q` で終了できます。

## 補足:man について

Linuxでは、 `man` コマンドでマニュアル(**man**nual)を読めます。

マニュアルの種類によって次の1から9のセクションが割り振られ、`$ man section page` というシンタックスで呼び出します。

1.   Executable programs or shell commands
2.   System calls (functions provided by the kernel)
3.   Library calls (functions within program libraries)
4.   Special files (usually found in /dev)
5.   File formats and conventions, e.g. /etc/passwd
6.   Games
7.   Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7), man-pages(7)
8.   System administration commands (usually only for root)
9.   Kernel routines [Non standard]

`$ man man` とすると `MAN(1)` に誘導されます。
これは、`man` はプログラムのため、セクション1に該当することを意味し、`$ man 1 man` と呼び出しても同じです。

ページだけではセクションが一意に定まらない例もあり、`$ man 1 printf` とすると、`printf` コマンドのページが表示され、`$ man 3 printf` とすると、`glibc` 等で実装されたC言語のライブラリ関数の `printf(3)` 関数のページが表示されます。

## glibc を使ってみる

C言語向けの print 関数である `printf(3)` は glibc のライブラリ関数であることを次の `hello.c` ファイルから確認します。。

```
#include <stdio.h>

int main() {
    printf("hello, world\n");
    return 0;
}
```

C言語のコンパイラである `gcc` でコンパイルします

```
$ gcc hello.c
$ ls
a.out  hello.c
$ ./a.out # 実行ファイル
hello, world
```

Cの実行ファイルの作成は次の2ステップからなります。

1. ソースコードからオブジェクトファイルの作成
2. オブジェクトファイルとライブラリのリンク

`$ gcc -o hello hello.c` はこの2処理を1ステップで行っています。

次のように2ステップに分割できます:

```
$ gcc -c hello.c -o hello.o # ステップ1 : オブジェクトファイルの作成
$ gcc hello.o -o hello      # ステップ2 : オブジェクトファイルとライブラリのリンク
$ ./hello
hello, world

$ file hello.o  # オブジェクトファイル
hello.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped

$ file hello    # 実行ファイル
hello: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=b3e50f2741351b23a37148846fc2a42a6394f609, for GNU/Linux 3.2.0, not stripped
```

特に、オブジェクトファイルやライブラリを結合して一つの実行ファイルに作成するプログラムをリンカと呼びます。`ld`(gcc デフォルト)、`lld`(LLVM)、`mold`(`lld`作者による次世代リンカ)などが有名です。

参考 [『リンカー moldをいろんなターゲットに移植した話』を視聴してCPUやpsABIの世界を覗き見してみた #kernelvm](https://zenn.dev/quiver/articles/7aa6deb2d77e44)


## 共有ライブラリと静的ライブラリ

ライブラリの利用方法は2種類

- 共有ライブラリ
    - プログラムがライブラリを参照する(動的リンク)
    - 多くのプログラミング言語が採用
    - Linuxだと `.so`ファイル、Windowsだと `.DLL`(Dynamic Link Library)
    - ライブラリとプログラムは1:多の関係
- 静的ライブラリ
    - プログラムにライブラリを組み込む(静的リンク)
    - Go言語が採用

## 共有ライブラリを確認

実行ファイルが依存する共有ライブラリは `ldd` プログラムで確認できます(`$ man 1 ldd`)

```
$ ldd ./a.out
        linux-vdso.so.1 (0x00007ffe02dcf000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007eb1a3e00000)
        /lib64/ld-linux-x86-64.so.2 (0x00007eb1a408e000)
```

`.so` は shared objects(共有ライブラリ)を表し、バイナリファイルからテキストを抜き出す `string` プログラムを利用し、この実体が glibc(GNU C Library) であることを確認します。

```
$ strings /lib/x86_64-linux-gnu/libc.so.6 | grep GNU
GNU C Library (Ubuntu GLIBC 2.39-0ubuntu8.2) stable release version 2.39.
Compiled by GNU CC version 13.2.0.
```

## Go言語は静的リンク

多くの言語はライブラリを動的リンクしますが、Go言語はデフォルトで静的リンクします。

次の `hello.go` ファイルを用意します。

```
package main

import "fmt"

func main() {
    fmt.Println("hello, world")
}
```

コンパイルして実行します。

```
$ go build hello.go
$ ./hello
hello, world
```

`ldd` や `file` から、静的リンク(`statically linked`)されていることがわかります。

```
$ ldd hello
        not a dynamic executable

$ file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, Go BuildID=A90LZtCkwgdOUoJtexOQ/iNVx11jHsmJ7aeiqlqKe/0EYraerkXrcvu0txJPvy/St2KmXJJMkh5V-u0vDo-, with debug_info, not stripped
```

## 静的ライブラリのメリット・デメリット

静的リンクすると実行ファイルを配布するだけ動作し、動的リンク時のように、実行環境でのライブラリの存在を意識しなくてすみます。

一方で、ライブラリをプログラムに直接組み込んでいるので、ファイルサイズが大きくなります。

"hello, world"を表示するだけのC言語とGo言語のファイルサイズを比較します。

```
$ ls -lh go/hello c/hello
-rwxrwxr-x 1 ubuntu ubuntu  16K Jul  9 03:52 c/hello
-rwxrwxr-x 1 ubuntu ubuntu 1.9M Jul  9 04:06 go/hello
```

動的リンクしたC版は16Kなのに対して、静的リンクしたGo版は1.9Mと100倍以上の開きがあります。

## コンテナでの静的リンクの活用例

コンテナでは、プログラムをビルドし、生成されたアーティファクトを実行する

```
FROM golang:1.21
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

CMD ["/bin/hello"]
```

このコンテナイメージには、Goのコンパイラやライブラリ群も含まれるため、サイズが肥大化してしまう。

コンテナのビルドには **マルチステージビルド** という手法があり、ビルド時と実行時でベースイメージを分け、ビルドフェーズで作成したアーティファクトだけを軽量なベースイメージで実行させることができる。

Goのように静的リンクされている場合、依存するライブラリはバイナリに組み込まれているため、マルチステージビルドと相性がよい。

```
FROM golang:1.21
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

※ Dockerfileの引用元 https://docs.docker.com/build/building/multi-stage/

## 発展:CPUアーキテクチャ

現在サーバー市場で主流のCPUはIntel/AMD系(x86)とArm系(arm)の2つの流派があり、CPUアーキテクチャと呼びます。

AWS Gravitionチップや最近のMacやiPhone/AndroidといったスマートフォンはArm系であり、Armより1世代前のMacや市場で入手可能なほとんどのサーバーはx86系です。

GoでコンパイルしたバイナリはLinuxならどこでも動くわけではなく、それぞれのCPUアーキテクチャ向けにコンパイルする必要があります。

## 発展:クロスコンパイル

CPUに限らず、異なるアーキテクチャをターゲットにコンパイルすることをクロスコンパイルと呼びます。

x86_64(x86の64ビット)環境で arm64(armの64ビット)環境向けにコンパイルする場合、次の様に `GOARCH=arm64` を渡します。

```
$ GOARCH=arm64 go build -o hello_arm64 hello.go

$ file hello_arm64
hello_arm64: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, Go BuildID=FZ2AWXSFUuSYLS3Mxpkr/SlETPZpre-ppkjqaPiD5/Ax0CehQj51JoVk4Pl26k/j5j7u9T703xJIpUt_JGq, with debug_info, not stripped
```

x86_64環境では、アーキテクチャが異なるため、実行できません

```
$ uname -m
x86_64

$ ./hello_arm64
-bash: ./hello_arm64: cannot execute binary file: Exec format error

$ file ./hello_arm64
./hello_arm64: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, Go BuildID=FZ2AWXSFUuSYLS3Mxpkr/SlETPZpre-ppkjqaPiD5/Ax0CehQj51JoVk4Pl26k/j5j7u9T703xJIpUt_JGq, with debug_info, not stripped
```

## コンテナのCPUアーキテクチャ

パブリッククラウド環境(アプリの実行環境)では、x86だけでなくAWS GravitonのようなArm系プロセッサを手軽に利用できます(例:AWS Lambda/AWS ECS Fargate)。
また、Arm系のMacをローカル開発環境にしている人も多く、[GitHub ActionsもArmに対応し](https://github.blog/2024-06-03-arm64-on-github-actions-powering-faster-more-efficient-build-systems/
)、結果的に、Armでビルドしたものをx86で実行したり、x86でビルドしたものをArmで実行するケースが増えています。

そのために、クロスコンパイルしたり、マルチプラットフォーム対応のコンテナイメージをビルドする機会が増えています。

## (発展)静的リンクされたAWS CLI(AWSのREST APIのコマンドラインツール)

AWSはAPIをコマンドラインから操作するためのAWS CLIというPython製のコマンドラインプログラムを提供しています。

https://docs.aws.amazon.com/cli/

Pythonインタープリターとプログラムの同梱が必要であることや起動が遅いことから、静的リンクしたシングルバイナリでAWS CLIを(部分的に)実装した人もいます。

[awslim - Goで実装された高速なAWS CLIの代替品を作った/layerx.go#1 - Speaker Deck](https://speakerdeck.com/fujiwara3/layerx-dot-go-number-1)

![](fujiwara_awslim.jpg)

このスライドでは、静的リンクによる起動時間やバイナリサイズやメモリへの影響や、マルチステージビルドのようなテクニックなども紹介されています。