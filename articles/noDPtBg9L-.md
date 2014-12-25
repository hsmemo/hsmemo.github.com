---
layout: default
title: ciType クラス関連のクラス (ciType, ciReturnAddress)
---
[Top](../index.html)

#### ciType クラス関連のクラス (ciType, ciReturnAddress)

これらは, JIT Compiler 内で JVM の「型」を扱うためのユーティリティ・クラス.


### クラス一覧(class list)

  * [ciType](#noHOkPqbeW)
  * [ciReturnAddress](#noTP1YVm6k)


---
## <a name="noHOkPqbeW" id="noHOkPqbeW">ciType</a>

### 概要(Summary)
JIT Compiler 内で JVM の「型」を扱うためのユーティリティ・クラス.
1つの ciType オブジェクトが 1つの型に対応する.

(一応 ciObject クラスの具象サブクラスの1つだが, 
対応する oop を持つわけではないので ciObject という感じがあんまりしないが... #TODO)


```cpp
    ((cite: hotspot/src/share/vm/ci/ciType.hpp))
    // ciType
    //
    // This class represents either a class (T_OBJECT), array (T_ARRAY),
    // or one of the primitive types such as T_INT.
    class ciType : public ciObject {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ciType クラスの _basic_types フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは ciType の配列を格納するフィールド.
この中に, 全ての ciType オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
ciType クラスの _basic_types フィールドは, ポインタ型ではなく実体なので,
配列用のメモリ領域は初期段階で自動的に生成される.

そのメモリ領域中に個別の ciType オブジェクトを書き込む作業は 
ciObjectFactory::init_shared_objects() 内で(のみ)行われている.

### 内部構造(Internal structure)
スーパークラスである ciObject クラスのフィールドに加えて, 以下のフィールドを持つ.

* BasicType 	_basic_type
  
  (その ciType オブジェクトが表している型を示す)


```cpp
    ((cite: hotspot/src/share/vm/ci/ciType.hpp))
      BasicType _basic_type;
```




### 詳細(Details)
See: [here](../doxygen/classciType.html) for details

---
## <a name="noTP1YVm6k" id="noTP1YVm6k">ciReturnAddress</a>

### 概要(Summary)
ciType クラスのサブクラスの1つ.
JIT Compiler 内で returnAddress 型の値を扱うためのユーティリティ・クラス
(なお, ここでの returnAddress 型とは jsr バイトコード/ret バイトコードが使用する returnAddress 型のこと).

(一応 ciObject クラスの具象サブクラスの1つだが, 
対応する oop を持つわけではないので ciObject という感じがあんまりしないが... #TODO)

なお, 一度生成された ciReturnAddress オブジェクトはメモイズされており, 
bci が同じものに対しては同一の ciReturnAddress オブジェクトが返されるようになっている.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciType.hpp))
    // ciReturnAddress
    //
    // This class represents the type of a specific return address in the
    // bytecodes.
    class ciReturnAddress : public ciType {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciObjectFactory オブジェクトの _return_addresses フィールドに(のみ)格納されている.

(正確には, このフィールドは ciReturnAddress の GrowableArray を格納するフィールド.
この中に, その ciObjectFactory の ciObjectFactory::get_return_address() で生成された全ての 
ciReturnAddress オブジェクトが格納されている)

(ただし, ciReturnAddress オブジェクトの生成自体は実際に必要になるまで遅延されている)

#### 生成箇所(where its instances are created)
GrowableArray 用のメモリ領域は ciObjectFactory::ciObjectFactory() 内で(のみ)確保されている. 

ciReturnAddress オブジェクト自体は 
ciObjectFactory::get_return_address() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).

### 内部構造(Internal structure)
スーパークラスである ciType クラスのフィールドに加えて, 以下のフィールドを持つ.

* int _bci
  
  (リターン先のバイトコードを示す (= 要するに対応する jsr バイトコードの次のバイトコード))


```cpp
    ((cite: hotspot/src/share/vm/ci/ciType.hpp))
      // The bci of this return address.
      int _bci;
```

### 備考(Notes)
なお ciType としての型は T_ADDRESS.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciType.cpp))
    ciReturnAddress::ciReturnAddress(int bci) : ciType(T_ADDRESS) {
      assert(0 <= bci, "bci cannot be negative");
      _bci = bci;
    }
```




### 詳細(Details)
See: [here](../doxygen/classciReturnAddress.html) for details

---
