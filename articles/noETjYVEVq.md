---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ThreadStart イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ThreadStart イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; JNI_CreateJavaVM()
   -&gt; JvmtiExport::post_thread_start()
      -&gt; (登録されているコールバックを呼び出す)

(略) (See: <a href="no7882jgS.html">here</a> for details)
-&gt; JavaThread::run()
   -&gt; JvmtiExport::post_thread_start()
      -&gt; (同上)

(略) (See: <a href="noxegGjntv.html">here</a> for details)
-&gt; jni_AttachCurrentThread()
   -&gt; attach_current_thread()
      -&gt; JvmtiExport::post_thread_start()
         -&gt; (同上)

(略) (See: <a href="noxegGjntv.html">here</a> for details)
-&gt; jni_AttachCurrentThreadAsDaemon()
   -&gt; attach_current_thread()
      -&gt; JvmtiExport::post_thread_start()
         -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






