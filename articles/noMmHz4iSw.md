---
layout: default
title: Serviceability 機能 ： OS のプロファイラ機能との連携 ： OProfile
---
[Up](no3ZCVJmJ2.html) [Top](../index.html)

#### Serviceability 機能 ： OS のプロファイラ機能との連携 ： OProfile

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 備考(Notes)
この機能を使用するには UseOprofile オプションをセットする必要がある.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(See: )
-&gt; CodeHeap::reserve()
   -&gt; CodeHeap::on_code_mapping()
      -&gt; linux_wrap_code()

(See: )
-&gt; CodeHeap::expand_by()
   -&gt; CodeHeap::on_code_mapping()
      -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)







