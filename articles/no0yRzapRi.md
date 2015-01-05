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
<div class="flow-abst"><pre>
(See: <a href="no28916-pc.html">here</a> for details)
-&gt; SharedHeap::permanent_mem_allocate()
   -&gt; CompactingPermGen::mem_allocate()  または  CMSPermGen::mem_allocate()
      -&gt; PermGen::mem_allocate_in_gen()
         -&gt;
         -&gt; 確保に失敗したら VM_GenCollectForPermanentAllocation で GC を行う.
</pre></div>






