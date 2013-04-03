---
layout: default
title: Thread の開始処理の枠組み ： 生成されたスレッド側での動き ： Windows の場合
---
[Up](no3059-9C.html) [Top](../index.html)

#### Thread の開始処理の枠組み ： 生成されたスレッド側での動き ： Windows の場合

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
-> (1) NUMA 関係の設定
       -> os::numa_get_group_id()
       -> Thread::set_lgrp_id()

   (1) 実際にこのスレッドのメイン処理を実行
       -> Thread::run()
          -> (Thread の各サブクラスでオーバーライドされた run() メソッドが呼び出される)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### java_start() (Windows の場合)
See: [here](no3059xeg.html) for details
### os::numa_get_group_id()  (Windows の場合)
(#TODO)

### Thread::set_lgrp_id()
See: [here](no3059-hy.html) for details






