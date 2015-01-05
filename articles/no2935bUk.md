---
layout: default
title: JNI の処理 ： Invocation API の処理 ： GetEnv() の処理 ： GetEnv() による JNIEnv 取得処理  
---
[Up](no5248dl2.html) [Top](../index.html)

#### JNI の処理 ： Invocation API の処理 ： GetEnv() の処理 ： GetEnv() による JNIEnv 取得処理  

--- 
## 概要(Summary)
JNI の GetEnv() 関数は, 引数で指定されたバージョンに応じて以下のどれかを返す.

  * 指定されたバージョンが JVMTI のものである場合:

    JvmtiExport::get_jvmti_interface() で jvmtiEnv を取得してリターン (See: [here](no30592Ee.html) for details)

  * 指定されたバージョンが JNI のものである場合:

    JavaThread::jni_environment() で JNIEnv を取得してリターン.

  * それ以外の場合:

    エラー


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
jni_GetEnv()
-&gt; JvmtiExport::is_jvmti_version()
   -&gt; (See: <a href="no30592Ee.html">here</a> for details)
-&gt; JvmtiExport::get_jvmti_interface()
   -&gt; (See: <a href="no30592Ee.html">here</a> for details)
-&gt; Threads::is_supported_jni_version_including_1_1()
   -&gt; Threads::is_supported_jni_version()
-&gt; JavaThread::jni_environment()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_GetEnv()
See: [here](no17119fYI.html) for details
### Threads::is_supported_jni_version_including_1_1()
See: [here](no17119TBh.html) for details
### Threads::is_supported_jni_version()
See: [here](no17119Gwm.html) for details
### JavaThread::jni_environment()
See: [here](no3059DPk.html) for details






