---
layout: default
title: java.lang.Thread オブジェクト (= JavaThread オブジェクト) の作成／開始／終了処理
---
[Up](noJb9iXZL-.html) [Top](../index.html)

#### java.lang.Thread オブジェクト (= JavaThread オブジェクト) の作成／開始／終了処理

--- 
## 概要(Summary)
Java の標準クラスである java.lang.Thread クラスには以下の 2通りの使われ方がある.

  * java.lang.Thread のサブクラスを作って run() をオーバーライドする方法: 

    java.lang.Thread.start() を呼び出すと, 
    最終的に java.lang.Thread.run() をオーバーライドしたメソッドが呼び出される.

  * java.lang.Thread() のコンストラクタに java.lang.Runnable オブジェクトを渡す方法: 

    java.lang.Thread.start() を呼び出すと, 
    最終的にはオーバーライドされていない java.lang.Thread.run() メソッドが呼び出される.
    
    この java.lang.Thread.run() メソッド内で渡された Runnable が呼び出される.

どちらの場合も, 処理の流れは大きく分けると以下の 2つからなる.

  * スレッドを作成する側(java.lang.Thread.start() を呼び出す側)の処理
  * 生成されたスレッド側の処理

### スレッドを作成する側(java.lang.Thread.start() を呼び出す側)の処理
  1. 適当な Thread オブジェクト (あるいは Thread のサブクラスのオブジェクト) を生成する.

  2. java.lang.Thread.start() を呼び出す.
  
     この時に, HotSpot 内で JavaThread オブジェクトが生成され,
     新しいネイティブスレッドの生成処理と開始処理が行われる.

### 生成されたスレッド側の処理
  1. スレッドの起動処理(全 Thread クラス共通部分)  (See: [here](no3059-9C.html) for details)

     生成されたスレッドは java_start() から実行が開始される.

     JavaThread は Thread::run() をオーバーライドしているので, 
     java_start() からは JavaThraed::run() が呼び出される.

  2. スレッドの起動処理(JavaThread 独自の部分)
  
     JavaThread::run() からは, 
     最終的に JavaThread オブジェクト生成時に指定された thread_entry() 関数が呼び出される.
     
     この thread_entry() 内で java.lang.Thread.run() が呼び出され, 生成されたスレッドのメイン処理が開始される.
     
  3. スレッドの終了処理
  
     生成されたスレッドのメイン処理が終わると, JavaThread::exit() による終了処理が実行される.



## Subcategories
* [java.lang.Thread オブジェクト (= JavaThread オブジェクト) を生成する側の処理 (1) ： `java.lang.Thread.<init>()` の処理](nokX0HkHvq.html)
* [java.lang.Thread オブジェクト (= JavaThread オブジェクト) を生成する側の処理 (2) ： `java.lang.Thread.start()` の処理  ](no2935KMw.html)
* [生成された java.lang.Thread (= JavaThread) 側での処理 (1) ： スレッドの起動処理(全 Thread クラス共通部分)](noSx64g0e3.html)
* [生成された java.lang.Thread (= JavaThread) 側での処理 (2) ： スレッドの起動処理(JavaThread 独自の部分)  ](no7882jgS.html)
* [生成された java.lang.Thread (= JavaThread) 側での処理 (3) ： スレッドの終了処理  ](no2935w3j.html)



