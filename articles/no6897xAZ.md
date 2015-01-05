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
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; universe2_init()
         -&gt; Universe::genesis()
            -&gt; typeArrayKlass::create_klass(BasicType type, int scale, TRAPS)
               -&gt; typeArrayKlass::create_klass(BasicType type, int scale, const char* name_str, TRAPS)
                  -&gt; (1) 新しい typeArrayKlass オブジェクトを生成する
                         -&gt; arrayKlass::base_create_array_klass()
                            -&gt; Klass::base_create_klass()
                               -&gt; Klass::base_create_klass_oop()
                                  -&gt; Klass_vtbl::allocate_permanent(KlassHandle&amp; klass, int size, TRAPS) (を各サブクラスがオーバーライドしたもの)
                                     -&gt; (1) 新しい Klass クラス (のサブクラス) オブジェクトを確保
                                        (1) 確保した klassOop の klass フィールドを初期化
                                            -&gt; Klass_vtbl::post_new_init_klass()
                                               -&gt; CollectedHeap::post_allocation_install_obj_klass()

                     (1) 生成した配列クラスの vtable や mirror オブジェクト (Java レベルのクラスオブジェクト) を生成する.
                         -&gt; arrayKlass::complete_create_array_klass()
                            -&gt; Klass::initialize_supers()
                            -&gt; klassVtable::initialize_vtable()
                            -&gt; java_lang_Class::create_mirror()
</pre></div>

### その他の配列クラスの作成処理
* 配列クラスのロード処理 (See: [here](noIvSV0NZj.html) for details)

<div class="flow-abst"><pre>
(See: <a href="noIvSV0NZj.html">here</a> for details)
-&gt; SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
   -&gt; * ロード対象が配列クラスの場合:
        -&gt; SystemDictionary::resolve_array_class_or_null()
           -&gt; * オブジェクト型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
                -&gt; (1) 要素であるオブジェクト型のクラス(instanceKlass)を取得する
                       -&gt; SystemDictionary::resolve_instance_class_or_null()
                          -&gt; (See: <a href="noIvSV0NZj.html">here</a> for details)
                   (1) 取得したinstanceKlassを基に配列クラスを取得する
                       -&gt; Klass::array_klass()
                          -&gt; instanceKlass::array_klass_impl()
                             -&gt; (1) まだ作成してなければ, 一次元配列クラスを作成する
                                    -&gt; objArrayKlassKlass::allocate_objArray_klass()
                                       -&gt; objArrayKlassKlass::allocate_objArray_klass_impl()
                                          -&gt; arrayKlass::base_create_array_klass()
                                             -&gt; (同上)
                                (1) 対応する次元の配列クラスを取得する
                                    -&gt; Klass::array_klass()  (再帰呼び出し)
                                       -&gt; objArrayKlass::array_klass_impl()
                                          -&gt; (1) 引数の次元数が一致すれば, それをリターン
                                             (1) まだ作成してなければ, 1つ高次元の配列クラスを作成する
                                                 -&gt; objArrayKlassKlass::allocate_objArray_klass()
                                                    -&gt; (同上)
                                             (1) 1つ高次元の配列クラスに対して再帰呼び出し
                                                 -&gt; Klass::array_klass()  (再帰呼び出し)
                                                    -&gt; objArrayKlass::array_klass_impl()
                                                       -&gt; (以下, 繰り返し)

              * プリミティブ型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
                -&gt; (1) プリミティブ型の1次元配列クラスを取得する
                       -&gt; Universe::typeArrayKlassObj()
                   (1) 取得した配列クラスから配列クラスを取得する
                       -&gt; Klass::array_klass()
                          -&gt; typeArrayKlass::array_klass_impl()
                             -&gt; (1) 引数の次元数が一致すれば, それをリターン
                                (1) まだ作成してなければ, 配列クラスを作成する.
                                    -&gt; objArrayKlassKlass::allocate_objArray_klass()
                                       -&gt; (同上)
                                (1) 再帰呼び出しにより, 対応する次元の配列クラスを取得する
                                    -&gt; Klass::array_klass()  (再帰呼び出し)
                                       -&gt; objArrayKlass::array_klass_impl()
                                          -&gt; (同上)
</pre></div>

* JNI による配列処理 (See: [here](noj4FhtQM1.html) for details)
  
<div class="flow-abst"><pre>
jni_NewObjectArray()
-&gt; Klass::array_klass()
   -&gt; (同上)
</pre></div>

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






