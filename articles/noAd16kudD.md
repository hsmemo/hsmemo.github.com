---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： C2 JIT Compiler が生成したコードでの処理
---
[Up](no2114EV0.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： C2 JIT Compiler が生成したコードでの処理

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
-&gt; GraphKit::pre_barrier() が生成するコード
   -&gt; 使用する GC アルゴリズムが G1GC かどうかに応じて 2通りの処理が存在
      * G1GC 以外の場合:
        -&gt; 何もしない

      * G1GC の場合:
        -&gt; GraphKit::g1_write_barrier_pre() が生成するコード
           -&gt; SharedRuntime::g1_wb_pre()
              -&gt; (同上)

-&gt; GraphKit::post_barrier() が生成するコード
   -&gt; 使用する GC アルゴリズムが G1GC かどうかに応じて 2通りの処理が存在
      * G1GC 以外の場合:
        -&gt; GraphKit::write_barrier_post() が生成するコード
           -&gt; 使用する GC アルゴリズムが CMS かどうかに応じて 2通りの処理が存在
              * CMS 以外の場合:
                -&gt; IdealKit::store() が生成するコード
                   -&gt; StoreBNode に対応するコード
              * CMS の場合:
                -&gt; IdealKit::storeCM() が生成するコード
                   -&gt; StoreCMNode に対応するコード

      * G1GC の場合:
        -&gt; GraphKit::g1_write_barrier_post() が生成するコード
           -&gt; GraphKit::g1_mark_card() が生成するコード
              -&gt; IdealKit::storeCM() が生成するコード
                 -&gt; StoreCMNode に対応するコード
              -&gt; SharedRuntime::g1_wb_post() (&lt;= もう空きスペースが無い場合に呼び出す)
                 -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### GraphKit::pre_barrier()
See: [here](no31977gkO.html) for details
### GraphKit::g1_write_barrier_pre()
(#Under Construction)


### GraphKit::post_barrier()
See: [here](no3718a2U.html) for details
### GraphKit::write_barrier_post()
See: [here](no3718nAb.html) for details
### GraphKit::g1_write_barrier_post()
(#Under Construction)

### GraphKit::g1_mark_card()
(#Under Construction)

### IdealKit::storeCM()
(#Under Construction)







