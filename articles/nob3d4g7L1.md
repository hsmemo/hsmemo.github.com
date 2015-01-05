---
layout: default
title: constantPoolCacheOopDesc クラス関連のクラス (ConstantPoolCacheEntry, constantPoolCacheOopDesc, 及びそれらの補助クラス(LocalOopClosure))
---
[Top](../index.html)

#### constantPoolCacheOopDesc クラス関連のクラス (ConstantPoolCacheEntry, constantPoolCacheOopDesc, 及びそれらの補助クラス(LocalOopClosure))

これらは, Constant Pool 情報(の中でも特によく参照される「フィールド情報」と「メソッド情報」)をキャッシュして処理を高速化するためのクラス.


### クラス一覧(class list)

  * [constantPoolCacheOopDesc](#noXgmqGTsm)
  * [ConstantPoolCacheEntry](#nooatpR40u)
  * [LocalOopClosure](#noWi05g_nR)


---
## <a name="noXgmqGTsm" id="noXgmqGTsm">constantPoolCacheOopDesc</a>

### 概要(Summary)
Constant Pool 内の「フィールド情報」及び「メソッド情報」をキャッシュし, 処理を高速化するためのクラス.
1つの constantPoolCacheOopDesc オブジェクトが 1つの Constant Pool (= 1つのクラス) に対応する.

なお, constantPoolCacheOopDesc 自体は ConstantPoolCacheEntry を入れておくコンテナクラス.
実際のキャッシュ情報は ConstantPoolCacheEntry オブジェクト内に格納されている.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.hpp))
    // A constant pool cache is a runtime data structure set aside to a constant pool. The cache
    // holds interpreter runtime information for all field access and invoke bytecodes. The cache
    // is created and initialized before a class is actively used (i.e., initialized), the indivi-
    // dual cache entries are filled at resolution (i.e., "link") time (see also: rewriter.*).
    
    class constantPoolCacheOopDesc: public oopDesc {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
constantPoolOopDesc オブジェクトの _cache フィールドに(のみ)格納されている.

(なお, 逆に constantPoolCacheOopDesc 側から対応する constantPoolOopDesc を参照するには _constant_pool フィールドを用いればいい模様)

#### 生成箇所(where its instances are created)
constantPoolCacheKlass::allocate() というファクトリメソッドが用意されており, その中で生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
* クラスの link 処理時

  instanceKlass::link_class_impl()
  -&gt; instanceKlass::rewrite_class()
     -&gt; Rewriter::rewrite(instanceKlassHandle klass, TRAPS)
        -&gt; Rewriter::Rewriter()
           -&gt; Rewriter::make_constant_pool_cache()
              -&gt; oopFactory::new_constantPoolCache()
                 -&gt; constantPoolCacheKlass::allocate()

* JVMTI の RedefineClasses() 及び RetransformClasses() の処理時

  VM_RedefineClasses::doit_prologue()
  -&gt; VM_RedefineClasses::load_new_class_versions()
     -&gt; Rewriter::rewrite(instanceKlassHandle klass, TRAPS)
        -&gt; (同上)

* MethodHandle に対応する methodOop の生成処理時

  MethodHandleCompiler::compile()
  -&gt; MethodHandleCompiler::get_method_oop()
     -&gt; Rewriter::rewrite(instanceKlassHandle klass, constantPoolHandle cpool, objArrayHandle methods, TRAPS)
        -&gt; Rewriter::Rewriter()
           -&gt; (同上)
</pre></div>

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * int             _length

    自分自身 (= この constantPoolCacheOopDesc オブジェクト) の大きさ.

  * constantPoolOop _constant_pool

    この constantPoolCacheOopDesc に対応する constantPoolOopDesc オブジェクト

また, これ以降のフィールド宣言されていない領域に ConstantPoolCacheEntry オブジェクトが格納されている
(この部分の大きさは _length フィールドで把握可能).


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.hpp))
      int             _length;
      constantPoolOop _constant_pool;                // the corresponding constant pool
```

### 備考(Notes)
なお, 実際の使用箇所では constantPoolCacheOop という別名(もしくはラッパークラス)で使われることが多い (See: constantPoolCacheOop).




### 詳細(Details)
See: [here](../doxygen/classconstantPoolCacheOopDesc.html) for details

---
## <a name="nooatpR40u" id="nooatpR40u">ConstantPoolCacheEntry</a>

### 概要(Summary)
constantPoolCacheOopDesc クラス用の補助クラス.

Constant Pool 内の「フィールド情報」及び「メソッド情報」をキャッシュし, 処理を高速化するためのクラス.
1つの ConstantPoolCacheEntry オブジェクトが 1つの Constant Pool エントリに対応する.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.hpp))
    class ConstantPoolCacheEntry VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
constantPoolCacheOopDesc オブジェクト内に格納されている.

なお, ConstantPoolCacheEntry オブジェクトを格納している領域にはフィールド名は付けられていない.
(constantPoolCacheOopDesc の全てのフィールドが終わった後の領域に配置されている)

アクセスするには constantPoolCacheOopDesc::entry_at() メソッドを用いる.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.hpp))
      // Fetches the entry at the given index.
      // The entry may be either primary or secondary.
      // In either case the index must not be encoded or byte-swapped in any way.
      ConstantPoolCacheEntry* entry_at(int i) const {
        assert(0 <= i && i < length(), "index out of bounds");
        return base() + i;
      }
```

(なお, constantPoolCacheOopDesc::base() は以下のように定義されている)


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.hpp))
      ConstantPoolCacheEntry* base() const           { return (ConstantPoolCacheEntry*)((address)this + in_bytes(base_offset())); }
