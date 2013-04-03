---
layout: default
title: instanceKlass クラス関連のクラス (FieldClosure, FieldPrinter, OopMapBlock, instanceKlass, JNIid, PreviousVersionNode, PreviousVersionInfo, PreviousVersionWalker, 及びそれらの補助クラス(nmethodBucket, VerifyFieldClosure))
---
[Top](../index.html)

#### instanceKlass クラス関連のクラス (FieldClosure, FieldPrinter, OopMapBlock, instanceKlass, JNIid, PreviousVersionNode, PreviousVersionInfo, PreviousVersionWalker, 及びそれらの補助クラス(nmethodBucket, VerifyFieldClosure))

これらは, Java レベルでの「クラス」を表すためのクラス (See: [here](no7882m2Z.html) and [here](no2935YZn.html) for details).


### クラス一覧(class list)

  * [instanceKlass](#noFqMJRxTD)
  * [OopMapBlock](#no9B28NQID)
  * [FieldClosure](#no-SCi_zvW)
  * [FieldPrinter](#noFftUhqNK)
  * [JNIid](#nobGr7A20Q)
  * [PreviousVersionNode](#noIK7ottP2)
  * [PreviousVersionInfo](#norPjjnX6e)
  * [PreviousVersionWalker](#noIHmCK_Vr)
  * [nmethodBucket](#nopq-Jx8JU)
  * [VerifyFieldClosure](#no8_1qjHJi)


---
## <a name="noFqMJRxTD" id="noFqMJRxTD">instanceKlass</a>

### 概要(Summary)
instanceOopDesc 用の Klass クラス.
1つの instanceKlass オブジェクトが Java レベルでの 1つの「クラス」に対応する (See: [here](no7882m2Z.html) and [here](no2935YZn.html) for details).


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    // An instanceKlass is the VM level representation of a Java class.
    // It contains all information needed for at class at execution runtime.
```


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    class instanceKlass: public Klass {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
instanceKlassKlass::allocate_instance_klass() というファクトリメソッドが用意されており, その中で(のみ)生成されている. 
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている (See: [here](no7882m2Z.html) for details).

```
ClassFileParser::parseClassFile()
-> oopFactory::new_instanceKlass()
   -> instanceKlassKlass::allocate_instance_klass()
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * klassOop        _array_klasses
    
    この「クラス」の配列クラス.

  * objArrayOop     _methods
    
    この「クラス」のメソッドを表す methodOopDesc の配列

  * typeArrayOop    _method_ordering
    
    メソッドの元々の順番を表すための int 配列
    (_methods フィールド内の methodOopDesc はクラスファイル中の順序とは異なるため, 
    JVMTI や CDS で必要な場合には元の順序を覚えておく必要がある.
    (See: ClassFileParser::sort_methods()))

  * objArrayOop     _local_interfaces
    
    この「クラス」が実装しているインターフェースを表す klassOop 配列.

  * objArrayOop     _transitive_interfaces
    
    この「クラス」が実装しているインターフェース(の推移的閉包)を表す klassOop 配列.

  * typeArrayOop    _fields
    
    この「クラス」で定義されているフィールド情報
    (各フィールドに対して u2 7個分を使って情報を記録している (See: ClassFileParser::parse_fields())).

  * constantPoolOop _constants
    
    この「クラス」の Constant Pool 情報を格納する constantPoolOopDesc オブジェクト.

  * oop             _class_loader
    
    この「クラス」をロードした Class Loader (VM loader が使用された場合は NULL).

  * oop             _protection_domain
    
    この「クラス」のロード処理で使用された protection domain.

  * klassOop        _host_klass
    
    

  * objArrayOop     _signers
    
    

  * typeArrayOop    _inner_classes
    
    

  * klassOop        _implementors[implementors_limit]



  * typeArrayOop    _class_annotations



  * objArrayOop     _fields_annotations



  * objArrayOop     _methods_annotations



  * objArrayOop     _methods_parameter_annotations



  * objArrayOop     _methods_default_annotations




  * Symbol*         _source_file_name



  * Symbol*         _source_debug_extension



  * Symbol*         _generic_signature



  * Symbol*         _array_name




  * int             _nonstatic_field_size



  * int             _static_field_size



  * int             _static_oop_field_count



  * int             _nonstatic_oop_map_size



  * bool            _is_marked_dependent



  * bool            _rewritten



  * bool            _has_nonstatic_fields



  * bool            _should_verify_class



  * u2              _minor_version



  * u2              _major_version



  * ClassState      _init_state



  * Thread*         _init_thread



  * int             _vtable_len
    
    この「クラス」の vtable の大きさ (単位は word).

  * int             _itable_len

    この「クラス」の itable の大きさ (単位は word).

  * ReferenceType   _reference_type



  * OopMapCache*    volatile _oop_map_cache
    
    この「クラス」用の OopMapCache オブジェクト.

  * JNIid*          _jni_ids
    
    この「クラス」の static field 用の JNIid オブジェクト(の線形リスト).

  * jmethodID*      _methods_jmethod_ids



  * int*            _methods_cached_itable_indices



  * nmethodBucket*  _dependencies
    
    JIT コード精製時にこの「クラス」に関する仮定を使用した nmethod (を示すための nmethodBucket の線形リスト).

  * nmethod*        _osr_nmethods_head
    
    (この「クラス」のメソッドで OSR (On-Stack Replacement) 方式で JIT コードが生成されたものについては, ここに nmethod がある)

  * BreakpointInfo* _breakpoints
    
    

  * int             _nof_implementors




  * GrowableArray<PreviousVersionNode *>* _previous_versions



  * u2              _enclosing_method_class_index



  * u2              _enclosing_method_method_index




  * unsigned char * _cached_class_file_bytes



  * jint            _cached_class_file_len



  * JvmtiCachedClassFieldMap* _jvmti_cached_class_field_map



  * volatile u2     _idnum_allocated_count




  * (フィールドとしては定義されていない)   (下図中の embedded Java vtable follows here)
    
    ここにこの「クラス」用の vtable が格納されている
    (See: klassVtable, vtableEntry).

  * (フィールドとしては定義されていない)   (下図中の embedded Java itables follows here)

    ここにこの「クラス」用の itable が格納されている
    (See: klassItable, itableOffsetEntry, itableMethodEntry).

  * (下図中ではここに static field が存在することになっている("embedded static fields follows here")が実際には存在しない.
     static field は対応する mirror オブジェクト内に存在する.)

  * (フィールドとしては定義されていない)   (下図中の embedded nonstatic oop-map blocks follows here)
    
    ここにこの「クラス」用の oop-map が格納されている
    (See: OopMapBlock).


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      //
      // The oop block.  See comment in klass.hpp before making changes.
      //
    
      // Array classes holding elements of this class.
      klassOop        _array_klasses;
      // Method array.
      objArrayOop     _methods;
      // Int array containing the original order of method in the class file (for
      // JVMTI).
      typeArrayOop    _method_ordering;
      // Interface (klassOops) this class declares locally to implement.
      objArrayOop     _local_interfaces;
      // Interface (klassOops) this class implements transitively.
      objArrayOop     _transitive_interfaces;
      // Instance and static variable information, 5-tuples of shorts [access, name
      // index, sig index, initval index, offset].
      typeArrayOop    _fields;
      // Constant pool for this class.
      constantPoolOop _constants;
      // Class loader used to load this class, NULL if VM loader used.
      oop             _class_loader;
      // Protection domain.
      oop             _protection_domain;
      // Host class, which grants its access privileges to this class also.
      // This is only non-null for an anonymous class (JSR 292 enabled).
      // The host class is either named, or a previously loaded anonymous class.
      klassOop        _host_klass;
      // Class signers.
      objArrayOop     _signers;
      // inner_classes attribute.
      typeArrayOop    _inner_classes;
      // Implementors of this interface (not valid if it overflows)
      klassOop        _implementors[implementors_limit];
      // Annotations for this class, or null if none.
      typeArrayOop    _class_annotations;
      // Annotation objects (byte arrays) for fields, or null if no annotations.
      // Indices correspond to entries (not indices) in fields array.
      objArrayOop     _fields_annotations;
      // Annotation objects (byte arrays) for methods, or null if no annotations.
      // Index is the idnum, which is initially the same as the methods array index.
      objArrayOop     _methods_annotations;
      // Annotation objects (byte arrays) for methods' parameters, or null if no
      // such annotations.
      // Index is the idnum, which is initially the same as the methods array index.
      objArrayOop     _methods_parameter_annotations;
      // Annotation objects (byte arrays) for methods' default values, or null if no
      // such annotations.
      // Index is the idnum, which is initially the same as the methods array index.
      objArrayOop     _methods_default_annotations;
    
      //
      // End of the oop block.
      //
    
      // Name of source file containing this klass, NULL if not specified.
      Symbol*         _source_file_name;
      // the source debug extension for this klass, NULL if not specified.
      Symbol*         _source_debug_extension;
      // Generic signature, or null if none.
      Symbol*         _generic_signature;
      // Array name derived from this class which needs unreferencing
      // if this class is unloaded.
      Symbol*         _array_name;
    
      // Number of heapOopSize words used by non-static fields in this klass
      // (including inherited fields but after header_size()).
      int             _nonstatic_field_size;
      int             _static_field_size;    // number words used by static fields (oop and non-oop) in this klass
      int             _static_oop_field_count;// number of static oop fields in this klass
      int             _nonstatic_oop_map_size;// size in words of nonstatic oop map blocks
      bool            _is_marked_dependent;  // used for marking during flushing and deoptimization
      bool            _rewritten;            // methods rewritten.
      bool            _has_nonstatic_fields; // for sizing with UseCompressedOops
      bool            _should_verify_class;  // allow caching of preverification
      u2              _minor_version;        // minor version number of class file
      u2              _major_version;        // major version number of class file
      ClassState      _init_state;           // state of class
      Thread*         _init_thread;          // Pointer to current thread doing initialization (to handle recusive initialization)
      int             _vtable_len;           // length of Java vtable (in words)
      int             _itable_len;           // length of Java itable (in words)
      ReferenceType   _reference_type;       // reference type
      OopMapCache*    volatile _oop_map_cache;   // OopMapCache for all methods in the klass (allocated lazily)
      JNIid*          _jni_ids;              // First JNI identifier for static fields in this class
      jmethodID*      _methods_jmethod_ids;  // jmethodIDs corresponding to method_idnum, or NULL if none
      int*            _methods_cached_itable_indices;  // itable_index cache for JNI invoke corresponding to methods idnum, or NULL
      nmethodBucket*  _dependencies;         // list of dependent nmethods
      nmethod*        _osr_nmethods_head;    // Head of list of on-stack replacement nmethods for this class
      BreakpointInfo* _breakpoints;          // bpt lists, managed by methodOop
      int             _nof_implementors;     // No of implementors of this interface (zero if not an interface)
      // Array of interesting part(s) of the previous version(s) of this
      // instanceKlass. See PreviousVersionWalker below.
      GrowableArray<PreviousVersionNode *>* _previous_versions;
      u2              _enclosing_method_class_index;  // Constant pool index for class of enclosing method, or 0 if none
      u2              _enclosing_method_method_index; // Constant pool index for name and type of enclosing method, or 0 if none
      // JVMTI fields can be moved to their own structure - see 6315920
      unsigned char * _cached_class_file_bytes;       // JVMTI: cached class file, before retransformable agent modified it in CFLH
      jint            _cached_class_file_len;         // JVMTI: length of above
      JvmtiCachedClassFieldMap* _jvmti_cached_class_field_map;  // JVMTI: used during heap iteration
      volatile u2     _idnum_allocated_count;         // JNI/JVMTI: increments with the addition of methods, old ids don't change
    
      // embedded Java vtable follows here
      // embedded Java itables follows here
      // embedded static fields follows here
      // embedded nonstatic oop-map blocks follows here
```


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    //  instanceKlass layout:
    //    [header                     ] klassOop
    //    [klass pointer              ] klassOop
    //    [C++ vtbl pointer           ] Klass
    //    [subtype cache              ] Klass
    //    [instance size              ] Klass
    //    [java mirror                ] Klass
    //    [super                      ] Klass
    //    [access_flags               ] Klass
    //    [name                       ] Klass
    //    [first subklass             ] Klass
    //    [next sibling               ] Klass
    //    [array klasses              ]
    //    [methods                    ]
    //    [local interfaces           ]
    //    [transitive interfaces      ]
    //    [number of implementors     ]
    //    [implementors               ] klassOop[2]
    //    [fields                     ]
    //    [constants                  ]
    //    [class loader               ]
    //    [protection domain          ]
    //    [signers                    ]
    //    [source file name           ]
    //    [inner classes              ]
    //    [static field size          ]
    //    [nonstatic field size       ]
    //    [static oop fields size     ]
    //    [nonstatic oop maps size    ]
    //    [has finalize method        ]
    //    [deoptimization mark bit    ]
    //    [initialization state       ]
    //    [initializing thread        ]
    //    [Java vtable length         ]
    //    [oop map cache (stack maps) ]
    //    [EMBEDDED Java vtable             ] size in words = vtable_len
    //    [EMBEDDED nonstatic oop-map blocks] size in words = nonstatic_oop_map_size
    //
    //    The embedded nonstatic oop-map blocks are short pairs (offset, length) indicating
    //    where oops are located in instances of this klass.
```




### 詳細(Details)
See: [here](../doxygen/classinstanceKlass.html) for details

---
## <a name="no9B28NQID" id="no9B28NQID">OopMapBlock</a>

### 概要(Summary)
instanceKlass クラス内で使用される補助クラス.

Garbage Collection 処理用のクラス.
「その Java クラス(= instanceKlass) に対応するオブジェクトでは, どこに oop が存在するか? (= ポインタフィールドはどこか?)」, 
という情報を格納している
(正確には, instanceKlass オブジェクト内では複数の OopMapBlock オブジェクトを用いてこの情報を記録している.
oop フィールドが連続している場合にはその分はまとめて記録しており, 
1つの OopMapBlock オブジェクトが oop フィールドからなる連続領域 1つに対応する.)


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    // ValueObjs embedded in klass. Describes where oops are located in instances of
    // this klass.
    class OopMapBlock VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 instanceKlass オブジェクト内に格納されている.

なお, OopMapBlock オブジェクトを格納している領域にはフィールド名は付けられていない (See: instanceKlass).
格納している領域の先頭アドレスは instanceKlass::start_of_nonstatic_oop_maps() メソッドで取得できる.
また, 領域の長さは instanceKlass::_nonstatic_oop_map_size フィールドに入っている
(OopMapBlock::size_in_words() 単位での長さでよければ instanceKlass::nonstatic_oop_map_count() でも取得できる).

#### 生成箇所(where its instances are created)
メモリ領域は oopFactory::new_instanceKlass() 内で(のみ)確保されている
(なお, instanceKlass オブジェクト内で OopMapBlock が占める大きさについては, 
 ClassFileParser::compute_oop_map_count() で計算している).

そのメモリ領域中に個別の OopMapBlock オブジェクトを書き込む作業は, 
ClassFileParser::fill_oop_maps() 内で(のみ)行われている.

#### 使用箇所(where its instances are used)
以下の箇所で使用されている. (#TODO 他にもあるか??)

* InstanceKlass_OOP_MAP_ITERATE()

  (See: instanceKlass::oop_follow_contents(), instanceKlass::oop_oop_iterate_*(), 
  instanceKlass::oop_adjust_pointers(), instanceKlass::oop_update_pointers())

* InstanceKlass_OOP_MAP_REVERSE_ITERATE()
  
  (See: instanceKlass::oop_oop_iterate_backwards_*(), instanceKlass::oop_push_contents())

* InstanceKlass_BOUNDED_OOP_MAP_ITERATE()
  
  (See: instanceKlass::oop_oop_iterate_*_m())


### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ)

* int  _offset
  
  oop フィールドの連続領域がどこから始まるか(= 先頭からのオフセット), を示すフィールド (単位は byte).

* uint _count
  
  oop(または narrowOop)が何個連続しているか (= その連続領域の大きさ), を示すフィールド (単位は oop の個数).


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      int  _offset;
      uint _count;
```




### 詳細(Details)
See: [here](../doxygen/classOopMapBlock.html) for details

---
## <a name="no-SCi_zvW" id="no-SCi_zvW">FieldClosure</a>

### 概要(Summary)
fieldDescriptor クラス用のユーティリティ・クラス.

fieldDescriptor オブジェクトへのポインタに対して何らかの処理を行う Closure クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    // This is used in iterators below.
    class FieldClosure: public StackObj {
```

### 使われ方(Usage)
fieldDescriptor* を処理する do_field() メソッドを備えている.


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      virtual void do_field(fieldDescriptor* fd) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classFieldClosure.html) for details

---
## <a name="noFftUhqNK" id="noFftUhqNK">FieldPrinter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

フィールドに関する情報を出力する FieldClosure クラス.


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    #ifndef PRODUCT
    // Print fields.
    // If "obj" argument to constructor is NULL, prints static fields, otherwise prints non-static fields.
    class FieldPrinter: public FieldClosure {
```

### 使われ方(Usage)
instanceKlassKlass::oop_print_on() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classFieldPrinter.html) for details

---
## <a name="nobGr7A20Q" id="nobGr7A20Q">JNIid</a>

### 概要(Summary)
JNI 及び JVMTI の処理を高速化するため(?)のクラス.
JNI/JVMTI によるフィールドアクセス処理で使用される (より正確に言うと「static field」へのアクセス処理で使用される)
(See: [here](no5248c5L.html) for details).

JNI で取得される jfieldID 値は, 
対象が instance field の場合と static field の場合で実装が異なっている.
static field の場合, jfieldID の正体は対応する JNIid オブジェクトへのポインタになっている.


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      // JNI identifier support (for static fields - for jni performance)
```


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    /* JNIid class for jfieldIDs only */
    class JNIid: public CHeapObj {
```

(ところで, "for performance" と書いてあるが, どこら辺が高速化されている??
 instance field と同じような実装でもあまり変わらないような気もするが... #TODO)

### 使われ方(Usage)
#### 使用方法の概要(how to use)
instanceKlass::jni_id_for() で, 指定したオフセット位置に対応する JNIid オブジェクトが取得される
(もし対応するものがなければ, この際に遅延生成される).

取得した JNIid オブジェクトは, jfieldIDWorkaround::to_static_jfieldID() で jfieldID に変換される.
(といっても実際には何もしていない. 単に型キャストしているだけ).

#### インスタンスの格納場所(where its instances are stored)
各 instanceKlass オブジェクトの _jni_ids フィールドに(のみ)格納されている.

(正確には, このフィールドは JNIid の線形リストを格納するフィールド.
JNIid オブジェクトは _next フィールドで次の JNIid オブジェクトを指せる構造になっている. 
その instanceKlass オブジェクト内で生成した JNIid オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
instanceKlass::jni_id_for_impl() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* JNI の GetStaticFieldID() 関数の処理
  jni_GetStaticFieldID()
  -> instanceKlass::jni_id_for()
     -> instanceKlass::jni_id_for_impl()

* JNI の FromReflectedField() 関数の処理
  jni_FromReflectedField()
  -> instanceKlass::jni_id_for()
     -> (同上)

* JVMTI の field access 監視機能の処理 (See: SetFieldAccessWatch())
  InterpreterRuntime::post_field_access()
  -> jfieldIDWorkaround::to_jfieldID()
     -> instanceKlass::jni_id_for()
        -> (同上)

* JVMTI の field modification 監視機能の処理 (See: SetFieldModificationWatch())
  InterpreterRuntime::post_field_access()
  -> jfieldIDWorkaround::to_jfieldID()
     -> instanceKlass::jni_id_for()
        -> (同上)

* JVMTI の GetClassFields() 関数の処理
  JvmtiEnv::GetClassFields()
  -> jfieldIDWorkaround::to_jfieldID()
     -> instanceKlass::jni_id_for()
        -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classJNIid.html) for details

---
## <a name="noIK7ottP2" id="noIK7ottP2">PreviousVersionNode</a>

### 概要(Summary)
JVMTI の RedefineClass 機能 (RedefineClasses(), RetransformClasses()) の実装のための補助クラス.
RedefineClass() によって生じた EMCP メソッド (およびそれらに対応する古い Constant Pool 情報) を記録しておくために使用される
(See: [here](no2935-Vj.html) for details).


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    // A collection point for interesting information about the previous
    // version(s) of an instanceKlass. This class uses weak references to
    // the information so that the information may be collected as needed
    // by the system. If the information is shared, then a regular
    // reference must be used because a weak reference would be seen as
    // collectible. A GrowableArray of PreviousVersionNodes is attached
    // to the instanceKlass as needed. See PreviousVersionWalker below.
    class PreviousVersionNode : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
現状の実装では, RedefineClass() 処理が起こると必ず1つの PreviousVersionNode オブジェクトが生成される.
この中には, その時点での Constant Pool 情報 (constantPoolOopDesc), 
および(もし EMCP メソッドが生じていれば) EMCP メソッドに対応する methodOopDesc が記録されている.

(なお, 追加する処理を行う関数内(instanceKlass::add_previous_version())では, 
不要になった PreviousVersionNode を削除する処理もついでに行われている)

その後, 記録された情報は PreviousVersionWalker 経由で参照される.

#### インスタンスの格納場所(where its instances are stored)
各 instanceKlass オブジェクトの _previous_versions フィールドに(のみ)格納されている.

(正確には, このフィールドは PreviousVersionNode の配列を格納するフィールド.
その instanceKlass オブジェクト内で生成した PreviousVersionNode オブジェクトは全てこの中に格納されている)

#### 生成箇所(where its instances are created)
instanceKlass::add_previous_version() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている (See: [here](no2935-Vj.html) for details).

```
VM_RedefineClasses::doit()
-> VM_RedefineClasses::redefine_single_class()
   -> instanceKlass::add_previous_version()
```

#### 削除箇所(where its instances are deleted)
以下の箇所で(のみ)削除されている.

* instanceKlass::add_previous_version() 
* instanceKlass::release_C_heap_structures()




### 詳細(Details)
See: [here](../doxygen/classPreviousVersionNode.html) for details

---
## <a name="norPjjnX6e" id="norPjjnX6e">PreviousVersionInfo</a>

### 概要(Summary)
JVMTI の RedefineClass 機能 (RedefineClasses(), RetransformClasses()) の実装のための補助クラス (See: [here](no2935-Vj.html) for details).

PreviousVersionNode オブジェクト内に記録した情報を参照するためのクラス
(PreviousVersionNode として記録された情報は PreviousVersionWalker 経由でアクセスされる.
 その際には, PreviousVersionNode ではなく PreviousVersionInfo オブジェクトとして参照される).

中身自体は PreviousVersionNode とほぼ同じだが, それぞれの情報を Handle 化した形で保持している.

(なお, このクラスは ResourceObj クラスだが, C ヒープ上に確保されている)


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    // A Handle-ized version of PreviousVersionNode.
    class PreviousVersionInfo : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
PreviousVersionNode を iterate する関数である
PreviousVersionWalker::next_previous_version() を呼び出すと, 
返り値として PreviousVersionInfo オブジェクトが返される.

#### 生成箇所(where its instances are created)
PreviousVersionWalker::next_previous_version() 内で(のみ)生成されている.

#### 削除箇所(where its instances are deleted)
以下の箇所で削除される

* PreviousVersionWalker::next_previous_version()

  1つ前の PreviousVersionInfo オブジェクトが削除される.

* PreviousVersionWalker::~PreviousVersionWalker()

  最後に使った PreviousVersionInfo オブジェクトが削除される.




### 詳細(Details)
See: [here](../doxygen/classPreviousVersionInfo.html) for details

---
## <a name="noIHmCK_Vr" id="noIHmCK_Vr">PreviousVersionWalker</a>

### 概要(Summary)
JVMTI の RedefineClass 機能 (RedefineClasses(), RetransformClasses()) の実装のための補助クラス
(See: [here](no2935-Vj.html) for details).

PreviousVersionNode として記録された情報をたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    // Helper object for walking previous versions. This helper cleans up
    // the Handles that it allocates when the helper object is destroyed.
    // The PreviousVersionInfo object returned by next_previous_version()
    // is only valid until a subsequent call to next_previous_version() or
    // the helper object is destroyed.
    class PreviousVersionWalker : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. PreviousVersionWalker 型の局所変数を宣言する.
2. PreviousVersionWalker::next_previous_version() で, 
   次の PreviousVersionNode オブジェクトを (PreviousVersionInfo オブジェクトという形で) 取得できる.
   全て辿り終わった場合は NULL が返される.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* instanceKlassKlass::oop_print_on() 内
* JvmtiBreakpoint::each_method_version_do() 内
* VM_RedefineClasses::adjust_cpool_cache_and_vtable() 内
* VM_RedefineClasses::redefine_single_class() 内




### 詳細(Details)
See: [here](../doxygen/classPreviousVersionWalker.html) for details

---
## <a name="nopq-Jx8JU" id="nopq-Jx8JU">nmethodBucket</a>

### 概要(Summary)
JIT コンパイラ用のクラス.

JIT コンパイラが特定のクラスへの仮定に基づいてコードを精製した場合, 
そのクラスに変更があった場合にはコードを脱最適化する必要がある.
nmethodBucket クラスは, どの nmethod がどのクラスに依存しているかを記録しておくためのクラス.


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.cpp))
    //
    // nmethodBucket is used to record dependent nmethods for
    // deoptimization.  nmethod dependencies are actually <klass, method>
    // pairs but we really only care about the klass part for purposes of
    // finding nmethods which might need to be deoptimized.  Instead of
    // recording the method, a count of how many times a particular nmethod
    // was recorded is kept.  This ensures that any recording errors are
    // noticed since an nmethod should be removed as many times are it's
    // added.
    //
    class nmethodBucket {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 instanceKlass オブジェクトの _dependencies フィールドに(のみ)格納されている.

(正確には, このフィールドは nmethodBucket の線形リストを格納するフィールド.
nmethodBucket オブジェクトは _next フィールドで次の nmethodBucket オブジェクトを指せる構造になっている.
その instanceKlass オブジェクト内で生成した nmethodBucket オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
instanceKlass::add_dependent_nmethod() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* C1 の場合:
  (略) (See: )
  -> Compilation::install_code()
     -> ciEnv::register_method()
        -> nmethod::new_nmethod()
           -> instanceKlass::add_dependent_nmethod()

* C2 の場合:
  (略) (See: [here](no3718SNC.html) for details)
  -> Compile::Compile()
     -> ciEnv::register_method()
        -> nmethod::new_nmethod()
           -> instanceKlass::add_dependent_nmethod()

* Shark の場合:
  (略) (See: )
  -> SharkCompiler::compile_method()
     -> ciEnv::register_method()
        -> nmethod::new_nmethod()
           -> instanceKlass::add_dependent_nmethod()
```

#### 使用箇所(where its instances are used)
instanceKlass::mark_dependent_nmethods() 内で(のみ)使用されている
(この関数はクラス階層に変更があった場合に呼び出され, ここで印が付けられたものが deopt 対象になる).
この関数は, 現在は以下のパスで(のみ)呼び出されている.

```
SystemDictionary::parse_stream()
-> SystemDictionary::add_to_hierarchy()
   -> Universe::flush_dependents_on()
      -> CodeCache::mark_for_deoptimization(DepChange& changes)
         -> instanceKlass::mark_dependent_nmethods()

SystemDictionary::define_instance_class()
-> SystemDictionary::add_to_hierarchy()
   -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classnmethodBucket.html) for details

---
## <a name="no8_1qjHJi" id="no8_1qjHJi">VerifyFieldClosure</a>

### 概要(Summary)
instanceKlass 内で使用される補助クラス.

フィールドに入っているポインタ値が Java ヒープ内を指しており,
かつ差し先が妥当な oop または NULL であることをチェックする.


```
    ((cite: hotspot/src/share/vm/oops/instanceKlass.cpp))
    // Verification
    
    class VerifyFieldClosure: public OopClosure {
```

### 使われ方(Usage)
instanceKlass::oop_verify_on() 内で(のみ)使用されている.

### 備考(Notes)
同名のクラスが hotspot/src/share/vm/oops/instanceKlassKlass.cpp にいたりするが特に関係は無い模様.




### 詳細(Details)
See: [here](../doxygen/classVerifyFieldClosure.html) for details

---
