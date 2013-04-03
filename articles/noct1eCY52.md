---
layout: default
title: Type クラス(とそのサブクラス) (Type, TypeF, TypeD, TypeInt, TypeLong, TypeTuple, TypeAry, TypePtr, TypeRawPtr, TypeOopPtr, TypeInstPtr, TypeAryPtr, TypeKlassPtr, TypeNarrowOop, TypeFunc)
---
[Top](../index.html)

#### Type クラス(とそのサブクラス) (Type, TypeF, TypeD, TypeInt, TypeLong, TypeTuple, TypeAry, TypePtr, TypeRawPtr, TypeOopPtr, TypeInstPtr, TypeAryPtr, TypeKlassPtr, TypeNarrowOop, TypeFunc)

これらは, C2 JIT Compiler 用のユーティリティ・クラス.

### 概要(Summary)
これらは, C2 JIT Compiler が扱う「型」の情報を表すクラス.

なお, これらの型は lattice をなす (下記参照).
ただし lattice 構造はクラスの継承関係とは無関係で meet/join 操作にだけ影響する.

また, これらのクラスは単なる型情報ではなく取り得る値の情報も保持している.
この情報は定数伝播や配列要素の依存解析等に使用されている.

(なおコメントでは, 整数型なら RSD(regular section descriptor) の 
 '(lower bound, upper bound, stride)' を持つ, 
 と書かれているが実際の整数型は lower bound と upper bound しか持っていない)


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    // This class defines a Type lattice.  The lattice is used in the constant
    // propagation algorithms, and for some type-checking of the iloc code.
    // Basic types include RSD's (lower bound, upper bound, stride for integers),
    // float & double precision constants, sets of data-labels and code-labels.
    // The complete lattice is described below.  Subtypes have no relationship to
    // up or down in the lattice; that is entirely determined by the behavior of
    // the MEET/JOIN functions.
    
    class Dict;
    class Type;
    class   TypeD;
    class   TypeF;
    class   TypeInt;
    class   TypeLong;
    class   TypeNarrowOop;
    class   TypeAry;
    class   TypeTuple;
    class   TypePtr;
    class     TypeRawPtr;
    class     TypeOopPtr;
    class       TypeInstPtr;
    class       TypeAryPtr;
    class       TypeKlassPtr;
```

(<= 値を持っていることから "sized type" みたいな若干制限された dependent type とみなせるわけで, 
 そこら辺の研究成果を突っ込んでみると面白いことがあるかもしれない(??).
 実際, 現状でも配列の境界違反なんかはコンパイル時にチェックできてる
 (ただしマイナス方向にだけど..)(See: AllocateArrayNode::Ideal()))


### クラス一覧(class list)

  * [Type](#noqfAiE2NL)
  * [TypeF](#nosjLEhrlD)
  * [TypeD](#noqihBNLCy)
  * [TypeInt](#nooZN_40bE)
  * [TypeLong](#nosQfxDpg5)
  * [TypeTuple](#noSYH_qwSj)
  * [TypeAry](#no_3ZaqJR5)
  * [TypePtr](#nolObXGSc7)
  * [TypeRawPtr](#noZasm8yaU)
  * [TypeOopPtr](#noeKd-eej2)
  * [TypeInstPtr](#nopzYcezT9)
  * [TypeAryPtr](#no6fx0T9A2)
  * [TypeKlassPtr](#nohFLUe4lE)
  * [TypeNarrowOop](#no9IPB1PT0)
  * [TypeFunc](#no2-kTL9Iu)


---
## <a name="noqfAiE2NL" id="noqfAiE2NL">Type</a>

### 概要(Summary)
C2 JIT Compiler 内で扱う「型」情報を表すクラス.

コメントによると, 
「Type オブジェクトは immutable.
 また hash consing しているため同じ内容の Type オブジェクトは一つしか存在しない」
とのこと.

なお, このクラスは abstract class ではない.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------Type-------------------------------------------
    // Basic Type object, represents a set of primitive Values.
    // Types are hash-cons'd into a private class dictionary, so only one of each
    // different kind of Type exists.  Types are never modified after creation, so
    // all their interesting fields are constant.
    class Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Type::make()
* Type::xdual()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* TYPES _base
  
  この Type オブジェクトが表す「型」を示す.
  なお TYPES 型は以下のように定義された enum 値.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      enum TYPES {
        Bad=0,                      // Type check
        Control,                    // Control of code (not in lattice)
        Top,                        // Top of the lattice
        Int,                        // Integer range (lo-hi)
        Long,                       // Long integer range (lo-hi)
        Half,                       // Placeholder half of doubleword
        NarrowOop,                  // Compressed oop pointer
    
        Tuple,                      // Method signature or object layout
        Array,                      // Array types
    
        AnyPtr,                     // Any old raw, klass, inst, or array pointer
        RawPtr,                     // Raw (non-oop) pointers
        OopPtr,                     // Any and all Java heap entities
        InstPtr,                    // Instance pointers (non-array objects)
        AryPtr,                     // Array pointers
        KlassPtr,                   // Klass pointers
        // (Ptr order matters:  See is_ptr, isa_ptr, is_oopptr, isa_oopptr.)
    
        Function,                   // Function signature
        Abio,                       // Abstract I/O
        Return_Address,             // Subroutine return address
        Memory,                     // Abstract store
        FloatTop,                   // No float value
        FloatCon,                   // Floating point constant
        FloatBot,                   // Any float value
        DoubleTop,                  // No double value
        DoubleCon,                  // Double precision constant
        DoubleBot,                  // Any double value
        Bottom,                     // Bottom of lattice
        lastype                     // Bogus ending type (not in lattice)
      };
```

