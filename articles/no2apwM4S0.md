---
layout: default
title: ソースコードのディレクトリ構成 ： jdk ディレクトリ下
---
[Up](noazh2rR49.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： jdk ディレクトリ下

--- 


File Name                                          | Description
-------------------------------------------------- | -----------------------------------------------------------------
jdk/make/                                          | ビルド用の Makefile が納められているディレクトリ
jdk/src/                                           | JDK 部のソースコード用のディレクトリ
- jdk/src/linux/                                   | OS に依存するソースコード&ドキュメント(manページ等)用のディレクトリ (linux 用)
- jdk/src/solaris/                                 | 〃 (solaris 用)
- jdk/src/windows/                                 | 〃 (windows 用)
- jdk/src/share/                                   | OS に依存しないソースコード用のディレクトリ
- - jdk/src/share/back/                            | JDWP プロトコルのバックエンド用のソースコード
- - jdk/src/share/bin/                             | CLI のコマンド (java コマンドなど) 用のソースコード
- - jdk/src/share/classes/                         | JDK のクラスファイル
- - jdk/src/share/demo/                            | applet や jfc のデモ
- - jdk/src/share/doc/                             | ドキュメント (README 等)
- - jdk/src/share/instrument/                      | JPLIS (libinstrument.so) 関連のソースコード
- - jdk/src/share/javavm/                          | HotSpot とクラス間で共有されるヘッダファイル (jni.h, jvm.h, jvmti.h, 等)
- - jdk/src/share/lib/                             | リソースファイル類(フォントファイル, .properties ファイル, 画像ファイル, 等) <= .properties ファイルは(例えばjava.util.logging等の)設定ファイル
- - jdk/src/share/native/                          | java のクラスから(JNI 経由で)呼び出されるネイティブコードのソースコード (java/lang/Class.c 等)
- - jdk/src/share/npt/                             | libnpt (NPT:Native Platform Toolkit) 用のソースコード
- - jdk/src/share/sample/                          | サンプル集
- - jdk/src/share/transport/                       | プロセス間通信用のソースコード?? (shmem と socket という名前のディレクトリがあり, それぞれにソースコードがいくつか入っている #TODO)

=






