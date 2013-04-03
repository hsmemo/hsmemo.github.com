---
layout: default
title: Runtime による Thread の一時停止処理 ： VMThread の生成処理
---
[Up](no2480eqy.html) [Top](../index.html)

#### Runtime による Thread の一時停止処理 ： VMThread の生成処理

--- 
## 概要(Summary)
VMThread は, HotSpot の初期化時に Threads::create_vm() の中で生成される.

VMThread の作成は os::create_thread() 及び os::start_thread() で行われる.
このため, 生成された VMThread スレッドは java_start() から実行が開始される (See: [here](noYHbL-pQM.html) for details).

java_start() から始まるスレッドの起動処理では最終的に Thread::run() が呼び出される (See: [here](no3059-9C.html) for details).
VMThread は Thread::run() をオーバーライドしているので, 
実際に呼び出されるのは VMThread::run() になる.
この VMThread::run() の中で実際の VM Operation の管理処理が行われる (See: [here](no2935qaz.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
### VMThread を作成する処理
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> VMThread::create()
      -> VMThread::VMThread()
         -> NamedThread::NamedThread()
            -> Thread::Thread()
      -> VMOperationQueue::VMOperationQueue()
   -> os::create_thread()
      -> (See: [here](noYHbL-pQM.html) for details)
   -> os::start_thread()
      -> (See: [here](noYHbL-pQM.html) for details)
```

### 生成された VMThread 側の処理
```
-> java_start()
   -> (See: [here](noaGdrH-zs.html), [here](noQiWP6ip-.html) and [here](nobwSeebST.html) for details)
      -> VMThread::run()
         -> (See: [here](no2935qaz.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### VMThread::create()
See: [here](no344LLu.html) for details
### VMThread::VMThread()
See: [here](no3059LIJ.html) for details
### NamedThread::NamedThread()
See: [here](no3059YSP.html) for details
### Thread::Thread()
See: [here](no2114wpW.html) for details
### VMOperationQueue::VMOperationQueue()
See: [here](no3059lcV.html) for details






