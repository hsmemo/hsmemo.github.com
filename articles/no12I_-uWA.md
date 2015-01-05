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
<div class="flow-abst"><pre>
-&gt; SharkBuilder::CreateUpdateBarrierSet() が生成するコード
   -&gt; 使用する GC アルゴリズムが G1GC かどうかに応じて 2通りの処理が存在
      * G1GC 以外の場合:
        -&gt; (1) 書き換え箇所に対応する Barrier Set 中の値を dirty にする
               -&gt; llvm::IRBuilder&lt;&gt;::CreateStore() が生成するコード

      * G1GC の場合:
        -&gt; Unimplemented()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)








