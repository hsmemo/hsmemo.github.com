---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： オブジェクトの処理(Object Operations) ： その他
---
[Up](noscE20YAT.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： オブジェクトの処理(Object Operations) ： その他

--- 
## 該当する JNI 関数
* `GetObjectClass`,
* `GetObjectRefType`,
* `IsInstanceOf`,
* `IsSameObject`,

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### GetObjectClass() の処理
<div class="flow-abst"><pre>
jni_GetObjectClass()
-&gt; Klass::java_mirror()
-&gt; JNIHandles::make_local()
   -&gt; (See: <a href="noNzTqB3WT.html">here</a> for details)
</pre></div>

### GetObjectRefType() の処理
<div class="flow-abst"><pre>
jni_GetObjectRefType()
-&gt; JNIHandles::is_local_handle()
   -&gt; JNIHandleBlock::chain_contains()
-&gt; JNIHandles::is_frame_handle()
-&gt; JNIHandles::is_global_handle()
   -&gt; JNIHandleBlock::chain_contains()
-&gt; JNIHandles::is_weak_global_handle()
   -&gt; JNIHandleBlock::chain_contains()
</pre></div>

### IsInstanceOf() の処理
<div class="flow-abst"><pre>
jni_IsInstanceOf()
-&gt; oopDesc::is_a()
   -&gt; (See: ...#TODO)
</pre></div>

### IsSameObject() の処理
<div class="flow-abst"><pre>
jni_IsSameObject()
-&gt; (ポインタ値を比較)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_GetObjectClass()
See: [here](no3059CcF.html) for details
### jni_GetObjectRefType()
See: [here](no3059cwR.html) for details
### JNIHandles::is_local_handle()
See: [here](no9282G6b.html) for details
### JNIHandleBlock::chain_contains()
See: [here](no92829zx.html) for details
### JNIHandles::is_frame_handle()
See: [here](no9282hPv.html) for details
### JNIHandles::is_global_handle()
See: [here](no92826po.html) for details
### JNIHandles::is_weak_global_handle()
See: [here](no9282gcQ.html) for details
### jni_IsInstanceOf()
See: [here](no3059PmL.html) for details
### jni_IsSameObject()
See: [here](no3059p6X.html) for details






