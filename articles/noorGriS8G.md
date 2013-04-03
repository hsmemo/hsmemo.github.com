---
layout: default
title: Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： GenCollectedHeap の場合
---
[Up](no28916_jv.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： GenCollectedHeap の場合

--- 
## 概要(Summary)
メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等)では, 
最終的に CollectedHeap::collect() が呼び出される.

GenCollectedHeap は CollectedHeap::collect() をオーバーライドしているので, 
実際に呼び出されるのは GenCollectedHeap::collect() になる.
なお, GenCollectedHeap::collect() 内では CMS か否かに応じて処理が分岐しており, 実際に行われる処理は 2通り存在する
(See: [here](no28916YTF.html) and [here](noz6ysK1-k.html) for details)

## 処理の流れ (概要)(Execution Flows : Summary)
```
GenCollectedHeap::collect(GCCause::Cause cause)
-> * アルゴリズムが CMS で, かつ特定のオプション(※)がセットされている場合:
     -> GenCollectedHeap::collect_mostly_concurrent()
        -> (See: [here](noz6ysK1-k.html) for details)
   * それ以外の場合:
     -> GenCollectedHeap::collect(GCCause::Cause cause, int max_level)
        -> (See: [here](no28916YTF.html) for details)
```

(※) java.lang.System.gc() が原因の場合は ExplicitGCInvokesConcurrent オプション, 
JNI の ReleasePrimitiveArrayCritical() 及び ReleaseStringCritical() が原因の場合は GCLockerInvokesConcurrent オプションがセットされているとこちらになる.
それ以外の原因の場合にこちらになることはない
(See: GenCollectedHeap::should_do_concurrent_full_gc()).

## 処理の流れ (詳細)(Execution Flows : Details)
### GenCollectedHeap::collect(GCCause::Cause cause)
See: [here](no28916ygd.html) for details
### GenCollectedHeap::should_do_concurrent_full_gc()
See: [here](no28916Z_v.html) for details



## Subcategories
* [Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： GenCollectedHeap の場合 ： CMS ではない場合  ](no28916YTF.html)
* [Memory allocation (& GC 処理) ： メモリ確保以外の処理パスで発生する GC 処理 (java.lang.System.gc() 等) ： GenCollectedHeap の場合 ： CMS の場合](noz6ysK1-k.html)