* Type *_dual
  
  lattice 上でこの Type オブジェクトの dual になる Type を指す.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      // Each class of type is also identified by its base.
      const TYPES _base;            // Enum of Types type
```


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      // DUAL operation: reflect around lattice centerline.  Used instead of
      // join to ensure my lattice is symmetric up and down.  Dual is computed
      // lazily, on demand, and cached in _dual.
      const Type *_dual;            // Cached dual value
```

### 備考(Notes)
operator new がオーバーライドされており, 
このクラスのオブジェクトは Compile::_type_arena フィールドの Arena 内に確保される.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      inline void* operator new( size_t x ) {
        Compile* compile = Compile::current();
        compile->set_type_last_size(x);
        void *temp = compile->type_arena()->Amalloc_D(x);
        compile->set_type_hwm(temp);
        return temp;
      }
      inline void operator delete( void* ptr ) {
        Compile* compile = Compile::current();
        compile->type_arena()->Afree(ptr,compile->type_last_size());
      }
```




### 詳細(Details)
See: [here](../doxygen/classType.html) for details

---
## <a name="nosjLEhrlD" id="nosjLEhrlD">TypeF</a>

### 概要(Summary)
Type クラスのサブクラスの1つ.
このクラスは float の定数値を表す型
(なお, 単なる float 型ではなく JIT コンパイル時に値が一意に定まっているもの).


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeF------------------------------------------
    // Class of Float-Constant Types.
    class TypeF : public Type {
```

<!-- (...最後にFが付くと中ニ病的な格好よさがあるな. System F とか Macross F とか...) -->

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
TypeF::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const float _f;               // Float constant
```

なお, _base は FloatCon.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeF( float f ) : Type(FloatCon), _f(f) {};
```




### 詳細(Details)
See: [here](../doxygen/classTypeF.html) for details

---
## <a name="noqihBNLCy" id="noqihBNLCy">TypeD</a>

### 概要(Summary)
Type クラスのサブクラスの1つ.
このクラスは double の定数値を表す型
(なお, 単なる double 型ではなく JIT コンパイル時に値が一意に定まっているもの).


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeD------------------------------------------
    // Class of Double-Constant Types.
    class TypeD : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
TypeD::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const double _d;              // Double constant
```

なお, _base は DoubleCon.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeD( double d ) : Type(DoubleCon), _d(d) {};
```




### 詳細(Details)
See: [here](../doxygen/classTypeD.html) for details

---
## <a name="nooZN_40bE" id="nooZN_40bE">TypeInt</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスは int 値を表す型. 加えてその値が取り得る範囲(下限/上限)も指定できる.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeInt----------------------------------------
    // Class of integer ranges, the set of integers between a lower bound and an
    // upper bound, inclusive.
    class TypeInt : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeInt::make( jint lo )
* TypeInt::make( jint lo, jint hi, int w )
* TypeInt::xdual()

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.

(なお _widen は定数伝播時の widen の残り回数を示す (See: TypeInt::widen()))


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const jint _lo, _hi;          // Lower bound, upper bound
      const short _widen;           // Limit on times we widen this sucker
```

なお, _base は Int.


```
    ((cite: hotspot/src/share/vm/opto/type.cpp))
    TypeInt::TypeInt( jint lo, jint hi, int w ) : Type(Int), _lo(lo), _hi(hi), _widen(w) {
    }
