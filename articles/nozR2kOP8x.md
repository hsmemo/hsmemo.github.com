---
layout: default
title: instanceRefKlass クラス 
---
[Top](../index.html)

#### instanceRefKlass クラス 



---
## <a name="noWrwgKeso" id="noWrwgKeso">instanceRefKlass</a>

### 概要(Summary)
java.lang.ref.Reference (のサブクラス)のクラスオブジェクト用の Klass クラス (soft/weak/final/phantom reference).

これらは GC 時の扱いが通常の instanceKlass と異なるので特別扱いしている, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceRefKlass.hpp))
    // An instanceRefKlass is a specialized instanceKlass for Java
    // classes that are subclasses of java/lang/ref/Reference.
    //
    // These classes are used to implement soft/weak/final/phantom
    // references and finalization, and need special treatment by the
    // garbage collector.
    //
    // During GC discovered reference objects are added (chained) to one
    // of the four lists below, depending on the type of reference.
    // The linked occurs through the next field in class java/lang/ref/Reference.
    //
    // Afterwards, the discovered references are processed in decreasing
    // order of reachability. Reference objects eligible for notification
    // are linked to the static pending_list in class java/lang/ref/Reference,
    // and the pending list lock object in the same class is notified.
    
    
    class instanceRefKlass: public instanceKlass {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SystemDictionary クラスの _well_known_klasses フィールド (static フィールド) に(のみ)格納されている.

(アクセサメソッドは
SystemDictionary::SoftReference_klass(),
SystemDictionary::WeakReference_klass(),
SystemDictionary::FinalReference_klass(),
SystemDictionary::PhantomReference_klass()).

(正確には, このフィールドは klassOop の配列を格納するフィールド.
配列長は SystemDictionary::WKID_LIMIT)

(なお, これらは各 java.lang.ref.Reference オブジェクトの klass フィールドからも参照されている)

#### 生成箇所(where its instances are created)
instanceRefKlass オブジェクトの生成処理, 及び
SystemDictionary::_well_known_klasses フィールドへの登録処理は,
SystemDictionary::initialize_preloaded_classes() 内で行われている.

(より正確に言うと, 実際の生成処理自体はそこから呼び出される instanceKlassKlass::allocate_instance_klass() 内で行われる.
なお, この関数では instanceRefKlass オブジェクトではなく
instanceKlass オブジェクトや instanceMirrorKlass オブジェクトが生成されることもある.
instanceRefKlass オブジェクトは, スーパークラスの instanceKlass::reference_type() の値が REF_NONE 以外である場合にのみ生成される.
上の ４種類の Reference サブクラスについては,
java.lang.ref.Reference の instanceKlass::reference_type() が REF_OTHER に設定済みの状態で生成されるので instanceRefKlass になる)

(See: instanceKlassKlass::allocate_instance_klass(), ClassFileParser::parseClassFile(), SystemDictionary::initialize_preloaded_classes())]

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> init_globals()
      -> universe2_init()
         -> Universe::genesis()
            -> SystemDictionary::initialize()
               -> SystemDictionary::initialize_preloaded_classes()
                  -> SystemDictionary::initialize_wk_klasses_through()
                     -> SystemDictionary::initialize_wk_klasses_until()
                        -> SystemDictionary::initialize_wk_klass()
                           -> SystemDictionary::resolve_or_fail()
                              -> (See: ... #TODO)
```




### 詳細(Details)
See: [here](../doxygen/classinstanceRefKlass.html) for details

---
