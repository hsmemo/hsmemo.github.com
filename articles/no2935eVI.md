---
layout: default
title: Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： G1CollectedHeap の場合  
---
[Up](no28916_jv.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： G1CollectedHeap の場合  

--- 
## 概要(Summary)
メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等)では, 
最終的に CollectedHeap::collect() が呼び出される.

G1CollectedHeap は CollectedHeap::collect() をオーバーライドしているので, 
実際に呼び出されるのは G1CollectedHeap::collect() になる.
この中で Minor GC または Major GC が実行される.

## 処理の流れ (概要)(Execution Flows : Summary)
```
G1CollectedHeap::collect()
-> * Concurrent Full GC を実行すべき場合:
     -> VMThread::execute()
        -> (略) (See: [here](no2935qaz.html) for details)
           -> VM_G1IncCollectionPause::doit()
              -> (略) (See: [here](no2935YzN.html) for details)

   * Concurrent Full GC の必要はなく, 呼び出し元が GC_locker の場合:
     -> VMThread::execute()
        -> (略) (See: [here](no2935qaz.html) for details)
           -> VM_G1IncCollectionPause::doit()
              -> (略) (See: [here](no2935YzN.html) for details)

   * Concurrent Full GC の必要はなく, 呼び出し元が GC_locker 以外の場合:
     -> VMThread::execute()
        -> (略) (See: [here](no2935qaz.html) for details)
           -> VM_G1CollectFull::doit()
              -> G1CollectedHeap::do_full_collection()
                 -> G1CollectedHeap::do_collection()
                    -> (略) (See: [here](no2935ATn.html) for details)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### G1CollectedHeap::collect()
See: [here](no2935zBt.html) for details
### G1CollectedHeap::should_do_concurrent_full_gc()
See: [here](no2935_fI.html) for details
### VM_G1CollectFull::doit()
See: [here](no2935AMz.html) for details
### G1CollectedHeap::do_full_collection()
See: [here](no2935yVC.html) for details






