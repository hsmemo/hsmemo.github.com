---
layout: default
title: constantPoolOopDesc クラス関連のクラス (CPSlot, constantPoolOopDesc, SymbolHashMapEntry, SymbolHashMapBucket, SymbolHashMap)
---
[Top](../index.html)

#### constantPoolOopDesc クラス関連のクラス (CPSlot, constantPoolOopDesc, SymbolHashMapEntry, SymbolHashMapBucket, SymbolHashMap)

これらは, 各クラスの Constant Pool 情報を管理するためのクラス (See: [here](no2935KOa.html) for details).


### クラス一覧(class list)

  * [constantPoolOopDesc](#noq9zeO3n4)
  * [CPSlot](#noP1NB7bgV)
  * [SymbolHashMap](#no2AXVDs-h)
  * [SymbolHashMapEntry](#noQrjEtvyC)
  * [SymbolHashMapBucket](#no7Ma4uE1H)


---
## <a name="noq9zeO3n4" id="noq9zeO3n4">constantPoolOopDesc</a>

### 概要(Summary)
各クラスの Constant Pool 情報を管理するためのクラス.
1つの constantPoolOopDesc オブジェクトが 1つのクラスに対応する (See: [here](no2935KOa.html) for details).


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
    // A constantPool is an array containing class constants as described in the
    // class file.
    //
    // Most of the constant pool entries are written during class parsing, which
    // is safe.  For klass and string types, the constant pool entry is
    // modified when the entry is resolved.  If a klass or string constant pool
    // entry is read without a lock, only the resolved state guarantees that
    // the entry in the constant pool is a klass or String object and
    // not a Symbol*.
```


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
    class constantPoolOopDesc : public oopDesc {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 instanceKlass オブジェクトの _constants フィールド

  そのクラス用の Constant Pool 情報を格納.

* 各 methodOopDesc オブジェクトの _constants フィールド

  そのメソッド用の Constant Pool 情報を格納. (そのメソッドが属するクラス用の constantPoolOopDesc と同じオブジェクト)

* 各 constantPoolCacheOopDesc オブジェクトの _constant_pool フィールド

  その constantPoolCacheOopDesc に対応する constantPoolOopDesc オブジェクト.
  
#### 生成箇所(where its instances are created)
constantPoolKlass::allocate() というファクトリメソッドが用意されており, その中で生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
* クラスファイルのパース処理時

  ClassFileParser::parseClassFile()
  -> ClassFileParser::parse_constant_pool()
     -> oopFactory::new_constantPool()
        -> constantPoolKlass::allocate()

* JVMTI の RedefineClasses() 及び RetransformClasses() の処理時

  VM_RedefineClasses::doit_prologue() @ hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp
  -> VM_RedefineClasses::load_new_class_versions() @ hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp
     -> VM_RedefineClasses::merge_cp_and_rewrite()
        -> oopFactory::new_constantPool()
           -> (同上)
        -> VM_RedefineClasses::set_new_constant_pool()
           -> oopFactory::new_constantPool()
              -> (同上)

* MethodHandle に対応する methodOop の生成処理時

  MethodHandleCompiler::compile()
  -> MethodHandleWalker::walk()
     -> MethodHandleCompiler::make_invoke()
        -> methodOopDesc::make_invoke_method()
           -> oopFactory::new_constantPool()
              -> (同上)
  -> MethodHandleCompiler::get_method_oop()
     -> MethodHandleCompiler::get_constant_pool()
        -> oopFactory::new_constantPool()
           -> (同上)

* ?? (#TODO)

  MethodHandleWalker::retype_raw_conversion()
  -> MethodHandleCompiler::make_invoke()
     -> (同上)

  SystemDictionary::find_method_handle_invoke()
  -> methodOopDesc::make_invoke_method()
     -> oopFactory::new_constantPool()
        -> (同上)
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * typeArrayOop         _tags

    この constantPoolOopDesc オブジェクト内に格納されている要素の constant tag 情報だけを集めた配列.
    (constant tag とは, constant pool 内の要素の種別を表すタグのこと.
     (See: constantTag))

  * constantPoolCacheOop _cache

    この constantPoolOopDesc に対応する constantPoolCacheOopDesc オブジェクト.

  * klassOop             _pool_holder

    この constantPoolOopDesc に対応する KlassOopDesc オブジェクト.

  * typeArrayOop         _operands
    
    

  * int                  _flags
    
    

  * int                  _length

    実際の Constant Pool 情報部分(後述)の大きさ
    (なお, これはクラスファイル中に記載されている情報(constant_pool_count))

  * volatile bool        _is_conc_safe
    
    

  * int                  _orig_length
    
    

また, これ以降のフィールド宣言されていない領域に実際の Constant Pool 情報が格納されている
(この部分の大きさは _length フィールドで把握可能).


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
      typeArrayOop         _tags; // the tag array describing the constant pool's contents
      constantPoolCacheOop _cache;         // the cache holding interpreter runtime information
      klassOop             _pool_holder;   // the corresponding class
      typeArrayOop         _operands;      // for variable-sized (InvokeDynamic) nodes, usually empty
      int                  _flags;         // a few header bits to describe contents for GC
      int                  _length; // number of elements in the array
      volatile bool        _is_conc_safe; // if true, safe for concurrent
                                          // GC processing
      // only set to non-zero if constant pool is merged by RedefineClasses
      int                  _orig_length;
```

### 備考(Notes)
なお, 実際の使用箇所では constantPoolOop という別名(もしくはラッパークラス)で使われることが多い (See: constantPoolOop).




### 詳細(Details)
See: [here](../doxygen/classconstantPoolOopDesc.html) for details

---
## <a name="noP1NB7bgV" id="noP1NB7bgV">CPSlot</a>

### 概要(Summary)
constantPoolOopDesc 内で使用される補助クラス.

constantPoolOopDesc 内のデータを扱う際に使用される一時オブジェクト(ValueObjクラス).
Symbol* と「最下位1bitが立ったポインタ」の変換, 及び oop とポインタの変換を行う.

(constantPoolOopDesc 内の CONSTANT_Class_info 情報と CONSTANT_String_info 情報には, 
最初に使用されるまでは Symbol オブジェクトへのポインタが格納され, その後は klassOop や文字列オブジェクトが格納される.
その際, Symbol オブジェクトへのポインタの方については最下位1bitを立てておくことで, 
どちらが格納されているかを区別できるようにしている (See: [here](no2935KOa.html) for details))


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
    class CPSlot VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
* Symbol* や oop から (最下位1bitが立ったポインタ, もしくはポインタそのものへ) 変換するには?

  変換したい値をコンストラクタ引数として CPSlot オブジェクトを生成し, 
  CPSlot::value() で取り出せばいい.

* (最下位1bitが立ったポインタ, もしくはポインタそのものから) Symbol* や oop へ変換するには?

  変換したい値をコンストラクタ引数として CPSlot オブジェクトを生成し, 
  CPSlot::get_oop() もしくは CPSlot::get_symbol() で取り出せばいい.

#### 使用箇所(where its instances are used)
以下の箇所で使用されている. (#TODO 他にもあるか??)

* constantPoolOopDesc 内のデータを oop や Symbol* として取り出す作業

  constantPoolOopDesc::slot_at() 経由で取り出される他, 
  変換処理のためだけに CPSlot オブジェクトをその場で生成している箇所もある
  (See: constantPoolOopDesc::resolved_klass_at(), constantPoolOopDesc::unresolved_klass_at(),
   constantPoolOopDesc::resolved_string_at(), constantPoolOopDesc::unresolved_string_at()).

* Symbol* を constantPoolOopDesc 内に格納する作業

  Symbol* を格納する作業では constantPoolOopDesc::slot_at_put() が使用されている.
  (constantPoolOopDesc::slot_at_put() は引数の型が CPSlot となっているため, 
  Symbol* から CPSlot へ暗黙的に型変換される)
  (See: constantPoolOopDesc::slot_at_put())

### 内部構造(Internal structure)
定義されているフィールドは以下の 1つのみ.


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
      intptr_t _ptr;
```

(Symbol* の場合にのみ, コンストラクタ内で最下位ビットを立てている)


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
      CPSlot(intptr_t ptr): _ptr(ptr) {}
      CPSlot(void* ptr): _ptr((intptr_t)ptr) {}
      CPSlot(oop ptr): _ptr((intptr_t)ptr) {}
      CPSlot(Symbol* ptr): _ptr((intptr_t)ptr | 1) {}
    
      intptr_t value()   { return _ptr; }
      bool is_oop()      { return (_ptr & 1) == 0; }
      bool is_metadata() { return (_ptr & 1) == 1; }
```

(逆に CPSlot::get_symbol() で Symbol* を取り出す際には, 最下位ビットをクリアしている)


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
      oop get_oop() {
        assert(is_oop(), "bad call");
        return oop(_ptr);
      }
      Symbol* get_symbol() {
        assert(is_metadata(), "bad call");
        return (Symbol*)(_ptr & ~1);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCPSlot.html) for details

---
## <a name="no2AXVDs-h" id="no2AXVDs-h">SymbolHashMap</a>

### 概要(Summary)
JvmtiConstantPoolReconstituter (及びそのサブクラスである JvmtiClassFileReconstituter) 内で使用される補助クラス.

Symbol を key, 任意の u2 値を value とするハッシュ
(現状では, 
constantPoolOopDesc 内の CONSTANT_Class_info 情報や CONSTANT_String_info 情報に対して, 
対応する Symbol オブジェクトから constantPoolOopDesc 内での index 番号への写像を作るために使用されている模様).


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
    class SymbolHashMap: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiConstantPoolReconstituter オブジェクトの _symmap フィールド, 及び _classmap フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
JvmtiConstantPoolReconstituter::JvmtiConstantPoolReconstituter() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classSymbolHashMap.html) for details

---
## <a name="noQrjEtvyC" id="noQrjEtvyC">SymbolHashMapEntry</a>

### 概要(Summary)
SymbolHashMap 内で使用される補助クラス.

SymbolHashMap オブジェクト内に格納されるハッシュテーブル・エントリ.


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
    class SymbolHashMapEntry : public CHeapObj {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ)


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
      unsigned int        _hash;   // 32-bit hash for item
      SymbolHashMapEntry* _next;   // Next element in the linked list for this bucket
      Symbol*             _symbol; // 1-st part of the mapping: symbol => value
      u2                  _value;  // 2-nd part of the mapping: symbol => value
```




### 詳細(Details)
See: [here](../doxygen/classSymbolHashMapEntry.html) for details

---
## <a name="no7Ma4uE1H" id="no7Ma4uE1H">SymbolHashMapBucket</a>

### 概要(Summary)
SymbolHashMap 内で使用される補助クラス.

SymbolHashMapEntry オブジェクトを格納するための線形リスト
(SymbolHashMap はこのクラスを用いてハッシュテーブルを構築する).


```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
    class SymbolHashMapBucket : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SymbolHashMap オブジェクトの _buckets フィールドに(のみ)格納されている
(正確には, このフィールドは SymbolHashMapBucket の配列を格納するフィールド).

### 内部構造(Internal structure)
以下の _entry フィールドに SymbolHashMapEntry オブジェクトを線形リスト状に格納している.

```
    ((cite: hotspot/src/share/vm/oops/constantPoolOop.hpp))
      SymbolHashMapEntry*    _entry;
```




### 詳細(Details)
See: [here](../doxygen/classSymbolHashMapBucket.html) for details

---
