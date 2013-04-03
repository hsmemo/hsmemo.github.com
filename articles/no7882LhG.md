---
layout: default
title: (参考) ビルド後に生成されるファイル  
---
[Up](no3yhpM-MW.html) [Top](../index.html)

#### (参考) ビルド後に生成されるファイル  

--- 
## 概要
hotspot/ からは libjvm と gamma が生成される.
それ以外のもの全てが jdk/ 以下から生成される.

### hotspot/ 以下から生成されるもの

File Name                                     | Description
--------------------------------------------- | -----------------------------------------------------------------
libjvm                                        | hotspot 本体 (hotspot のメインルーチンが納められたライブラリ)
gamma                                         | テスト用の簡易的な java コマンド(launcher)

### jdk/ 以下から生成されるもの

File Name                                     | Description
--------------------------------------------- | -----------------------------------------------------------------
java                                          | java コマンド(launcher)
libjli                                        | java コマンド等の launcher の本体 (なお jli は "Java Launcher Infrastructure" の略). ここから libjvm が呼び出される (java コマンドなどは libjli の関数を呼び出しているだけ).
libjava                                       | Java の標準ライブラリが使用するネイティブメソッドを納めたライブラリ
libverify                                     | クラスファイルの verify 処理を格納したライブラリ (ただし, JDK 1.6 以降のクラスファイルでは libjvm 内に verify ルーチンが入っているためこれは使われない模様)
libjsig                                       | ネイティブコードのシグナルハンドリングと連携(「シグナル連鎖」)するためのライブラリ (See: [here](no7882MNx.html) for details)
libnpt                                        | NPT(Native Platform Toolkit) 用のライブラリ
libinstrument                                 | JPLIS(java.lang.instrument, javaagent) の実装が格納されているライブラリ (See: [here](no7882YrM.html) for details)
libmanagement                                 | JMM (Monitoring and Management Interface) 用のネイティブメソッドを納めたライブラリ (See: [here](no7882-WA.html) for details)
libattach                                     | Dynamic Attach 用のネイティブメソッドを納めたライブラリ (See: [here](no3026gMG.html) for details)
libsaproc                                     | Serviceability Agent (SA) 用のネイティブメソッドを納めたライブラリ (See: [here](no7882l1S.html) for details)
libfdlibm.${ARCH}.a	                      | java の strictfp や java.lang.StrictMath で必要とされる精度の数学関数を実装したライブラリ (なお, fdlibm は "Freely Distributable Math Library" の略).
libzip                                        | zip ファイル処理用のライブラリ (jar を開いたりするのに使用)
...(#Under Construction)                      | ...
libnet                                        | java.net パッケージ内のクラス用のネイティブライブラリ
libnio                                        | java.nio パッケージ内のクラス用のネイティブライブラリ
...(#Under Construction)                      | ...
rt.jar                                        | 標準ライブラリの class ファイルが納められた jar ファイル
...(#Under Construction)                      | ...
javac                                         | javac コマンド
javah                                         | javah コマンド
javap                                         | javap コマンド
...(#Under Construction)                      | ...







