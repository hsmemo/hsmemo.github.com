---
layout: default
title: Universe クラス関連のクラス (CommonMethodOopCache, ActiveMethodOopsCache, LatestMethodOopCache, Universe, DeferredObjAllocEvent, 及びそれらの補助クラス(FixupMirrorClosure))
---
[Top](../index.html)

#### Universe クラス関連のクラス (CommonMethodOopCache, ActiveMethodOopsCache, LatestMethodOopCache, Universe, DeferredObjAllocEvent, 及びそれらの補助クラス(FixupMirrorClosure))

これらは, メモリ管理用, および HotSpot 内の処理で参照される Java のクラスオブジェクト／メソッド／オブジェクトの管理用のクラス.


### クラス一覧(class list)

  * [Universe](#no6ZOuud8R)
  * [CommonMethodOopCache](#noEYaCLt0I)
  * [ActiveMethodOopsCache](#noOhAV4oP-)
  * [LatestMethodOopCache](#noNae65SV_)
  * [DeferredObjAllocEvent](#noEBipWQxp)
  * [FixupMirrorClosure](#nowB7y8HIw)


---
## <a name="no6ZOuud8R" id="no6ZOuud8R">Universe</a>

### 概要(Summary)
HotSpot 内の処理で使用される Java オブジェクトや Java のシステムクラスへの参照を納めた名前空間(AllStatic クラス).

また, それらが配置される Java ヒープ領域の管理も一部担当している
(具体的には Java ヒープ領域を確保する処理(CollectedHeapオブジェクトを生成する処理)と, 
生成した CollectedHeapオブジェクトを格納する役割を果たしている.
このため CollectedHeapオブジェクトは Universe 経由でアクセスされる)

(なお, コメントには "Scavenge::invoke_and_allocate()" という謎のメソッド名が書かれている. 
そんなメソッドは現在のソースにはないが...)


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    // Universe is a name space holding known system classes and objects in the VM.
    //
    // Loaded classes are accessible through the SystemDictionary.
    //
    // The object heap is allocated and accessed through Universe, and various allocation
    // support is provided. Allocation by the interpreter and compiled code is done inline
    // and bails out to Scavenge::invoke_and_allocate.
```


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    class Universe: AllStatic {
```

### 内部構造(Internal structure)
内部には, 以下のようなフィールド/メソッドを備えている.

* HotSpot 内で使用される Java のシステムクラスへのポインタ


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      // Known classes in the VM
      static klassOop _boolArrayKlassObj;
      static klassOop _byteArrayKlassObj;
      static klassOop _charArrayKlassObj;
      static klassOop _intArrayKlassObj;
      static klassOop _shortArrayKlassObj;
      static klassOop _longArrayKlassObj;
      static klassOop _singleArrayKlassObj;
      static klassOop _doubleArrayKlassObj;
      static klassOop _typeArrayKlassObjs[T_VOID+1];
    
      static klassOop _objectArrayKlassObj;
    
      static klassOop _methodKlassObj;
      static klassOop _constMethodKlassObj;
      static klassOop _methodDataKlassObj;
      static klassOop _klassKlassObj;
      static klassOop _arrayKlassKlassObj;
      static klassOop _objArrayKlassKlassObj;
      static klassOop _typeArrayKlassKlassObj;
      static klassOop _instanceKlassKlassObj;
      static klassOop _constantPoolKlassObj;
      static klassOop _constantPoolCacheKlassObj;
      static klassOop _compiledICHolderKlassObj;
      static klassOop _systemObjArrayKlassObj;
```

* HotSpot 内で使用される Java オブジェクトへのポインタ


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      // Known objects in the VM
    
      // Primitive objects
      static oop _int_mirror;
      static oop _float_mirror;
      static oop _double_mirror;
      static oop _byte_mirror;
      static oop _bool_mirror;
      static oop _char_mirror;
      static oop _long_mirror;
      static oop _short_mirror;
      static oop _void_mirror;
    
      static oop          _main_thread_group;             // Reference to the main thread group object
      static oop          _system_thread_group;           // Reference to the system thread group object
    
      static typeArrayOop _the_empty_byte_array;          // Canonicalized byte array
      static typeArrayOop _the_empty_short_array;         // Canonicalized short array
      static typeArrayOop _the_empty_int_array;           // Canonicalized int array
      static objArrayOop  _the_empty_system_obj_array;    // Canonicalized system obj array
      static objArrayOop  _the_empty_class_klass_array;   // Canonicalized obj array of type java.lang.Class
      static objArrayOop  _the_array_interfaces_array;    // Canonicalized 2-array of cloneable & serializable klasses
      static oop          _the_null_string;               // A cache of "null" as a Java string
      static oop          _the_min_jint_string;          // A cache of "-2147483648" as a Java string
      static LatestMethodOopCache* _finalizer_register_cache; // static method for registering finalizable objects
      static LatestMethodOopCache* _loader_addClass_cache;    // method for registering loaded classes in class loader vector
      static ActiveMethodOopsCache* _reflect_invoke_cache;    // method for security checks
      static oop          _out_of_memory_error_java_heap; // preallocated error object (no backtrace)
      static oop          _out_of_memory_error_perm_gen;  // preallocated error object (no backtrace)
      static oop          _out_of_memory_error_array_size;// preallocated error object (no backtrace)
      static oop          _out_of_memory_error_gc_overhead_limit; // preallocated error object (no backtrace)
    
      // array of preallocated error objects with backtrace
      static objArrayOop   _preallocated_out_of_memory_error_array;
    
      // number of preallocated error objects available for use
      static volatile jint _preallocated_out_of_memory_error_avail_count;
    
      static oop          _null_ptr_exception_instance;   // preallocated exception object
      static oop          _arithmetic_exception_instance; // preallocated exception object
      static oop          _virtual_machine_error_instance; // preallocated exception object
      // The object used as an exception dummy when exceptions are thrown for
      // the vm thread.
      static oop          _vm_exception;
```

* CollectedHeap オブジェクト (アクセサメソッドは Universe::heap())


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      // The particular choice of collected heap.
      static CollectedHeap* heap() { return _collectedHeap; }
```




### 詳細(Details)
See: [here](../doxygen/classUniverse.html) for details

---
## <a name="noEYaCLt0I" id="noEYaCLt0I">CommonMethodOopCache</a>

### 概要(Summary)
Universe クラス内で使用される補助クラス (の基底クラス).

HotSpot 内の処理で使用される Java メソッド (methodOop) をキャッシュすることができるクラス. キャッシュの具体的な利用用途はサブクラス毎に異なる.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    // Common parts of a methodOop cache. This cache safely interacts with
    // the RedefineClasses API.
    //
    class CommonMethodOopCache : public CHeapObj {
```

### 内部構造(Internal structure)
このクラス自体の内部には以下の2つのフィールド(のみ)を保持する


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      // We save the klassOop and the idnum of methodOop in order to get
      // the current cached methodOop.
     private:
      klassOop              _klass;
      int                   _method_idnum;
```




### 詳細(Details)
See: [here](../doxygen/classCommonMethodOopCache.html) for details

---
## <a name="noOhAV4oP-" id="noOhAV4oP-">ActiveMethodOopsCache</a>

### 概要(Summary)
CommonMethodOopCache クラスの具象サブクラスの1つ.

指定の methodOop がキャッシュ対象と等しいかどうかを返す
ActiveMethodOopsCache::is_same_method() メソッドを提供する.

現状では java.lang.reflect.Method.invoke() を表す methodOop をキャッシュしておくためだけに使用されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    // A helper class for caching a methodOop when the user of the cache
    // cares about all versions of the methodOop.
    //
    class ActiveMethodOopsCache : public CommonMethodOopCache {
```


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      // This subclass adds weak references to older versions of the
      // methodOop and a query method for a methodOop.
```

なお, このクラスは, JVMTI の RedefineClasses() 関数で対象のメソッドが上書きされた場合でも,
上書きされた過去のバージョン全てを記録している.
そして, ActiveMethodOopsCache::is_same_method() はそれらのうちどれかに一致すれば true を返す.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      void add_previous_version(const methodOop method);
      bool is_same_method(const methodOop method) const;
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _reflect_invoke_cache フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    class Universe: AllStatic {
    ...
      static ActiveMethodOopsCache* _reflect_invoke_cache;    // method for security checks
```

#### 生成箇所(where its instances are created)
universe_init() 内で(のみ)生成されている.

#### 参考(for your information): universe_init()
See: [here](no344yw0.html) for details
#### 初期化箇所(where its instances are initialized)
初期化は universe_post_init() 内で行われている.

#### 参考(for your information): universe_post_init()
See: [here](no3269WqK.html) for details
#### 使用箇所(where its instances are used)
初期化時と VM_RedefineClasses::redefine_single_class() 内で情報が蓄えられ,
その情報が vframeStreamCommon::security_get_caller_frame() 内で使用されている.

* vframeStreamCommon::security_get_caller_frame()

  (この関数はスタックフレームを指定された数だけ遡る.
  その際に java.lang.reflect.Method.invoke() 等のメソッドによる "pseudo frames" は数に含めない)

  フレームが java.lang.reflect.Method.invoke() のものかどうかを判定するために使用されている.
  (See: vframeStreamCommon::security_get_caller_frame())

* VM_RedefineClasses::redefine_single_class()

  RedefineClasses() で java.lang.reflect.Method.invoke() が上書きされると,
  古いバージョンの methodOop が Universe::_reflect_invoke_cache フィールドにある
  ActiveMethodOopsCache オブジェクト内に記録されていく.




### 詳細(Details)
See: [here](../doxygen/classActiveMethodOopsCache.html) for details

---
## <a name="noNae65SV_" id="noNae65SV_">LatestMethodOopCache</a>

### 概要(Summary)
CommonMethodOopCache クラスの具象サブクラスの1つ.

対応する methodOop を取得するための LatestMethodOopCache::get_methodOop() メソッドを提供している
(なお, JVMTI の RedefineClasses() 関数で対象のメソッドが上書きされた場合には最新のメソッドだけを返す).

現状では以下のメソッドを表す methodOop をキャッシュしておくためだけに使用されている.

* java.lang.ref.Finalizer.register()
* java.lang.ClassLoader.addClass()


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    // A helper class for caching a methodOop when the user of the cache
    // only cares about the latest version of the methodOop.
    //
    class LatestMethodOopCache : public CommonMethodOopCache {
```


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      // This subclass adds a getter method for the latest methodOop.
```


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      methodOop get_methodOop();
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Universe クラスの _finalizer_register_cache フィールドおよび
_loader_addClass_cache フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    class Universe: AllStatic {
    ...
      static LatestMethodOopCache* _finalizer_register_cache; // static method for registering finalizable objects
      static LatestMethodOopCache* _loader_addClass_cache;    // method for registering loaded classes in class loader vector
```

#### 生成箇所(where its instances are created)
universe_init() 内で(のみ)生成されている.

#### 参考(for your information): universe_init()
See: [here](no344yw0.html) for details
#### 初期化箇所(where its instances are initialized)
初期化は universe_post_init() 内で行われている.

#### 参考(for your information): universe_post_init()
See: [here](no3269WqK.html) for details
#### 使用箇所(where its instances are used)
これらは, 対応する methodOop を返す以下の関数内で(のみ)使用されている.

* Universe::finalizer_register_method()

  なお, この関数自体は
  instanceKlass::register_finalizer() 内で
  java.lang.ref.Finalizer.register() を呼び出すために(のみ)使用されている.

* Universe::loader_addClass_method()

  なお, この関数自体は
  SystemDictionary::define_instance_class() 内で
  java.lang.ClassLoader.addClass() を呼び出すために(のみ)使用されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
      static methodOop    finalizer_register_method()     { return _finalizer_register_cache->get_methodOop(); }
      static methodOop    loader_addClass_method()        { return _loader_addClass_cache->get_methodOop(); }
```

#### 参考(for your information): instanceKlass::register_finalizer()
See: [here](no344nLB.html) for details
#### 参考(for your information): SystemDictionary::define_instance_class()
See: [here](no3269wFL.html) for details



### 詳細(Details)
See: [here](../doxygen/classLatestMethodOopCache.html) for details

---
## <a name="noEBipWQxp" id="noEBipWQxp">DeferredObjAllocEvent</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.hpp))
    class DeferredObjAllocEvent : public CHeapObj {
```

### 使われ方(Usage)
(現状では使われていない)




### 詳細(Details)
See: [here](../doxygen/classDeferredObjAllocEvent.html) for details

---
## <a name="nowB7y8HIw" id="nowB7y8HIw">FixupMirrorClosure</a>

### 概要(Summary)
Universe クラス内で使用される補助クラス
(より正確には, Universe クラスから呼び出される SystemDictionary の初期化処理で使用されるクラス).

java.lang.Class よりも先にロードされたクラスに対して mirror を生成する処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.cpp))
    class FixupMirrorClosure: public ObjectClosure {
```

#### 使用箇所(where its instances are used)
Universe::fixup_mirrors() 内で(のみ)使用されている.
この関数は, 現在は以下のパスで(のみ)呼び出されている.

```
Universe::genesis()
-> SystemDictionary::initialize()
   -> SystemDictionary::initialize_preloaded_classes()
      -> Universe::fixup_mirrors()
```

### 内部構造(Internal structure)
FixupMirrorClosure::do_object() 中では, java_lang_Class::fixup_mirror()
(の中で呼び出される java_lang_Class::create_mirror()) によって mirror オブジェクトを生成している.

#### 参考(for your information): FixupMirrorClosure::do_object()
See: [here](no3269Xkd.html) for details
#### 参考(for your information): java_lang_Class::fixup_mirror()
See: [here](no3269kuj.html) for details
#### 参考(for your information): java_lang_Class::create_mirror()
See: [here](no3269x4p.html) for details



### 詳細(Details)
See: [here](../doxygen/classFixupMirrorClosure.html) for details

---
