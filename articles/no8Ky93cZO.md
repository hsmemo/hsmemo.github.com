---
layout: default
title: PhaseCFG::GlobalCodeMotion() 用の補助クラス (Node_Backward_Iterator) 
---
[Top](../index.html)

#### PhaseCFG::GlobalCodeMotion() 用の補助クラス (Node_Backward_Iterator) 



---
## <a name="no5sn5afzH" id="no5sn5afzH">Node_Backward_Iterator</a>

### 概要(Summary)
PhaseCFG クラス内で使用される補助クラス.

グラフ内の Node をたどるためのイテレータクラス.

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* PhaseCFG::ComputeLatenciesBackwards()
* PhaseCFG::schedule_late()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
PhaseCFG::GlobalCodeMotion()
-&gt; PhaseCFG::ComputeLatenciesBackwards()
-&gt; PhaseCFG::schedule_late()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classNode__Backward__Iterator.html) for details

---
