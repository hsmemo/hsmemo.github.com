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
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> init_globals()
      -> universe_post_init()
         -> instanceKlass::link_class()
            -> (See: [here](no3059xqe.html) for details)
         -> instanceKlass::link_class_or_fail()
            -> instanceKlass::link_class_impl()
               -> (See: [here](no3059xqe.html) for details)
```

### クラスの初期化処理
```
(See: [here](no9AAGw84F.html) for details)
-> instanceKlass::initialize()
   -> instanceKlass::initialize_impl()
      -> instanceKlass::link_class()
         -> (See: [here](no3059xqe.html) for details)
```

### Reflection 処理
```
(See: [here](no1904Wlh.html) for details)
-> JVM_GetClassDeclaredFields()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> JVM_GetClassDeclaredMethods()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> JVM_GetClassDeclaredConstructors()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> Reflection::reflect_field()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> Reflection::reflect_fields()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> Reflection::reflect_method()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> Reflection::reflect_methods()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> Reflection::reflect_constructor()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
  
(See: [here](no1904Wlh.html) for details)
-> Reflection::reflect_constructors()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
```

### JVMTI の RedefineClasses() 処理
```
(See: [here](no2935-Vj.html) for details)
-> VM_RedefineClasses::load_new_class_versions()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
```

### MethodHandles 関係の処理(#TODO)
```
MethodHandles::resolve_MemberName()
-> instanceKlass::link_class()
    -> (See: [here](no3059xqe.html) for details)

MethodHandles::init_DirectMethodHandle()
-> instanceKlass::link_class()
    -> (See: [here](no3059xqe.html) for details)
```

### Class Data Sharing (CDS) のダンプ出力処理
```
(See: [here](no2114Sn1.html) for details)
-> GenCollectedHeap::preload_and_dump()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)

(See: [here](no2114Sn1.html) for details)
-> LinkClassesClosure::do_object()
   -> instanceKlass::link_class()
      -> (See: [here](no3059xqe.html) for details)
```

### デバッグ用の処理 (EagerInitialization オプションの処理)
```
(See: [here](noIvSV0NZj.html) for details)
-> instanceKlass::eager_initialize()
   -> instanceKlass::eager_initialize_impl() @ hotspot/src/share/vm/oops/instanceKlass.cpp
      -> instanceKlass::link_class_impl()
         -> (See: [here](no3059xqe.html) for details)
```