```

#### 生成箇所(where its instances are created)
InterpreterRuntime::resolve_get_put() 及び InterpreterRuntime::resolve_invoke() 内で,
resolve 結果が対応する ConstantPoolCacheEntry にセットされる.

### 内部構造(Internal structure)
ConstantPoolCacheEntry 内の各フィールドの使われ方は以下の通り.

* _flags は, フィールドアクセスかメソッド呼び出しかで, フォーマットが異なる.
* メソッド呼び出しの場合, invokevirtual 以外の invoke* は _f1 と b1 だけを使い, invokevirtual は _f2 と b2 だけを使うとのこと.
  (<= f2 は invokeinterface の場合にも使われている気もするが... #TODO)
* フィールドアクセス ({put|get}{field|static}) の場合は, 全てのフィールド(_f1,_f2,_flags,_indices)を使う模様.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.hpp))
    // A ConstantPoolCacheEntry describes an individual entry of the constant
    // pool cache. There's 2 principal kinds of entries: field entries for in-
    // stance & static field access, and method entries for invokes. Some of
    // the entry layout is shared and looks as follows:
    //
    // bit number |31                0|
    // bit length |-8--|-8--|---16----|
    // --------------------------------
    // _indices   [ b2 | b1 |  index  ]
    // _f1        [  entry specific   ]
    // _f2        [  entry specific   ]
    // _flags     [t|f|vf|v|m|h|unused|field_index] (for field entries)
    // bit length |4|1|1 |1|1|0|---7--|----16-----]
    // _flags     [t|f|vf|v|m|h|unused|eidx|psze] (for method entries)
    // bit length |4|1|1 |1|1|1|---7--|-8--|-8--]
    
    // --------------------------------
    //
    // with:
    // index  = original constant pool index
    // b1     = bytecode 1
    // b2     = bytecode 2
    // psze   = parameters size (method entries only)
    // eidx   = interpreter entry index (method entries only)
    // field_index = index into field information in holder instanceKlass
    //          The index max is 0xffff (max number of fields in constant pool)
    //          and is multiplied by (instanceKlass::next_offset) when accessing.
    // t      = TosState (see below)
    // f      = field is marked final (see below)
    // vf     = virtual, final (method entries only : is_vfinal())
    // v      = field is volatile (see below)
    // m      = invokeinterface used for method in class Object (see below)
    // h      = RedefineClasses/Hotswap bit (see below)
    //
    // The flags after TosState have the following interpretation:
    // bit 27: f flag  true if field is marked final
    // bit 26: vf flag true if virtual final method
    // bit 25: v flag true if field is volatile (only for fields)
    // bit 24: m flag true if invokeinterface used for method in class Object
    // bit 23: 0 for fields, 1 for methods
    //
    // The flags 31, 30, 29, 28 together build a 4 bit number 0 to 8 with the
    // following mapping to the TosState states:
    //
    // btos: 0
    // ctos: 1
    // stos: 2
    // itos: 3
    // ltos: 4
    // ftos: 5
    // dtos: 6
    // atos: 7
    // vtos: 8
    //
    // Entry specific: field entries:
    // _indices = get (b1 section) and put (b2 section) bytecodes, original constant pool index
    // _f1      = field holder
    // _f2      = field offset in words
    // _flags   = field type information, original field index in field holder
    //            (field_index section)
    //
    // Entry specific: method entries:
    // _indices = invoke code for f1 (b1 section), invoke code for f2 (b2 section),
    //            original constant pool index
    // _f1      = method for all but virtual calls, unused by virtual calls
    //            (note: for interface calls, which are essentially virtual,
    //             contains klassOop for the corresponding interface.
    //            for invokedynamic, f1 contains the CallSite object for the invocation
    // _f2      = method/vtable index for virtual calls only, unused by all other
    //            calls.  The vf flag indicates this is a method pointer not an
    //            index.
    // _flags   = field type info (f section),
    //            virtual final entry (vf),
    //            interpreter entry index (eidx section),
    //            parameter size (psze section)
    //
    // Note: invokevirtual & invokespecial bytecodes can share the same constant
    //       pool entry and thus the same constant pool cache entry. All invoke
    //       bytecodes but invokevirtual use only _f1 and the corresponding b1
    //       bytecode, while invokevirtual uses only _f2 and the corresponding
    //       b2 bytecode.  The value of _flags is shared for both types of entries.
    //
    // The fields are volatile so that they are stored in the order written in the
    // source code.  The _indices field with the bytecode must be written last.
```




### 詳細(Details)
See: [here](../doxygen/classConstantPoolCacheEntry.html) for details

---
## <a name="noWi05g_nR" id="noWi05g_nR">LocalOopClosure</a>

### 概要(Summary)
ConstantPoolCacheEntry 内で使用される補助クラス(Closureクラス).

oop を引数とする関数を OopClosure として使うためのラッパークラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.cpp))
    class LocalOopClosure: public OopClosure {
```

### 使われ方(Usage)
ConstantPoolCacheEntry::oops_do() 内で(のみ)使用されている.

### 内部構造(Internal structure)
コンストラクタで oop を引数とする関数を受け取る. do_oop() ではその関数を処理対象に適用するだけ.


```cpp
    ((cite: hotspot/src/share/vm/oops/cpCacheOop.cpp))
      LocalOopClosure(void f(oop*))        { _f = f; }
      virtual void do_oop(oop* o)          { _f(o); }
```




### 詳細(Details)
See: [here](../doxygen/classLocalOopClosure.html) for details

---
