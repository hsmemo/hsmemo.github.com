---
layout: default
title: Class のロード/リンク/初期化 ： リンク処理 (1) ： リンク処理の開始点
---
[Up](noX5hsnWQw.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： リンク処理 (1) ： リンク処理の開始点

--- 
## 概要(Summary)
クラスのリンク処理は以下の契機で開始される.

* HotSpot 起動時の標準ライブラリクラスの初期化処理 (See: [here](no2114J7x.html) for details)

* クラスの初期化処理 (See: [here](no9AAGw84F.html) for details)
  
  (初期化前にはリンクされている必要があるため, 念のため常にリンク処理が呼び出される)
  
* その他
  
  * Reflection 処理 (See: [here](no1904Wlh.html) for details)
    
  * JVMTI の RedefineClasses() 処理 (See: [here](no2935-Vj.html) for details)
    
  * MethodHandles 関係の処理(#TODO)
    
  * Class Data Sharing (CDS) のダンプ出力処理 (See: [here](no2114Sn1.html) for details)
    
  * デバッグ用の処理 (EagerInitialization オプションの処理) (See: [here](noIvSV0NZj.html) for details)
    
    (develop オプションである EagerInitialization が true になっている場合にのみ実行 (デフォルト false))


## 処理の流れ (概要)(Execution Flows : Summary)
### 起動中の標準ライブラリクラスの初期化処理
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; universe_post_init()
         -&gt; instanceKlass::link_class()
            -&gt; (See: <a href="no3059xqe.html">here</a> for details)
         -&gt; instanceKlass::link_class_or_fail()
            -&gt; instanceKlass::link_class_impl()
               -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>

### クラスの初期化処理
<div class="flow-abst"><pre>
(See: <a href="no9AAGw84F.html">here</a> for details)
-&gt; instanceKlass::initialize()
   -&gt; instanceKlass::initialize_impl()
      -&gt; instanceKlass::link_class()
         -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>

### Reflection 処理
<div class="flow-abst"><pre>
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; JVM_GetClassDeclaredFields()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; JVM_GetClassDeclaredMethods()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; JVM_GetClassDeclaredConstructors()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; Reflection::reflect_field()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; Reflection::reflect_fields()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; Reflection::reflect_method()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; Reflection::reflect_methods()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; Reflection::reflect_constructor()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
  
(See: <a href="no1904Wlh.html">here</a> for details)
-&gt; Reflection::reflect_constructors()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>

### JVMTI の RedefineClasses() 処理
<div class="flow-abst"><pre>
(See: <a href="no2935-Vj.html">here</a> for details)
-&gt; VM_RedefineClasses::load_new_class_versions()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>

### MethodHandles 関係の処理(#TODO)
<div class="flow-abst"><pre>
MethodHandles::resolve_MemberName()
-&gt; instanceKlass::link_class()
    -&gt; (See: <a href="no3059xqe.html">here</a> for details)

MethodHandles::init_DirectMethodHandle()
-&gt; instanceKlass::link_class()
    -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>

### Class Data Sharing (CDS) のダンプ出力処理
<div class="flow-abst"><pre>
(See: <a href="no2114Sn1.html">here</a> for details)
-&gt; GenCollectedHeap::preload_and_dump()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)

(See: <a href="no2114Sn1.html">here</a> for details)
-&gt; LinkClassesClosure::do_object()
   -&gt; instanceKlass::link_class()
      -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>

### デバッグ用の処理 (EagerInitialization オプションの処理)
<div class="flow-abst"><pre>
(See: <a href="noIvSV0NZj.html">here</a> for details)
-&gt; instanceKlass::eager_initialize()
   -&gt; instanceKlass::eager_initialize_impl() @ hotspot/src/share/vm/oops/instanceKlass.cpp
      -&gt; instanceKlass::link_class_impl()
         -&gt; (See: <a href="no3059xqe.html">here</a> for details)
</pre></div>







