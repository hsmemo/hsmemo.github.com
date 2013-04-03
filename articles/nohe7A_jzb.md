---
layout: default
title: instanceMirrorKlass クラス 
---
[Top](../index.html)

#### instanceMirrorKlass クラス 



---
## <a name="nohQ_N_Lpy" id="nohQ_N_Lpy">instanceMirrorKlass</a>

### 概要(Summary)
java.lang.Class のインスタンス(= Java のクラスオブジェクト = mirror オブジェクト)用の Klass クラス.

クラスオブジェクトはそれぞれ異なる個数の static フィールドを持つので
(= サイズの計算や oop の iterate 処理をそれぞれ変える必要があるので), 特別扱いしている模様.

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SystemDictionary クラスの _well_known_klasses フィールド (static フィールド) に(のみ)格納されている
(アクセサメソッドは SystemDictionary::Class_klass()).

(正確には, このフィールドは klassOop の配列を格納するフィールド.
配列長は SystemDictionary::WKID_LIMIT)

(なお, このオブジェクトは各 mirror オブジェクトの klass フィールドからも参照されている)

#### 生成箇所(where its instances are created)
instanceMirrorKlass オブジェクトの生成処理, 及び
SystemDictionary::_well_known_klasses フィールドへの登録処理は,
SystemDictionary::initialize_preloaded_classes() 内で行われている.

(より正確に言うと, 実際の生成処理自体はそこから呼び出される instanceKlassKlass::allocate_instance_klass() 内で行われる.
なお, この関数では instanceMirrorKlass オブジェクトではなく
instanceKlass オブジェクトや instanceRefKlass オブジェクトが生成されることもある.
instanceMirrorKlass オブジェクトは, クラス名が java.lang.Class である場合にのみ生成される.)

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

### 内部構造(Internal structure)
クラスオブジェクトは static フィールドの数が不定なため,
instanceMirrorKlass::oop_oop_iterate() 等の処理では
実行時に java_lang_Class::static_oop_field_count() で oop 数を取得している.




### 詳細(Details)
See: [here](../doxygen/classinstanceMirrorKlass.html) for details

---
