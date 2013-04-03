---
layout: default
title: ソースコードのディレクトリ構成(Source Code Directory Structure)
---
[Up](../index.html) [Top](../index.html)

#### ソースコードのディレクトリ構成(Source Code Directory Structure)

--- 


トップディレクトリの下には以下のディレクトリ(および Makefile)がある.

<!-- Turn-ON: (orgtbl-mode 1), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table29770T6K -->
| File Name | Description |
|---|---|
| hotspot/ | hotspot 本体 (libjvm) を作るための Makefile とソースコードを納めたディレクトリ |
| jdk/ | libjvm 以外のライブラリを作るためのファイル(Makefile, ソースコード), 及び Java の標準ライブラリ用のソースコード(.java ファイル)を納めたディレクトリ |
| langtools/ | language tools(javac, javadoc, javah, javap, apt 等)を作るための Makefile とソースコードを納めたディレクトリ |
| corba/ | Corba 用のソースコードを納めたディレクトリ |
| jaxws/ | JAXWS 用のソースコードを納めたディレクトリ (※1) |
| jaxp/ | JAXP 用のソースコードを納めたディレクトリ (※1) |
| Makefile | OpenJDK をビルドするための Makefile |
| make/ | OpenJDK をビルドするための Makefile 群を納めたディレクトリ (上記の Makefile 内から呼び出される) |
| test/ | OpenJDK のテスト用の Makefile 群を納めたディレクトリ |
| build/ | (ビルド実行後に作成されるディレクトリ) ビルドで構築されたオブジェクトファイル\&{}クラスファイルが配置される (※2) |
<!-- END RECEIVE ORGTBL table29770T6K -->

<!-- 
#+ORGTBL: SEND table29770T6K orgtbl-to-gfm
| File Name  | Description                                                                                                                                         |
|------------+-----------------------------------------------------------------------------------------------------------------------------------------------------|
| hotspot/   | hotspot 本体 (libjvm) を作るための Makefile とソースコードを納めたディレクトリ                                                                      |
| jdk/       | libjvm 以外のライブラリを作るためのファイル(Makefile, ソースコード), 及び Java の標準ライブラリ用のソースコード(.java ファイル)を納めたディレクトリ |
| langtools/ | language tools(javac, javadoc, javah, javap, apt 等)を作るための Makefile とソースコードを納めたディレクトリ                                        |
| corba/     | Corba 用のソースコードを納めたディレクトリ                                                                                                          |
| jaxws/     | JAXWS 用のソースコードを納めたディレクトリ (※1)                                                                                                    |
| jaxp/      | JAXP 用のソースコードを納めたディレクトリ (※1)                                                                                                     |
| Makefile   | OpenJDK をビルドするための Makefile                                                                                                                 |
| make/      | OpenJDK をビルドするための Makefile 群を納めたディレクトリ (上記の Makefile 内から呼び出される)                                                     |
| test/      | OpenJDK のテスト用の Makefile 群を納めたディレクトリ                                                                                                |
| build/     | (ビルド実行後に作成されるディレクトリ) ビルドで構築されたオブジェクトファイル&クラスファイルが配置される (※2)                                      |
-->


### 備考(Notes)
* ※1:
  jaxp と jaxws も JDK の一部だが, それぞれ独立したプロジェクトが開発しているためにこうなっている.
  jaxp や jaxws のソースは openjdk リポジトリに含まれていないので,
  ビルド時に ant がインターネット上から取得する.
  ただし将来的には変わる可能性あり.
* ※2:
  $(PLATFORM)-$(ARCH)-$(DEBUG_NAME)/j2sdk-image 以下に構築されたファイルが配置される.



## Subcategories
* [ソースコードのディレクトリ構成 ： hotspot ディレクトリ下](nooSr9ojgi.html)
* [ソースコードのディレクトリ構成 ： jdk ディレクトリ下](no2apwM4S0.html)



