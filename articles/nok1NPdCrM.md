---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： JNI による例外処理(Exceptions)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： JNI による例外処理(Exceptions)

--- 
## 該当する JNI 関数
* `Throw`,
* `ThrowNew`,
* `ExceptionOccurred`,
* `ExceptionDescribe`,
* `ExceptionClear`,
* `FatalError`,
* `ExceptionCheck`,

## 概要(Summary)
(#Under Construction)


## 処理の流れ (概要)(Execution Flows : Summary)
### Throw() の処理
<div class="flow-abst"><pre>
jni_Throw()
-&gt; THROW_OOP_()
   -&gt; (See: <a href="no3059qOR.html">here</a> for details)
</pre></div>

### ThrowNew() の処理
<div class="flow-abst"><pre>
jni_ThrowNew()
-&gt; THROW_MSG_LOADER_()
   -&gt; (See: <a href="no3059qOR.html">here</a> for details)
</pre></div>

### ExceptionOccurred() の処理
<div class="flow-abst"><pre>
jni_ExceptionOccurred()
-&gt; jni_check_async_exceptions()
   -&gt; JavaThread::check_and_handle_async_exceptions()
-&gt; ThreadShadow::pending_exception()
</pre></div>

### ExceptionDescribe() の処理
<div class="flow-abst"><pre>
jni_ExceptionDescribe()
-&gt; JavaCalls::call_virtual()
   -&gt; (See: <a href="no3059iJu.html">here</a> for details)
      -&gt; java.lang.Throwable.printStackTrace()
</pre></div>

### ExceptionClear() の処理
<div class="flow-abst"><pre>
jni_ExceptionClear()
-&gt; ThreadShadow::clear_pending_exception()
</pre></div>

### FatalError() の処理
<div class="flow-abst"><pre>
jni_FatalError()
-&gt; os::abort()
</pre></div>

### ExceptionCheck() の処理
<div class="flow-abst"><pre>
jni_ExceptionCheck()
-&gt; jni_check_async_exceptions()
   -&gt; JavaThread::check_and_handle_async_exceptions()
-&gt; ThreadShadow::has_pending_exception()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_Throw()
See: [here](no3059OJ0.html) for details

### jni_ThrowNew()
See: [here](no3059ATD.html) for details

### jni_ExceptionOccurred()
See: [here](no3059NdJ.html) for details
### jni_check_async_exceptions()
See: [here](no3059BGi.html) for details
### JavaThread::check_and_handle_async_exceptions()
(#Under Construction)
See: [here](no3059OQo.html) for details

### jni_ExceptionDescribe()
See: [here](no3059anP.html) for details

### jni_ExceptionClear()
See: [here](no3059nxV.html) for details

### jni_FatalError()
See: [here](no305907b.html) for details

### jni_ExceptionCheck()
See: [here](no3059bau.html) for details