```




### 詳細(Details)
See: [here](../doxygen/classTypeInt.html) for details

---
## <a name="nosQfxDpg5" id="nosQfxDpg5">TypeLong</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスは long 値を表す型. 加えてその値が取り得る範囲(下限/上限)も指定できる.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeLong---------------------------------------
    // Class of long integer ranges, the set of integers between a lower bound and
    // an upper bound, inclusive.
    class TypeLong : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeLong::make( jlong lo )
* TypeLong::make( jlong lo, jlong hi, int w )
* TypeLong::xdual()

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.

(なお _widen は定数伝播時の widen の残り回数を示す (See: TypeLong::widen()))


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const jlong _lo, _hi;         // Lower bound, upper bound
      const short _widen;           // Limit on times we widen this sucker
```

なお, _base は Long.


```
    ((cite: hotspot/src/share/vm/opto/type.cpp))
    TypeLong::TypeLong( jlong lo, jlong hi, int w ) : Type(Long), _lo(lo), _hi(hi), _widen(w) {
    }
```




### 詳細(Details)
See: [here](../doxygen/classTypeLong.html) for details

---
## <a name="noSYH_qwSj" id="noSYH_qwSj">TypeTuple</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスは複数の値からなる組(tuple)を表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeTuple--------------------------------------
    // Class of Tuple Types, essentially type collections for function signatures
    // and class layouts.  It happens to also be a fast cache for the HotSpot
    // signature types.
    class TypeTuple : public Type {
```

なお, 現状では MultiNode (及びそのサブクラス) からしか生成されない模様(? #TODO).


```
    ((cite: hotspot/src/share/vm/opto/multnode.hpp))
    //------------------------------ProjNode---------------------------------------
    // This class defines a Projection node.  Projections project a single element
    // out of a tuple (or Signature) type.  Only MultiNodes produce TypeTuple
    // results.
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeTuple::make_range()
* TypeTuple::make_domain()
* TypeTuple::make()
* TypeTuple::xdual()

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const uint          _cnt;              // Count of fields
      const Type ** const _fields;           // Array of field types
```

なお, _base は Tuple.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeTuple( uint cnt, const Type **fields ) : Type(Tuple), _cnt(cnt), _fields(fields) { }
```




### 詳細(Details)
See: [here](../doxygen/classTypeTuple.html) for details

---
## <a name="no_3ZaqJR5" id="no_3ZaqJR5">TypeAry</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスは配列を表す型.

(なお, 「配列を指すポインタ」の型は TypeAryPtr が表す. このクラスは配列自体の情報(中身の型,長さ)を表す).


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeAry----------------------------------------
    // Class of Array Types
    class TypeAry : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeAry::make()
* TypeAry::xdual()

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const Type *_elem;            // Element type of array
      const TypeInt *_size;         // Elements in array
```

なお, _base は Array.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeAry( const Type *elem, const TypeInt *size) : Type(Array),
        _elem(elem), _size(size) {}
```




### 詳細(Details)
See: [here](../doxygen/classTypeAry.html) for details

---
## <a name="nolObXGSc7" id="nolObXGSc7">TypePtr</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスは何らかのポインタを表す型.

(具体的なポインタの差し先としては raw data, instances, arrays, 等があり得る.
指し先の種別が確定している場合は TypePtr クラスのサブクラスが利用できる.
そうでない場合 (= 何を指すかが確定しない場合) は TypePtr クラスが利用される)


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypePtr----------------------------------------
    // Class of machine Pointer Types: raw data, instances or arrays.
    // If the _base enum is AnyPtr, then this refers to all of the above.
    // Otherwise the _base will indicate which subset of pointers is affected,
    // and the class will be inherited from.
    class TypePtr : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypePtr::make()
* TypePtr::xdual()

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const int _offset;            // Offset into oop, with TOP & BOT
      const PTR _ptr;               // Pointer equivalence class
```

(なお, PTR 型は以下のように定義された enum 値)


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      enum PTR { TopPTR, AnyNull, Constant, Null, NotNull, BotPTR, lastPTR };
```

なお, _base は AnyPtr
(正確に言うと, _base はコンストラクタで指定された値になる. 
ただし TypePtr クラスの場合は現状では AnyPtr しか指定されていない. AnyPtr 以外が指定されるのは TypePtr のサブクラスの場合).


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypePtr( TYPES t, PTR ptr, int offset ) : Type(t), _ptr(ptr), _offset(offset) {}
```




### 詳細(Details)
See: [here](../doxygen/classTypePtr.html) for details

---
## <a name="noZasm8yaU" id="noZasm8yaU">TypeRawPtr</a>

### 概要(Summary)
TypePtr クラスのサブクラスの1つ. 
このクラスは oop 以外を指すポインタを表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeRawPtr-------------------------------------
    // Class of raw pointers, pointers to things other than Oops.  Examples
    // include the stack pointer, top of heap, card-marking area, handles, etc.
    class TypeRawPtr : public TypePtr {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeRawPtr::make( enum PTR ptr )
* TypeRawPtr::make( address bits )
* TypeRawPtr::xdual()

### 内部構造(Internal structure)
スーパークラスである TypePtr クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const address _bits;          // Constant value, if applicable
```

なお, _base は RawPtr.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeRawPtr( PTR ptr, address bits ) : TypePtr(RawPtr,ptr,0), _bits(bits){}
```




### 詳細(Details)
See: [here](../doxygen/classTypeRawPtr.html) for details

---
## <a name="noeKd-eej2" id="noeKd-eej2">TypeOopPtr</a>

### 概要(Summary)
TypePtr クラスのサブクラスの1つ. 
このクラスは何らかの oop を指すポインタを表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeOopPtr-------------------------------------
    // Some kind of oop (Java pointer), either klass or instance or array.
    class TypeOopPtr : public TypePtr {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeOopPtr::make()
* TypeOopPtr::xdual()

### 内部構造(Internal structure)
スーパークラスである TypePtr クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      // Oop is NULL, unless this is a constant oop.
      ciObject*     _const_oop;   // Constant oop
      // If _klass is NULL, then so is _sig.  This is an unloaded klass.
      ciKlass*      _klass;       // Klass object
      // Does the type exclude subclasses of the klass?  (Inexact == polymorphic.)
      bool          _klass_is_exact;
      bool          _is_ptr_to_narrowoop;
    
      // If not InstanceTop or InstanceBot, indicates that this is
      // a particular instance of this type which is distinct.
      // This is the the node index of the allocation node creating this instance.
      int           _instance_id;
```

(なお, InstanceTop や InstanceBot は以下のように定義された enum 値)


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      enum {
       InstanceTop = -1,   // undefined instance
       InstanceBot = 0     // any possible instance
      };
```

なお, _base はコンストラクタで指定された値になる.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeOopPtr( TYPES t, PTR ptr, ciKlass* k, bool xk, ciObject* o, int offset, int instance_id );
```




### 詳細(Details)
See: [here](../doxygen/classTypeOopPtr.html) for details

---
## <a name="nopzYcezT9" id="nopzYcezT9">TypeInstPtr</a>

### 概要(Summary)
TypeOopPtr クラスのサブクラスの1つ. 
このクラスは配列ではない Java のインスタンスオブジェクトもしくは klassOopを指すポインタを表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeInstPtr------------------------------------
    // Class of Java object pointers, pointing either to non-array Java instances
    // or to a klassOop (including array klasses).
    class TypeInstPtr : public TypeOopPtr {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeInstPtr::make()
* TypeInstPtr::xdual()

### 内部構造(Internal structure)
スーパークラスである TypeOopPtr クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      ciSymbol*  _name;        // class name
```

なお, _base は InstPtr.


```
    ((cite: hotspot/src/share/vm/opto/type.cpp))
    TypeInstPtr::TypeInstPtr(PTR ptr, ciKlass* k, bool xk, ciObject* o, int off, int instance_id)
     : TypeOopPtr(InstPtr, ptr, k, xk, o, off, instance_id), _name(k->name()) {
       assert(k != NULL &&
              (k->is_loaded() || o == NULL),
              "cannot have constants with non-loaded klass");
    };
```




### 詳細(Details)
See: [here](../doxygen/classTypeInstPtr.html) for details

---
## <a name="no6fx0T9A2" id="no6fx0T9A2">TypeAryPtr</a>

### 概要(Summary)
TypeOopPtr クラスのサブクラスの1つ. 
このクラスは配列オブジェクトを指すポインタを表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeAryPtr-------------------------------------
    // Class of Java array pointers
    class TypeAryPtr : public TypeOopPtr {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeAryPtr::make( PTR ptr, const TypeAry *ary, ciKlass* k, bool xk, int offset, int instance_id )
* TypeAryPtr::make( PTR ptr, ciObject* o, const TypeAry *ary, ciKlass* k, bool xk, int offset, int instance_id )
* TypeAryPtr::xdual()

### 内部構造(Internal structure)
スーパークラスである TypeOopPtr クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const TypeAry *_ary;          // Array we point into
```

なお, _base は AryPtr.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeAryPtr( PTR ptr, ciObject* o, const TypeAry *ary, ciKlass* k, bool xk, int offset, int instance_id ) : TypeOopPtr(AryPtr,ptr,k,xk,o,offset, instance_id), _ary(ary) {
    #ifdef ASSERT
        if (k != NULL) {
          // Verify that specified klass and TypeAryPtr::klass() follow the same rules.
          ciKlass* ck = compute_klass(true);
          if (k != ck) {
            this->dump(); tty->cr();
            tty->print(" k: ");
            k->print(); tty->cr();
            tty->print("ck: ");
            if (ck != NULL) ck->print();
            else tty->print("<NULL>");
            tty->cr();
            assert(false, "unexpected TypeAryPtr::_klass");
          }
        }
    #endif
      }
```




### 詳細(Details)
See: [here](../doxygen/classTypeAryPtr.html) for details

---
## <a name="nohFLUe4lE" id="nohFLUe4lE">TypeKlassPtr</a>

### 概要(Summary)
TypeOopPtr クラスのサブクラスの1つ. 
このクラスは klassOop を指すポインタを表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeKlassPtr-----------------------------------
    // Class of Java Klass pointers
    class TypeKlassPtr : public TypeOopPtr {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeKlassPtr::make()
* TypeKlassPtr::xdual()

### 内部構造(Internal structure)
このクラスで新たに定義されたフィールドはない.

なお, _base は KlassPtr.


```
    ((cite: hotspot/src/share/vm/opto/type.cpp))
    TypeKlassPtr::TypeKlassPtr( PTR ptr, ciKlass* klass, int offset )
      : TypeOopPtr(KlassPtr, ptr, klass, (ptr==Constant), (ptr==Constant ? klass : NULL), offset, 0) {
    }
```




### 詳細(Details)
See: [here](../doxygen/classTypeKlassPtr.html) for details

---
## <a name="no9IPB1PT0" id="no9IPB1PT0">TypeNarrowOop</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスは narrow oop 値を表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeNarrowOop----------------------------------
    // A compressed reference to some kind of Oop.  This type wraps around
    // a preexisting TypeOopPtr and forwards most of it's operations to
    // the underlying type.  It's only real purpose is to track the
    // oopness of the compressed oop value when we expose the conversion
    // between the normal and the compressed form.
    class TypeNarrowOop : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* TypeNarrowOop::make()
* TypeNarrowOop::xdual()

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const TypePtr* _ptrtype; // Could be TypePtr::NULL_PTR
```

なお, _base は NarrowOop.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeNarrowOop( const TypePtr* ptrtype): Type(NarrowOop),
        _ptrtype(ptrtype) {
        assert(ptrtype->offset() == 0 ||
               ptrtype->offset() == OffsetBot ||
               ptrtype->offset() == OffsetTop, "no real offsets");
      }
```




### 詳細(Details)
See: [here](../doxygen/classTypeNarrowOop.html) for details

---
## <a name="no2-kTL9Iu" id="no2-kTL9Iu">TypeFunc</a>

### 概要(Summary)
Type クラスのサブクラスの1つ. 
このクラスはメソッド(methodOopDesc)を表す型.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
    //------------------------------TypeFunc---------------------------------------
    // Class of Array Types
    class TypeFunc : public Type {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の格納箇所)

* 各 Compile オブジェクトの _type_dict フィールド
  
  (これは hash consing 用の辞書)
  
  (正確には, このフィールドは Dict オブジェクトを格納するフィールド.
  この中に, 生成された全ての Type オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
TypeFunc::make() 内で(のみ)生成されている.

### 内部構造(Internal structure)
スーパークラスである Type クラスのフィールドに加えて, 以下のフィールドを持つ.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      const TypeTuple* const _domain;     // Domain of inputs
      const TypeTuple* const _range;      // Range of results
```

なお, _base は Function.


```
    ((cite: hotspot/src/share/vm/opto/type.hpp))
      TypeFunc( const TypeTuple *domain, const TypeTuple *range ) : Type(Function),  _domain(domain), _range(range) {}
```




### 詳細(Details)
See: [here](../doxygen/classTypeFunc.html) for details

---
