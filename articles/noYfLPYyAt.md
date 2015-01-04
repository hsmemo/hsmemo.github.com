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
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> Klass::initialize() (※)
      -> (See: [here](no9AAGw84F.html) for details)
   -> initialize_class()
      -> Klass::initialize()
         -> (See: [here](no9AAGw84F.html) for details)

(※) 初期化されているのは java.util.HashMap 及び java.lang.StringValue
```

### 実行時コンスタントプールのシンボルの解決(resolution)時
#### static フィールドへのアクセス時
```
(See: [here](no7882NqI.html) for details)
-> LinkResolver::resolve_field()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

#### static メソッドへのアクセス時
```
(See: [here](no7882NqI.html) for details)
-> LinkResolver::resolve_static_call()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

#### インスタンスの生成時
```
(See: )
-> InterpreterRuntime::_new()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)

(See: ) 
-> Runtime1::new_instance()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
  
(See: ) 
-> OptoRuntime::new_instance_C()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

### 上記以外のフィールド/メソッドへのアクセスやインスタンス生成を行う処理
#### Reflection API による処理時
(#Under Construction)

```
(#TODO)
-> Reflection::invoke()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
  
(#TODO)
-> Reflection::invoke_constructor()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
  
(#TODO)
-> Reflection::resolve_field()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

#### JNI 関数による処理時
* フィールドへのアクセス時 (See: [here](noxvuCfaXS.html) for details)

```
jni_GetFieldID()
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)

jni_GetStaticFieldID()
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)
```

* メソッドへのアクセス時 (See: [here](no3059-0k.html) for details)

```
jni_GetMethodID()
-> get_method_id()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)

jni_GetStaticMethodID()
-> get_method_id()
   -> (同上)
```

* オブジェクトの生成時 (See: [here](no2935TLm.html) for details)

```
jni_AllocObject()
-> alloc_object()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)

jni_NewObject()
-> alloc_object()
   -> (同上)

jni_NewObjectV()
-> alloc_object()
   -> (同上)

jni_NewObjectA()
-> alloc_object()
   -> (同上)
```

* オブジェクト配列の生成時 (See: [here](noj4FhtQM1.html) for details)

```
jni_NewObjectArray()
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)
```

* Reflection処理 (See: [here](nodj-PTmlM.html) for details)

```
jni_FromReflectedMethod()
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)

jni_FromReflectedField()
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)
```

* DirectByteBuffer オブジェクトの生成時/へのアクセス時 (See: [here](no8QuCX1c9.html) for details)

```
jni_NewDirectByteBuffer()
-> initializeDirectBufferSupport()
   -> lookupDirectBufferClasses()
      -> lookupOne()
         -> find_class_from_class_loader()
            -> Klass::initialize()
               -> (See: [here](no9AAGw84F.html) for details)

jni_GetDirectBufferAddress()
-> initializeDirectBufferSupport()
   -> (同上)

jni_GetDirectBufferCapacity()
-> initializeDirectBufferSupport()
   -> (同上)
```

#### 上記以外のインスタンス生成時
* 例外の発生時 (= 例外オブジェクトの生成時) (See: [here](no3059qOR.html) for details)

```
(See: [here](no3059qOR.html) for details)
-> Exceptions::new_exception()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

* JMM 処理による運用管理オブジェクトの取得時 (= JMM 関係のオブジェクトの生成時) (See: [here](no2114S_x.html) for details)

```
(#TODO)
-> Management::load_and_initialize_klass()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

* java.lang.ClassLoader のアサーションステータス関係の処理時
  (= 内部で使用する java.lang.AssertionStatusDirectives オブジェクトの生成時)

```
java.lang.ClassLoader.setDefaultAssertionStatus()
-> java.lang.ClassLoader.initializeJavaAssertionMaps()
   -> JVM_AssertionStatusDirectives() (= java.lang.ClassLoader.retrieveDirectives())
      -> JavaAssertions::createAssertionStatusDirectives()
         -> Klass::initialize()
            -> (See: [here](no9AAGw84F.html) for details)

java.lang.ClassLoader.setPackageAssertionStatus()
-> java.lang.ClassLoader.initializeJavaAssertionMaps()
   -> (同上)

java.lang.ClassLoader.setClassAssertionStatus()
-> java.lang.ClassLoader.initializeJavaAssertionMaps()
   -> (同上)
```

* Dynamic Attach 機能における "properties" コマンド及び "agentProperties" 機能の処理時
  (= 内部で使用する sun.misc.VMSupport オブジェクトの生成処理時)

```
(See: [here](no3026gMG.html) for details)
-> get_system_properties()
   -> get_properties()
      -> load_and_initialize_klass()
         -> Klass::initialize()
            -> (See: [here](no9AAGw84F.html) for details)
  
(See: [here](no3026gMG.html) for details)
-> get_agent_properties()
   -> get_properties()
      -> (同上)
```

* その他の Java の標準ライブラリオブジェクトの生成時 (#TODO)

```
  java_lang_StackTraceElement::create()
  -> Klass::initialize()
     -> (See: [here](no9AAGw84F.html) for details)

  java_lang_reflect_Constructor::create()
  -> Klass::initialize()
     -> (See: [here](no9AAGw84F.html) for details)

  java_lang_reflect_Field::create()
  -> Klass::initialize()
     -> (See: [here](no9AAGw84F.html) for details)

  sun_reflect_ConstantPool::create()
  -> Klass::initialize()
     -> (See: [here](no9AAGw84F.html) for details)

  java_lang_boxing_object::initialize_and_allocate()
  -> Klass::initialize()
     -> (See: [here](no9AAGw84F.html) for details)

  java_security_AccessControlContext::create()
  -> Klass::initialize()
     -> (See: [here](no9AAGw84F.html) for details)
```

### クラスの初期化を伴う API が明示的に呼ばれたとき
#### java.lang.Class.forName(String className)
```
java.lang.Class.forName(String className) (※)
-> Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -> JVM_FindClassFromClassLoader()
      -> find_class_from_class_loader()
         -> Klass::initialize()
            -> (See: [here](no9AAGw84F.html) for details)

(※) なお, この関数はクラスローダーとして「現在のクラスを定義するクラスローダ」を使用.
```

#### java.lang.Class.forName(String name, boolean initialize, ClassLoader loader)
```
java.lang.Class.forName(String name, boolean initialize, ClassLoader loader) (※)
-> Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -> (同上)

(※) なお, この関数はクラスローダーとして「loader 引数で指定されたクラスローダ」を使用.
```

#### JNI の FindClass() (See: [here](no1lbl8Grr.html) for details)
```
jni_FindClass()
-> find_class_from_class_loader()
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

### その他
#### HotSpot の内部処理で初期化が必要になった際
* sun.misc.Unsafe.ensureClassInitialized() の処理

```
(#TODO)
-> Unsafe_EnsureClassInitialized() (= sun.misc.Unsafe.ensureClassInitialized())
   -> Klass::initialize()
      -> (See: [here](no9AAGw84F.html) for details)
```

#### 不明
```
(使われていない?)
JVM_AllocateNewObject
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)

(使われていない?)
JVM_AllocateNewArray
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)

(使われていない?)
JVM_NewInstance
-> Klass::initialize()
   -> (See: [here](no9AAGw84F.html) for details)

(使われていない?)
 JVM_FindClassFromClass()
 -> find_class_from_class_loader()
    -> Klass::initialize()
       -> (See: [here](no9AAGw84F.html) for details)

(使われていない?)
JVM_LoadClass0()
-> find_class_from_class_loader()
    -> Klass::initialize()
       -> (See: [here](no9AAGw84F.html) for details)
```

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






