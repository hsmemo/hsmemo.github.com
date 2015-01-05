---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ExceptionCatch イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ExceptionCatch イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(略) (See: <a href="no30593YX.html">here</a> for details)
-&gt; InterpreterRuntime::exception_handler_for_exception()
   -&gt; JvmtiExport::notice_unwind_due_to_exception()
      -&gt; (登録されているコールバックを呼び出す)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






