---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： オブジェクトの処理(Object Operations) ： メモリ確保(オブジェクトの確保)  
---
[Up](noscE20YAT.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： オブジェクトの処理(Object Operations) ： メモリ確保(オブジェクトの確保)  

--- 
## 該当する JNI 関数
* `AllocObject`,
* `NewObject`,
* `NewObjectV`,
* `NewObjectA`,


## 概要(Summary)
どの関数も, 基本的には instanceKlass::allocate_instance() でメモリを確保し,
JNIHandles::make_local() で JNI Handle 化した後, 
jni_invoke_nonstatic() でコンストラクタを呼び出すだけ.

## 備考(Notes)
AllocObject() の場合はコンストラクタ呼び出しはない. これは JNI の仕様に沿った挙動.


## 処理の流れ (概要)(Execution Flows : Summary)
### AllocObject() の処理
<div class="flow-abst"><pre>
jni_AllocObject()
-&gt; alloc_object()
   -&gt; instanceKlass::check_valid_for_instantiation()
   -&gt; instanceKlass::allocate_instance()
      -&gt; (See: <a href="no30267vB.html">here</a> for details)
-&gt; JNIHandles::make_local()
   -&gt; (See: <a href="noNzTqB3WT.html">here</a> for details)
</pre></div>

### NewObject() の処理
<div class="flow-abst"><pre>
jni_NewObject()
-&gt; alloc_object()
   -&gt; (同上)
-&gt; JNIHandles::make_local()
   -&gt; (See: <a href="noNzTqB3WT.html">here</a> for details)
-&gt; jni_invoke_nonstatic()
   -&gt; (See: <a href="no3059-0k.html">here</a> for details)
      -&gt; (指定されたコンストラクタメソッド)
</pre></div>

### NewObjectV() の処理
<div class="flow-abst"><pre>
jni_NewObjectV()
-&gt; alloc_object()
   -&gt; (同上)
-&gt; JNIHandles::make_local()
   -&gt; (See: <a href="noNzTqB3WT.html">here</a> for details)
-&gt; jni_invoke_nonstatic()
   -&gt; (See: <a href="no3059-0k.html">here</a> for details)
      -&gt; (指定されたコンストラクタメソッド)
</pre></div>

### NewObjectA() の処理
<div class="flow-abst"><pre>
jni_NewObjectA()
-&gt; alloc_object()
   -&gt; (同上)
-&gt; JNIHandles::make_local()
   -&gt; (See: <a href="noNzTqB3WT.html">here</a> for details)
-&gt; jni_invoke_nonstatic()
   -&gt; (See: <a href="no3059-0k.html">here</a> for details)
      -&gt; (指定されたコンストラクタメソッド)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_AllocObject()
See: [here](no3059PfX.html) for details
### jni_NewObjectA()
See: [here](no3059cpd.html) for details
### jni_NewObjectV()
See: [here](no3059pzj.html) for details
### jni_NewObject()
See: [here](no305929p.html) for details
### alloc_object()
See: [here](no3059DIw.html) for details
### instanceKlass::check_valid_for_instantiation()
See: [here](no3059QS2.html) for details






