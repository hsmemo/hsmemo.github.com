---
layout: default
title: Thread の開始処理の枠組み ： 生成されたスレッド側での動き ： Solaris の場合
---
[Up](no3059-9C.html) [Top](../index.html)

#### Thread の開始処理の枠組み ： 生成されたスレッド側での動き ： Solaris の場合

--- 
## 概要(Summary)
生成されたスレッド側の処理の流れは以下のようになる.

1. 実行が開始されると java_start() 関数から処理が始まる.

2. java_start() からは最終的に Thread::run() が呼び出される.
   
   (なお, java_start() は各 OS 毎に実装されている.
   ただし, どの場合でも最終的には Thread::run() に行き着く)

3. この Thread::run() は virtual function であり, 
   実際には Thread の各サブクラスでオーバーライドされた run() メソッドが実行される.

## 処理の流れ (概要)(Execution Flows : Summary)
```
java_start()
-> (1) LWP 関係の設定
       -> _lwp_self()
       -> OSThread::set_lwp_id()

   (1) NUMA 関係の設定
       -> os::numa_get_group_id()
       -> Thread::set_lgrp_id()

   (1) スレッドの優先度の設定
       -> os::set_native_priority()

   (1) シグナルマスクの設定
       -> os::Solaris::hotspot_sigmask()
          -> (See: [here](noNmlmYDJk.html) for details)

   (1) 実際にこのスレッドのメイン処理を実行
       -> Thread::run()
          -> (Thread の各サブクラスでオーバーライドされた run() メソッドが呼び出される)

   (1) 終了
       -> thr_exit() (<= UseDetachedThreads オプションが指定されている場合には呼び出す.
                         この場合, スレッド自体が消えるのでここで実行は終了.
                         そうでなければ単にリターンすることで終了させる.)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### java_start() (Solaris の場合)
See: [here](no305991H.html) for details
### OSThread::set_lwp_id()
See: [here](no3059M0z.html) for details
### os::numa_get_group_id()  (Solaris の場合)
(#TODO)

### Thread::set_lgrp_id()
See: [here](no3059-hy.html) for details
### os::set_native_priority()  (Solaris の場合)
See: [here](no2114YQk.html) for details








