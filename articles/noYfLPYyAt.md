---
layout: default
title: Class のロード/リンク/初期化 ： 初期化処理 (1) ： 初期化処理の開始点
---
[Up](no6dqMzJWt.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： 初期化処理 (1) ： 初期化処理の開始点

--- 
## 概要(Summary)
クラスの初期化処理は以下の契機で開始される.

  * HotSpot の起動時 (See: [here](no2114J7x.html) for details)
    
    HotSpot の起動時に, 幾つかの標準ライブラリのクラスが初期化される 
    
  * 実行時コンスタントプールのシンボルの解決(resolution)時 (See: [here](no7882NqI.html) for details)
    
    (static フィールドへのアクセス, static メソッドへのアクセス, インスタンスの生成, 等により)
    実行時コンスタントプール中のクラス名／インターフェース名の解決が行われた際, 
    対象がまだ初期化されていなければ初期化処理が実行される
    
  * 上記以外のフィールド/メソッドへのアクセス時やインスタンス生成時
    (JNI 関数, Reflection APIs, etc)
    
    対象のクラスがまだ初期化されていなければ, 初期化処理が実行される

  * クラスの初期化を伴う API が明示的に呼ばれたとき (java.lang.Class.forName(), JNI の FindClass(), etc)
    
    対象のクラスがまだ初期化されていなければ, 初期化処理が実行される

  * その他 (HotSpot の内部処理で初期化が必要になった際, etc)


## 処理の流れ (概要)(Execution Flows : Summary)
### 起動中の標準ライブラリクラスの初期化処理
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; Klass::initialize() (※)
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
   -&gt; initialize_class()
      -&gt; Klass::initialize()
         -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(※) 初期化されているのは java.util.HashMap 及び java.lang.StringValue
</pre></div>

### 実行時コンスタントプールのシンボルの解決(resolution)時
#### static フィールドへのアクセス時
<div class="flow-abst"><pre>
(See: <a href="no7882NqI.html">here</a> for details)
-&gt; LinkResolver::resolve_field()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

#### static メソッドへのアクセス時
<div class="flow-abst"><pre>
(See: <a href="no7882NqI.html">here</a> for details)
-&gt; LinkResolver::resolve_static_call()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

#### インスタンスの生成時
<div class="flow-abst"><pre>
(See: )
-&gt; InterpreterRuntime::_new()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(See: ) 
-&gt; Runtime1::new_instance()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
  
(See: ) 
-&gt; OptoRuntime::new_instance_C()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

### 上記以外のフィールド/メソッドへのアクセスやインスタンス生成を行う処理
#### Reflection API による処理時
(#Under Construction)

<div class="flow-abst"><pre>
(#TODO)
-&gt; Reflection::invoke()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
  
(#TODO)
-&gt; Reflection::invoke_constructor()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
  
(#TODO)
-&gt; Reflection::resolve_field()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

#### JNI 関数による処理時
* フィールドへのアクセス時 (See: [here](noxvuCfaXS.html) for details)

<div class="flow-abst"><pre>
jni_GetFieldID()
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

jni_GetStaticFieldID()
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

* メソッドへのアクセス時 (See: [here](no3059-0k.html) for details)

<div class="flow-abst"><pre>
jni_GetMethodID()
-&gt; get_method_id()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

jni_GetStaticMethodID()
-&gt; get_method_id()
   -&gt; (同上)
</pre></div>

* オブジェクトの生成時 (See: [here](no2935TLm.html) for details)

<div class="flow-abst"><pre>
jni_AllocObject()
-&gt; alloc_object()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

jni_NewObject()
-&gt; alloc_object()
   -&gt; (同上)

jni_NewObjectV()
-&gt; alloc_object()
   -&gt; (同上)

jni_NewObjectA()
-&gt; alloc_object()
   -&gt; (同上)
</pre></div>

* オブジェクト配列の生成時 (See: [here](noj4FhtQM1.html) for details)

<div class="flow-abst"><pre>
jni_NewObjectArray()
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

* Reflection処理 (See: [here](nodj-PTmlM.html) for details)

<div class="flow-abst"><pre>
jni_FromReflectedMethod()
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

jni_FromReflectedField()
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

* DirectByteBuffer オブジェクトの生成時/へのアクセス時 (See: [here](no8QuCX1c9.html) for details)

<div class="flow-abst"><pre>
jni_NewDirectByteBuffer()
-&gt; initializeDirectBufferSupport()
   -&gt; lookupDirectBufferClasses()
      -&gt; lookupOne()
         -&gt; find_class_from_class_loader()
            -&gt; Klass::initialize()
               -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

jni_GetDirectBufferAddress()
-&gt; initializeDirectBufferSupport()
   -&gt; (同上)

jni_GetDirectBufferCapacity()
-&gt; initializeDirectBufferSupport()
   -&gt; (同上)
</pre></div>

#### 上記以外のインスタンス生成時
* 例外の発生時 (= 例外オブジェクトの生成時) (See: [here](no3059qOR.html) for details)

<div class="flow-abst"><pre>
(See: <a href="no3059qOR.html">here</a> for details)
-&gt; Exceptions::new_exception()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

* JMM 処理による運用管理オブジェクトの取得時 (= JMM 関係のオブジェクトの生成時) (See: [here](no2114S_x.html) for details)

<div class="flow-abst"><pre>
(#TODO)
-&gt; Management::load_and_initialize_klass()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

* java.lang.ClassLoader のアサーションステータス関係の処理時
  (= 内部で使用する java.lang.AssertionStatusDirectives オブジェクトの生成時)

<div class="flow-abst"><pre>
java.lang.ClassLoader.setDefaultAssertionStatus()
-&gt; java.lang.ClassLoader.initializeJavaAssertionMaps()
   -&gt; JVM_AssertionStatusDirectives() (= java.lang.ClassLoader.retrieveDirectives())
      -&gt; JavaAssertions::createAssertionStatusDirectives()
         -&gt; Klass::initialize()
            -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

java.lang.ClassLoader.setPackageAssertionStatus()
-&gt; java.lang.ClassLoader.initializeJavaAssertionMaps()
   -&gt; (同上)

java.lang.ClassLoader.setClassAssertionStatus()
-&gt; java.lang.ClassLoader.initializeJavaAssertionMaps()
   -&gt; (同上)
</pre></div>

* Dynamic Attach 機能における "properties" コマンド及び "agentProperties" 機能の処理時
  (= 内部で使用する sun.misc.VMSupport オブジェクトの生成処理時)

<div class="flow-abst"><pre>
(See: <a href="no3026gMG.html">here</a> for details)
-&gt; get_system_properties()
   -&gt; get_properties()
      -&gt; load_and_initialize_klass()
         -&gt; Klass::initialize()
            -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
  
(See: <a href="no3026gMG.html">here</a> for details)
-&gt; get_agent_properties()
   -&gt; get_properties()
      -&gt; (同上)
</pre></div>

* その他の Java の標準ライブラリオブジェクトの生成時 (#TODO)

<div class="flow-abst"><pre>
  java_lang_StackTraceElement::create()
  -&gt; Klass::initialize()
     -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

  java_lang_reflect_Constructor::create()
  -&gt; Klass::initialize()
     -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

  java_lang_reflect_Field::create()
  -&gt; Klass::initialize()
     -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

  sun_reflect_ConstantPool::create()
  -&gt; Klass::initialize()
     -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

  java_lang_boxing_object::initialize_and_allocate()
  -&gt; Klass::initialize()
     -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

  java_security_AccessControlContext::create()
  -&gt; Klass::initialize()
     -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

### クラスの初期化を伴う API が明示的に呼ばれたとき
#### java.lang.Class.forName(String className)
<div class="flow-abst"><pre>
java.lang.Class.forName(String className) (※)
-&gt; Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -&gt; JVM_FindClassFromClassLoader()
      -&gt; find_class_from_class_loader()
         -&gt; Klass::initialize()
            -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(※) なお, この関数はクラスローダーとして「現在のクラスを定義するクラスローダ」を使用.
</pre></div>

#### java.lang.Class.forName(String name, boolean initialize, ClassLoader loader)
<div class="flow-abst"><pre>
java.lang.Class.forName(String name, boolean initialize, ClassLoader loader) (※)
-&gt; Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -&gt; (同上)

(※) なお, この関数はクラスローダーとして「loader 引数で指定されたクラスローダ」を使用.
</pre></div>

#### JNI の FindClass() (See: [here](no1lbl8Grr.html) for details)
<div class="flow-abst"><pre>
jni_FindClass()
-&gt; find_class_from_class_loader()
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

### その他
#### HotSpot の内部処理で初期化が必要になった際
* sun.misc.Unsafe.ensureClassInitialized() の処理

<div class="flow-abst"><pre>
(#TODO)
-&gt; Unsafe_EnsureClassInitialized() (= sun.misc.Unsafe.ensureClassInitialized())
   -&gt; Klass::initialize()
      -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

#### 不明
<div class="flow-abst"><pre>
(使われていない?)
JVM_AllocateNewObject
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(使われていない?)
JVM_AllocateNewArray
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(使われていない?)
JVM_NewInstance
-&gt; Klass::initialize()
   -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(使われていない?)
 JVM_FindClassFromClass()
 -&gt; find_class_from_class_loader()
    -&gt; Klass::initialize()
       -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(使われていない?)
JVM_LoadClass0()
-&gt; find_class_from_class_loader()
    -&gt; Klass::initialize()
       -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)
</pre></div>

### 未整理(#TODO)
* MethodHandles
  MethodHandles::new_MemberName
  
  MethodHandles::encode_target

  MethodHandles::init_DirectMethodHandle
  
  MethodHandles::raise_exception


## 処理の流れ (詳細)(Execution Flows : Details)
### Threads::create_vm()
See: [here](no4230j8a.html) for details
### initialize_class()
See: [here](no26814_eP.html) for details






