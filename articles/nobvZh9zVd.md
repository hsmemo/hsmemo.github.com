---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ResourceExhausted イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ResourceExhausted イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(略) (See: <a href="no2935KMw.html">here</a> for details)
-&gt; JVM_StartThread()
   -&gt; JvmtiExport::post_resource_exhausted()
      -&gt; (登録されているコールバックを呼び出す)

(略) (See: <a href="no28916Q0G.html">here</a> for details)
-&gt; CollectedHeap::common_mem_allocate_noinit()
   -&gt; JvmtiExport::post_resource_exhausted()
      -&gt; (登録されているコールバックを呼び出す)

(略) (See: <a href="no28916-pc.html">here</a> for details)
-&gt; CollectedHeap::common_permanent_mem_allocate_noinit()
   -&gt; JvmtiExport::post_resource_exhausted()
      -&gt; (登録されているコールバックを呼び出す)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






