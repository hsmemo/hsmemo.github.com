---
layout: default
title: 配列クラスの生成処理  
---
[Up](no38NSe1ks.html) [Top](../index.html)

#### 配列クラスの生成処理  

--- 
## 概要(Summary)
配列クラスは以下の契機で作成される.

  * プリミティブ型の1次元配列のクラスは, HotSpot の起動時に生成される.

  * その他の配列クラスは, そのクラスが初めて使用される際に生成される (#TODO).

## 備考(Notes)
* 作成された配列クラスは, それぞれ以下のフィールドから参照されている
  
    * プリミティブ型の1次元配列のクラス: (typeArrayKlass オブジェクト)
      
      Universe クラス内のフィールドから参照されている
      (See: Universe::typeArrayKlassObj())
      
    * オブジェクト型の1次元配列のクラス: (objArrayKlass オブジェクト)
      
      対応する instanceKlass オブジェクトの _array_klasses フィールドから参照されている.
      
    * 2次元以上の配列クラス: (objArrayKlass オブジェクト)
      
      次元が1つ下の配列クラスの arrayKlass::_higher_dimension フィールドから参照されている.
    
* 配列クラスの取得処理 (Klass::array_klass()) では,
  上記 arrayKlass::_higher_dimension フィールドを用いて再帰的に探索が行われる.
  (まず 1次元配列のクラスから開始し, 
  arrayKlass::_higher_dimension を辿って指定の次元数に達するまで配列クラスを辿る.
  途中で未作成の配列クラスがあれば作成する処理も行われる).


## 処理の流れ (概要)(Execution Flows : Summary)
### プリミティブ型の1次元配列クラスの作成処理
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> init_globals()
      -> universe2_init()
         -> Universe::genesis()
            -> typeArrayKlass::create_klass(BasicType type, int scale, TRAPS)
               -> typeArrayKlass::create_klass(BasicType type, int scale, const char* name_str, TRAPS)
                  -> (1) 新しい typeArrayKlass オブジェクトを生成する
                         -> arrayKlass::base_create_array_klass()
                            -> Klass::base_create_klass()
                               -> Klass::base_create_klass_oop()
                                  -> Klass_vtbl::allocate_permanent(KlassHandle& klass, int size, TRAPS) (を各サブクラスがオーバーライドしたもの)
                                     -> (1) 新しい Klass クラス (のサブクラス) オブジェクトを確保
                                        (1) 確保した klassOop の klass フィールドを初期化
                                            -> Klass_vtbl::post_new_init_klass()
                                               -> CollectedHeap::post_allocation_install_obj_klass()

                     (1) 生成した配列クラスの vtable や mirror オブジェクト (Java レベルのクラスオブジェクト) を生成する.
                         -> arrayKlass::complete_create_array_klass()
                            -> Klass::initialize_supers()
                            -> klassVtable::initialize_vtable()
                            -> java_lang_Class::create_mirror()
```

### その他の配列クラスの作成処理
* 配列クラスのロード処理 (See: [here](noIvSV0NZj.html) for details)

```
(See: [here](noIvSV0NZj.html) for details)
-> SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
   -> * ロード対象が配列クラスの場合:
        -> SystemDictionary::resolve_array_class_or_null()
           -> * オブジェクト型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
                -> (1) 要素であるオブジェクト型のクラス(instanceKlass)を取得する
                       -> SystemDictionary::resolve_instance_class_or_null()
                          -> (See: [here](noIvSV0NZj.html) for details)
                   (1) 取得したinstanceKlassを基に配列クラスを取得する
                       -> Klass::array_klass()
                          -> instanceKlass::array_klass_impl()
                             -> (1) まだ作成してなければ, 一次元配列クラスを作成する
                                    -> objArrayKlassKlass::allocate_objArray_klass()
                                       -> objArrayKlassKlass::allocate_objArray_klass_impl()
                                          -> arrayKlass::base_create_array_klass()
                                             -> (同上)
                                (1) 対応する次元の配列クラスを取得する
                                    -> Klass::array_klass()  (再帰呼び出し)
                                       -> objArrayKlass::array_klass_impl()
                                          -> (1) 引数の次元数が一致すれば, それをリターン
                                             (1) まだ作成してなければ, 1つ高次元の配列クラスを作成する
                                                 -> objArrayKlassKlass::allocate_objArray_klass()
                                                    -> (同上)
                                             (1) 1つ高次元の配列クラスに対して再帰呼び出し
                                                 -> Klass::array_klass()  (再帰呼び出し)
                                                    -> objArrayKlass::array_klass_impl()
                                                       -> (以下, 繰り返し)

              * プリミティブ型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
                -> (1) プリミティブ型の1次元配列クラスを取得する
                       -> Universe::typeArrayKlassObj()
                   (1) 取得した配列クラスから配列クラスを取得する
                       -> Klass::array_klass()
                          -> typeArrayKlass::array_klass_impl()
                             -> (1) 引数の次元数が一致すれば, それをリターン
                                (1) まだ作成してなければ, 配列クラスを作成する.
                                    -> objArrayKlassKlass::allocate_objArray_klass()
                                       -> (同上)
                                (1) 再帰呼び出しにより, 対応する次元の配列クラスを取得する
                                    -> Klass::array_klass()  (再帰呼び出し)
                                       -> objArrayKlass::array_klass_impl()
                                          -> (同上)
```

* JNI による配列処理 (See: [here](noj4FhtQM1.html) for details)
  
```
jni_NewObjectArray()
-> Klass::array_klass()
   -> (同上)
```

* ...


## 処理の流れ (詳細)(Execution Flows : Details)
### Universe::genesis()
See: [here](no4230JvC.html) for details
### typeArrayKlass::create_klass(BasicType type, int scale, TRAPS)
See: [here](no190186yT.html) for details
### typeArrayKlass::create_klass(BasicType type, int scale, const char* name_str, TRAPS)
(#Under Construction)
See: [here](no19018tvB.html) for details
### arrayKlass::base_create_array_klass()
See: [here](no1901876O.html) for details
### Klass::base_create_klass()
See: [here](no19018udW.html) for details
### Klass::base_create_klass_oop()
(#Under Construction)
See: [here](no19018I5W.html) for details
### Klass_vtbl::allocate_permanent(KlassHandle& klass, int size, TRAPS) (を各サブクラスがオーバーライドしたもの)
See: [here](no19018vlR.html) for details
### Klass_vtbl::post_new_init_klass()
See: [here](no19018KJN.html) for details
### CollectedHeap::post_allocation_install_obj_klass()
See: [here](no344CTs.html) for details

### arrayKlass::complete_create_array_klass()
(#Under Construction)
See: [here](no19018kpX.html) for details
### Klass::initialize_supers()
(#Under Construction)

### klassVtable::initialize_vtable()
(#Under Construction)
See: [here](no18536t22.html) for details
### java_lang_Class::create_mirror()
See: [here](no3269x4p.html) for details

### SystemDictionary::resolve_array_class_or_null()
See: [here](no7517ndW.html) for details
### Klass::array_klass(int rank, TRAPS)
See: [here](no19018eJH.html) for details
### Klass::array_klass(TRAPS)
See: [here](no190185Qy.html) for details
### instanceKlass::array_klass_impl(bool or_null, int n, TRAPS)
See: [here](no19018fDa.html) for details
### instanceKlass::array_klass_impl(bool or_null, TRAPS)
See: [here](no19018Gis.html) for details
### instanceKlass::array_klass_impl(instanceKlassHandle this_oop, bool or_null, int n, TRAPS)
See: [here](no19018ZoT.html) for details
### objArrayKlassKlass::allocate_objArray_klass()
See: [here](no19018fRC.html) for details
### objArrayKlassKlass::allocate_objArray_klass_impl()
(#Under Construction)
See: [here](no19018G3I.html) for details

### typeArrayKlass::array_klass_impl(bool or_null, int n, TRAPS)
See: [here](no190181HM.html) for details
### typeArrayKlass::array_klass_impl(bool or_null, TRAPS)
See: [here](no190181OA.html) for details
### typeArrayKlass::array_klass_impl(typeArrayKlassHandle h_this, bool or_null, int n, TRAPS)
See: [here](no19018PqA.html) for details

### objArrayKlass::array_klass_impl(bool or_null, int n, TRAPS)
See: [here](no19018HzF.html) for details
### objArrayKlass::array_klass_impl(bool or_null, TRAPS)
See: [here](no19018U9L.html) for details
### objArrayKlass::array_klass_impl(objArrayKlassHandle this_oop, bool or_null, int n, TRAPS)
See: [here](no190187be.html) for details






