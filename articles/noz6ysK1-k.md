---
layout: default
title: Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： GenCollectedHeap の場合 ： CMS の場合
---
[Up](noorGriS8G.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： GenCollectedHeap の場合 ： CMS の場合

--- 
## 概要(Summary)
CMS の場合(※), 
メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等)では, 
GenCollectedHeap::collect_mostly_concurrent() で処理が行われる.

この中では, VM_GenCollectFull クラスによって最終的に GenCollectedHeap::do_collection() が呼び出され, 
Minor GC または Major GC が実行される.

(※) より正確な条件は GenCollectedHeap::should_do_concurrent_full_gc() 参照 (See: [here](noorGriS8G.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
```
(See: [here](noorGriS8G.html) for details)
-> GenCollectedHeap::collect_mostly_concurrent()
   -> VMThread::execute()
      -> (略) (See: [here](no2935qaz.html) for details)
         -> VM_GenCollectFullConcurrent::doit()
            -> 
```

## 処理の流れ (詳細)(Execution Flows : Details)
### GenCollectedHeap::collect_mostly_concurrent()
See: [here](no28916M1p.html) for details
### VM_GenCollectFullConcurrent::doit()
(#Under Construction)







