---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： Exception イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： Exception イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(略) (See: <a href="no30593YX.html">here</a> for details)
-&gt; InterpreterRuntime::exception_handler_for_exception()
   -&gt; JvmtiExport::post_exception_throw()
      -&gt; (登録されているコールバックを呼び出す)

(略)
-&gt; Runtime1::post_jvmti_exception_throw()
   -&gt; JvmtiExport::post_exception_throw()
      -&gt; (同上)

(略)
-&gt; SharedRuntime::throw_and_post_jvmti_exception()
   -&gt; JvmtiExport::post_exception_throw()
      -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






