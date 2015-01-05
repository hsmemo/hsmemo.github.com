---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理 (2) ： ロード処理の流れ (= SystemDictionary クラスによる resolve 処理)
---
[Up](nohUsh5oi7.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理 (2) ： ロード処理の流れ (= SystemDictionary クラスによる resolve 処理)

--- 
## 概要(Summary)
ロード処理は SystemDictionary クラスによって実行される.

SystemDictionary クラスは, クラス名 (及びクラスローダ) からクラス(klassOop)への対応付けを管理している
(See: SystemDictionary).
まだ SystemDictionary 内に存在しないクラスが要求された場合,
SystemDictionary クラスはそのクラスのロード処理を行う.

ロード処理では, 排他のために PlaceholderTable が用いられる (See: PlaceholderTable).
また, ロード対象のクラスファイルをパースして klassOop を生成する処理は
ClassFileParser::parseClassFile() に実装されている (See: [here](no2114rPX.html) for details).

## 備考(Notes)
SystemDictionary クラスは次のような public メソッドを持つ.

* SystemDictionary::resolve_or_null()
  
  指定されたクラスを返す (必要ならばクラスのロード処理も行う).
  見つからなかった場合は NULL を返す.


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Returns a class with a given class name and class loader.
      // Loads the class if needed. If not found NULL is returned.
```

* SystemDictionary::resolve_or_fail()
  
  指定されたクラスを返す (必要ならばクラスのロード処理も行う).
  見つからなかった場合は例外が出る.


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Returns a class with a given class name and class loader.  Loads the
      // class if needed. If not found a NoClassDefFoundError or a
      // ClassNotFoundException is thrown, depending on the value on the
      // throw_error flag.  For most uses the throw_error argument should be set
      // to true.
```

* SystemDictionary::resolve_from_stream()
  
  指定されたストリームからクラスファイルを読み取り, クラスのロード処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Resolve from stream (called by jni_DefineClass and JVM_DefineClass)
```

* SystemDictionary::parse_stream()
  
  指定されたストリームからクラスファイルを読み取り, クラスのロード処理を行う
  (SystemDictionary::resolve_from_stream() に類似のメソッドだが, SystemDictionary を更新しない).
  

```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Parse new stream. This won't update the system dictionary or
      // class hierarchy, simply parse the stream. Used by JVMTI RedefineClasses.
```


なお, InitOption の種別で Pre, Pre_JSR292 のものは SystemDictionary::resolve_or_fail(),
それ以降のものは SystemDictionary::resolve_or_null() を使用するらしい.


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      enum InitOption {
        Pre,                        // preloaded; error if not present
        Pre_JSR292,                 // preloaded if EnableInvokeDynamic
    
        // Order is significant.  Options before this point require resolve_or_fail.
        // Options after this point will use resolve_or_null instead.
    
        Opt,                        // preload tried; NULL if not present
        Opt_Only_JDK14NewRef,       // preload tried; use only with NewReflection
        Opt_Only_JDK15,             // preload tried; use only with JDK1.5+
        Opt_Kernel,                 // preload tried only #ifdef KERNEL
        OPTION_LIMIT,
        CEIL_LG_OPTION_LIMIT = 4    // OPTION_LIMIT <= (1<<CEIL_LG_OPTION_LIMIT)
      };
```


## 処理の流れ (概要)(Execution Flows : Summary)
### SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
-&gt; * 配列クラスの場合:
     -&gt; SystemDictionary::resolve_array_class_or_null()
        -&gt; * オブジェクト型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
             -&gt; SystemDictionary::resolve_instance_class_or_null()
                -&gt; (後述)
             -&gt; Klass::array_klass()
           * プリミティブ型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
             -&gt; Universe::typeArrayKlassObj()
             -&gt; Klass::array_klass()

   * 配列クラスではない場合:
     -&gt; SystemDictionary::resolve_instance_class_or_null()
        -&gt; (1) ロード対象のクラスに対応する PlaceholderEntry を PlaceholderTable 内に登録する.
               -&gt; PlaceholderTable::find_and_add()

           (1) 実際のロード処理を行う.
               -&gt; SystemDictionary::load_instance_class()
                  -&gt; 使用する ClassLoader が指定されているかどうかで 2通りに分岐.
                     * 使用する ClassLoader が指定されていない場合
                       (1) まずは, shared archive (Class Data Sharing の領域) から探す.
                           -&gt; SystemDictionary::load_shared_class()

                       (2) 見つからなければ, HotSpot 内蔵のクラスローダ(ブートストラップ・クラスローダ)でロードを試みる.
                           -&gt; ClassLoader::load_classfile()
                              -&gt; ClassFileParser::parseClassFile()
                                 -&gt; (See: <a href="no2114rPX.html">here</a> for details)
                              -&gt; ClassLoader::add_package()
  
                       (3) (#ifdef KERNEL の場合) #TODO
                           -&gt; SystemDictionary::download_and_retry_class_load()

                       (4) 以上の処理でクラスで見つかった場合は, SystemDictionary に登録する.
                           -&gt; SystemDictionary::find_or_define_instance_class()
                              -&gt; SystemDictionary::define_instance_class()
                                 -&gt; (1) クラスローダーにクラスを登録する
                                        -&gt; JavaCalls::call()
                                           -&gt; (See: <a href="no3059iJu.html">here</a> for details)
                                              -&gt; java.lang.ClassLoader.addClass()
                                    (2) クラスを SystemDictionary に追加する
                                        -&gt; SystemDictionary::add_to_hierarchy()
                                           -&gt; Universe::flush_dependents_on()
                                        -&gt; SystemDictionary::update_dictionary()
                                           -&gt; Dictionary::add_klass()
                                        -&gt; instanceKlass::eager_initialize()

                     * 使用する ClassLoader が指定されている場合
                       (1) 指定された ClassLoader の loadClassInternal() もしくは loadClass() でロードを行う.
                           -&gt; JavaCalls::call_special() or JavaCalls::call_virtual()
</pre></div>

### SystemDictionary::resolve_or_null(Symbol *class_name, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::resolve_or_null(Symbol *class_name, TRAPS)
-&gt; SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
   -&gt; (上述)
</pre></div>

### SystemDictionary::resolve_or_fail(Symbol *class_name, Handle class_loader, Handle protection_domain, bool throw_error, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::resolve_or_fail(Symbol *class_name, Handle class_loader, Handle protection_domain, bool throw_error, TRAPS)
-&gt; SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
   -&gt; (上記参照)
-&gt; SystemDictionary::handle_resolution_exception()
</pre></div>

### SystemDictionary::resolve_or_fail(Symbol *class_name, bool throw_error, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::resolve_or_fail(Symbol *class_name, bool throw_error, TRAPS)
-&gt; SystemDictionary::resolve_or_fail(Symbol *class_name, Handle class_loader, Handle protection_domain, bool throw_error, TRAPS)
   -&gt; (上述)
</pre></div>

### SystemDictionary::resolve_super_or_fail(Symbol *child_name, Symbol *class_name, Handle class_loader, Handle protection_domain, bool is_superclass, TRAPS)
<div class="flow-abst"><pre>
#TODO
</pre></div>

### SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, TRAPS)
-&gt; SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, KlassHandle host_klass, GrowableArray&lt; Handle &gt; *cp_patches, TRAPS)
   -&gt; (下記参照)
</pre></div>

### SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, KlassHandle host_klass, GrowableArray< Handle > *cp_patches, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, KlassHandle host_klass, GrowableArray&lt; Handle &gt; *cp_patches, TRAPS)
-&gt; ClassFileParser::parseClassFile()
   -&gt; (See: <a href="no2114rPX.html">here</a> for details)
-&gt; SystemDictionary::add_to_hierarchy()
</pre></div>

### SystemDictionary::resolve_from_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, bool verify, TRAPS)
<div class="flow-abst"><pre>
SystemDictionary::resolve_from_stream()
-&gt; (1) クラスファイルのパース処理を行う
       -&gt; ClassFileParser::parseClassFile()
          -&gt; (See: <a href="no2114rPX.html">here</a> for details)
   (1) SystemDictionary に登録する.
       * 並行に処理できる場合:
         -&gt; SystemDictionary::find_or_define_instance_class()
            -&gt; SystemDictionary::define_instance_class()
               -&gt; (下記参照)
       * 〃できない場合:
         -&gt; SystemDictionary::define_instance_class()
            -&gt; (1) クラスローダーにクラスを登録する
                   -&gt; JavaCalls::call()
                      -&gt; (See: <a href="no3059iJu.html">here</a> for details)
                         -&gt; java.lang.ClassLoader.addClass()
               (2) クラスを SystemDictionary に追加する
                   -&gt; SystemDictionary::add_to_hierarchy()
                      -&gt; Universe::flush_dependents_on()
                   -&gt; SystemDictionary::update_dictionary()
                      -&gt; Dictionary::add_klass()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### SystemDictionary::resolve_or_fail(Symbol *class_name, Handle class_loader, Handle protection_domain, bool throw_error, TRAPS)
(#Under Construction)
See: [here](no7517kFl.html) for details
### SystemDictionary::resolve_or_fail(Symbol *class_name, bool throw_error, TRAPS)
See: [here](no7517Aiy.html) for details
### SystemDictionary::resolve_or_null(Symbol *class_name, Handle class_loader, Handle protection_domain, TRAPS)
See: [here](no7517zfh.html) for details
### SystemDictionary::resolve_or_null(Symbol *class_name, TRAPS)
See: [here](no7517ZRI.html) for details
### SystemDictionary::resolve_super_or_fail(Symbol *child_name, Symbol *class_name, Handle class_loader, Handle protection_domain, bool is_superclass, TRAPS)
(#Under Construction)


### SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, TRAPS)
See: [here](no75170mb.html) for details
### SystemDictionary::parse_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, KlassHandle host_klass, GrowableArray< Handle > *cp_patches, TRAPS)
See: [here](no7517Cr0.html) for details
### SystemDictionary::resolve_from_stream(Symbol *class_name, Handle class_loader, Handle protection_domain, ClassFileStream *st, bool verify, TRAPS)
See: [here](no7625MJG.html) for details

### SystemDictionary::resolve_array_class_or_null()
See: [here](no7517ndW.html) for details
### SystemDictionary::resolve_instance_class_or_null()
(#Under Construction)
See: [here](no7517ERI.html) for details
### SystemDictionary::load_instance_class()
See: [here](no7517fmb.html) for details
### SystemDictionary::load_shared_class(Symbol* class_name, Handle class_loader, TRAPS)
See: [here](no7517w2n.html) for details
### ClassLoader::load_classfile()
See: [here](no7517t4c.html) for details
### SystemDictionary::find_or_define_instance_class()
See: [here](no7625QGu.html) for details
### SystemDictionary::define_instance_class()
(#Under Construction)
See: [here](no7625r3R.html) for details
### java.lang.ClassLoader.addClass()
See: [here](no7625TXr.html) for details
### SystemDictionary::add_to_hierarchy()
(#Under Construction)
See: [here](no26814kOG.html) for details
### Universe::flush_dependents_on()
(#Under Construction)


### SystemDictionary::update_dictionary()
See: [here](no26814_jZ.html) for details
### Dictionary::add_klass()
See: [here](no26814O3h.html) for details
### instanceKlass::eager_initialize()
(#Under Construction)
See: [here](no26814X2X.html) for details

### SystemDictionary::handle_resolution_exception()
(#Under Construction)







