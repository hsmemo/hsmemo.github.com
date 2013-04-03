---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ： Generation からのメモリの確保処理 ： UseSerialGC の場合
---
[Up](noHy8JYLbh.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ： Generation からのメモリの確保処理 ： UseSerialGC の場合

--- 
## 概要(Summary)
GC 中の Generation からのメモリ確保処理は, 
Generation::par_allocate() または Generation::allocate() によって行われる (See: [here](no28916sKh.html) for details).

UseSerialGC の場合, DefNewGeneration 及び TenuredGeneration の対応するメソッドが呼び出される.
どちらの場合も, 実際の確保処理は 
ContiguousSpace::par_allocate_impl() または ContiguousSpace::allocate_impl() で行われている.

## 処理の流れ (概要)(Execution Flows : Summary)
### DefNewGeneration の場合
#### Generation::par_allocate() の処理
```
DefNewGeneration::par_allocate()
-> EdenSpace::par_allocate()
   -> ContiguousSpace::par_allocate_impl()
```

#### Generation::allocate() の処理
```
DefNewGeneration::allocate()
-> (1) Eden 領域中からの確保を試みる. 成功すれば結果をリターン
       -> EdenSpace::par_allocate()
          -> (同上)
   (1) Eden 領域の soft limit を増加させて確保を試みる. 成功すれば結果をリターン
   (1) From 領域中からの確保を試みる
       -> DefNewGeneration::allocate_from_space()
          -> ContiguousSpace::allocate()
             -> ContiguousSpace::allocate_impl()
```

### TenuredGeneration の場合
#### Generation::allocate() の処理
```
OneContigSpaceCardGeneration::allocate()
-> ContiguousSpace::allocate()
   -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### DefNewGeneration::par_allocate()
See: [here](no3440Hf.html) for details
### EdenSpace::par_allocate()
See: [here](no21493-mu.html) for details
### ContiguousSpace::par_allocate_impl()
See: [here](no21493XPQ.html) for details
### DefNewGeneration::allocate()
See: [here](no344Ocr.html) for details
### DefNewGeneration::allocate_from_space()
See: [here](no21493_n1.html) for details
### OneContigSpaceCardGeneration::allocate()
See: [here](no21493wbh.html) for details
### ContiguousSpace::allocate()
See: [here](no21493ZD2.html) for details
### ContiguousSpace::allocate_impl()
See: [here](no21493lhR.html) for details






