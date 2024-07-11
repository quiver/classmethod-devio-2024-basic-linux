## 学ぶこと

- ライブラリ
- glibc
- 静的・動的
- gcc/file/ldd/stringsコマンド

## ライブラリとは?

多くのプログラミング言語は、文字列処理やファイル操作やネットワーク通信をカンタンに行うためのライブラリを提供


## C言語とglibcの関係

Linux OS自身や主要プログラムはC言語で書かれています。

C言語の場合、ISOで標準ライブラリが定められています。

glibc はGNUプロジェクトによる標準ライブラリの実装であり、Linux環境のデファクトスタンダードです。

Linuxでは、 `man` コマンドでシステムリファレンスマニュアルを読めます。
コマンドから `$ man glibc` を実行しましょう

```
$ man 7 libc


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

## C

Cの hello worldプログラムでライブラリの利用を確認します。

次の `hello.c` ファイルを用意します。

```
#include <stdio.h>

int main() {
    printf("hello, world\n");
    return 0;
}
```

gcc でコンパイルします

```
$ gcc hello.c
$ ls
a.out  hello.c
$ ./a.out
hello, world
```

Cの実行ファイルの作成は次の2ステップ

1. ソースコードからオブジェクトファイルの作成
2. オブジェクトファイルとライブラリのリンク

`$ gcc -o hello hello.c` はこの2処理を1ステップで行っています。

次のように2ステップに分割できます

```
$ gcc -c hello.c -o hello.o # ステップ1
$ gcc hello.o -o hello      # ステップ2
$ ./hello
hello, world

$ file hello.o  # オブジェクトファイル
hello.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped

$ file hello    # 実行ファイル
hello: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=b3e50f2741351b23a37148846fc2a42a6394f609, for GNU/Linux 3.2.0, not stripped
```

特に、オブジェクトファイルやライブラリを結合して一つの実行ファイルに作成するプログラムをリンカと呼びます。

リンカの例

- ld(GNU linkder)
    - gcc がデフォルトで利用
- lld
    - LLVM
    - clang/Swift などが利用
- mold(lldの作者による次世代リンカ)

## 共有ライブラリと静的ライブラリ

ライブラリの利用方法は2種類

- 共有ライブラリ
    - プログラムがライブラリを参照する
    - 多くのプログラミング言語が採用
    - Linuxだと `.so`ファイル、Windowsだと `.DLL`(Dynamic Link Library)
    - ライブラリとプログラムは1:多の関係
- 静的ライブラリ
    - プログラムにライブラリを組み込む
    - Go言語はデフォルトで静的リンク

### 共有ライブラリを確認

`ldd` コマンドでこのプログラムの利用しているライブラリを確認します

```
$ ldd ./a.out
        linux-vdso.so.1 (0x00007ffe02dcf000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007eb1a3e00000)
        /lib64/ld-linux-x86-64.so.2 (0x00007eb1a408e000)
```

`.so` は shared objects(共有ライブラリ)のことで、バイナリファイルからテキストを抜き出す `string` コマンドを利用し、この実体が glibc(GNU C Library) であることを確認します。

```
$ strings /lib/x86_64-linux-gnu/libc.so.6 | grep GNU
GNU C Library (Ubuntu GLIBC 2.39-0ubuntu8.2) stable release version 2.39.
Compiled by GNU CC version 13.2.0.
```

### Goで静的ライブラリを確認

Go言語は静的リンクすることを確認します。


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

様々なライブラリを利用している場合、実行ファイルを配布するだけよく、依存するライブラリのことを考えなくて済む。

一方で、ライブラリをプログラムに直接組み込んでいるので、ファイルサイズが大きくなる。

```
$ ls -lh go/hello c/hello
-rwxrwxr-x 1 ubuntu ubuntu  16K Jul  9 03:52 c/hello
-rwxrwxr-x 1 ubuntu ubuntu 1.9M Jul  9 04:06 go/hello
```

動的リンクのCの16Kに対して、静的リンクのGoは1.9Mと100倍以上の開き

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

参考

- https://docs.docker.com/build/building/multi-stage/

## (発展)CPUアーキテクチャ

現在サーバー市場で主流のCPUはIntel/AMD系(x86)とArm系(arm)の2つの流派があり、CPUアーキテクチャと呼びます。

AWS Gravitionチップや最近のMacはArm系であり、Armより1世代前のMacや市場で入手可能なほとんどのサーバーはx86系です。

Goで作成したバイナリはLinuxならどこでも動くわけではなく、それぞれのCPUアーキテクチャ向けにコンパイルする必要があります。

## (発展)クロスコンパイル

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

このスライドでは、静的リンクによる起動時間やバイナリサイズやメモリへの影響や、マルチステージビルドのようなテクニックなどが紹介されています。