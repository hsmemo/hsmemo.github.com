---
layout: default
title: JNI の処理 ： Invocation API の処理 ： GetEnv() の処理 ： JavaThread：：jni_environment() の初期化処理 
---
[Up](no5248dl2.html) [Top](../index.html)

#### JNI の処理 ： Invocation API の処理 ： GetEnv() の処理 ： JavaThread：：jni_environment() の初期化処理 

--- 
## 概要(Summary)
JavaThread::jni_environment() の初期化は JavaThread オブジェクトの生成時に行われている.
使用する JNIEnv の選択は jni_functions() 関数で行われる.


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(See: <a href="no2935KMw.html">here</a> for details)
-&gt; JavaThread::initialize()
   -&gt; (1) 使用する JNIEnv を取得する
          -&gt; jni_functions()
             -&gt; 以下のどちらかをリターンする
                * jni_NativeInterface
                * checked_jni_NativeInterface (&lt;= jni_functions_check() の呼び出しにより取得)
      (1) JavaThread::jni_environment() に JNIEnv をセットする
          -&gt; JavaThread::set_jni_functions()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### jni_functions()
See: [here](no171196fz.html) for details
### jni_functions_check()
See: [here](no17119GFD.html) for details
### jni_functions_nocheck()
See: [here](no17119H4h.html) for details
### set_jni_functions()
See: [here](no17119tVt.html) for details






