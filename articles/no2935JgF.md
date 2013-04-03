---
layout: default
title: Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： ParallelScavengeHeap の場合  
---
[Up](no28916_jv.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： ParallelScavengeHeap の場合  

--- 
## 概要(Summary)
メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等)では, 
最終的に CollectedHeap::collect() が呼び出される.

ParallelScavengeHeap は CollectedHeap::collect() をオーバーライドしているので, 
実際に呼び出されるのは ParallelScavengeHeap::collect() になる.
この中で VM_ParallelGCSystemGC によって Minor GC か Major GC のどちらかが実行される.

## 処理の流れ (概要)(Execution Flows : Summary)
```
ParallelScavengeHeap::collect()
-> VMThread::execute()
   -> (略) (See: [here](no2935qaz.html) for details)
      -> VM_ParallelGCSystemGC::doit()
         * 呼び出し元が GC_locker の場合:
           -> ParallelScavengeHeap::invoke_scavenge()
              -> PSScavenge::invoke()
                 -> (See: [here](no289165Un.html) for details)
         * そうでない場合:
           -> ParallelScavengeHeap::invoke_full_gc()
              -> (See: [here](no3718vrX.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### ParallelScavengeHeap::collect()
See: [here](no28916-3E.html) for details
### VM_ParallelGCSystemGC::doit()
See: [here](no28916LCL.html) for details
### ParallelScavengeHeap::invoke_scavenge()
See: [here](no28916YMR.html) for details






