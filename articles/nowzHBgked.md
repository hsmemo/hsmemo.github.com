---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： VMDeath イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： VMDeath イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(HotSpot の正常終了処理) (See: <a href="no3059oro.html">here</a> for details)
-&gt; Threads::destroy_vm()
   -&gt; before_exit()
      -&gt; JvmtiExport::post_vm_death()
         -&gt; (登録されているコールバックを呼び出す)

(略) (?? 使われていない #TODO)
-&gt; JVM_Exit()
   -&gt; before_exit()
      -&gt; (同上)

(java.lang.System.exit() による終了処理) (See: <a href="no2935jtd.html">here</a> for details)
-&gt; JVM_Halt()
   -&gt; before_exit()
      -&gt; (同上)

(略)
-&gt; CompileBroker::handle_full_code_cache()
   -&gt; before_exit()  (ただし, 呼び出すのは #ifndef PRODUCT 時のみ)
      -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






