---
layout: default
title: カレントスレッドを表す Thread オブジェクトの取得処理 (java.lang.Thread.currentThread() の処理)
---
[Up](nokB2etACd.html) [Top](../index.html)

#### カレントスレッドを表す Thread オブジェクトの取得処理 (java.lang.Thread.currentThread() の処理)

--- 
## 概要(Summary)
JVM_ENTRY マクロ内でカレントスレッドを 
JavaThread::thread_from_jni_environment() によって取得しているので, 
それを JNI Handle 化して返すだけ.


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JVM_CurrentThread() (= java.lang.Thread.currentThread())
-&gt; JavaThread::threadObj()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### JVM_CurrentThread()
See: [here](no2114XWR.html) for details
#### 備考(Notes)
java.lang.Thread.currentThread() は JVM_CurrentThread() で実装されている.


```cpp
    ((cite: jdk/src/share/native/java/lang/Thread.c))
    static JNINativeMethod methods[] = {
    ...
        {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
```


### JavaThread::threadObj()
See: [here](no2114kgX.html) for details






