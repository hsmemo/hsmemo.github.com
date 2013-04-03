---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： Shark JIT Compiler が生成したコードでの処理
---
[Up](no2114EV0.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： Shark JIT Compiler が生成したコードでの処理

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
```
-> SharkBuilder::CreateUpdateBarrierSet() が生成するコード
   -> 使用する GC アルゴリズムが G1GC かどうかに応じて 2通りの処理が存在
      * G1GC 以外の場合:
        -> (1) 書き換え箇所に対応する Barrier Set 中の値を dirty にする
               -> llvm::IRBuilder<>::CreateStore() が生成するコード

      * G1GC の場合:
        -> Unimplemented()
```

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)








