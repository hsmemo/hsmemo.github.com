---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： クラスに関する処理(Class Operations)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： クラスに関する処理(Class Operations)

--- 
## 該当する JNI 関数
* `DefineClass`,
* `FindClass`,
* `GetSuperclass`,
* `IsAssignableFrom`,

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### DefineClass() の処理
<div class="flow-abst"><pre>
jni_DefineClass()
-&gt; SystemDictionary::resolve_from_stream()
   -&gt; (See: <a href="noIvSV0NZj.html">here</a> for details)
</pre></div>

### FindClass() の処理
<div class="flow-abst"><pre>
jni_FindClass() (※)
-&gt; find_class_from_class_loader()
   -&gt; SystemDictionary::resolve_or_fail()
      -&gt; (See: <a href="noIvSV0NZj.html">here</a> for details)
-&gt; CompilationPolicy::completed_vm_startup()  (&lt;= 初回の呼び出し時のみ)

(※) なお, この関数はクラスローダーとして「呼び出し元のクラスをロードしたクラスローダ (呼び出し元のクラスが存在しなければシステムクラスローダー)」を使用.
</pre></div>

### GetSuperclass() の処理
<div class="flow-abst"><pre>
jni_GetSuperclass()
-&gt; Klass::java_super()
   (instanceKlass::java_super() と arrayKlass::java_super() でオーバーライドされている)
</pre></div>

### IsAssignableFrom() の処理
<div class="flow-abst"><pre>
jni_IsAssignableFrom()
-&gt; Klass::is_subtype_of()
   -&gt; (See: ...#TODO)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_DefineClass()
See: [here](no3059dxY.html) for details

### jni_FindClass()
(#Under Construction)
See: [here](no3059QnS.html) for details
### find_class_from_class_loader()
(#Under Construction)
See: [here](no3059EQr.html) for details
### CompilationPolicy::completed_vm_startup()
See: [here](no3059Rax.html) for details

### jni_GetSuperclass()
See: [here](no3059DdM.html) for details
### instanceKlass::java_super()
See: [here](no30593Fl.html) for details
### arrayKlass::java_super()
See: [here](no3059q7e.html) for details

### jni_IsAssignableFrom()
See: [here](no30592SG.html) for details






