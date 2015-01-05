---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： DataDumpRequest イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： DataDumpRequest イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(略) (?? 使われていない #TODO)
-&gt; JVM_DumpAllStacks()
   -&gt; JvmtiExport::post_data_dump()
      -&gt; (登録されているコールバックを呼び出す)

(略) (See: <a href="no28916GhX.html">here</a> for details)
-&gt; signal_thread_entry()
   -&gt; JvmtiExport::post_data_dump()
      -&gt; (同上)

(略) (See: <a href="no3026gMG.html">here</a> for details)
-&gt; attach_listener_thread_entry()
   -&gt; data_dump()
      -&gt; JvmtiExport::post_data_dump()
         -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






