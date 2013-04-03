---
layout: default
title: Memory allocation (& GC 処理) ： Permanent 領域からの確保処理 ： SharedHeap の場合
---
[Up](no28916-pc.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Permanent 領域からの確保処理 ： SharedHeap の場合

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(See: [here](no28916-pc.html) for details)
-> SharedHeap::permanent_mem_allocate()
   -> CompactingPermGen::mem_allocate()  または  CMSPermGen::mem_allocate()
      -> PermGen::mem_allocate_in_gen()
         ->
         -> 確保に失敗したら VM_GenCollectForPermanentAllocation で GC を行う.
```






