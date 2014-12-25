---
layout: default
title: FieldType クラス関連のクラス (FieldArrayInfo, FieldType)
---
[Top](../index.html)

#### FieldType クラス関連のクラス (FieldArrayInfo, FieldType)

これらは, Java の型を表す文字列(Signature String)を扱うためのユーティリティ・クラス.

(なお Signature String とは "[Lfoo;D)I" みたいな文字列のこと)


### クラス一覧(class list)

  * [FieldType](#noOT8w6-c4)
  * [FieldArrayInfo](#noMLJwYHL2)


---
## <a name="noOT8w6-c4" id="noOT8w6-c4">FieldType</a>

### 概要(Summary)
Java の型を表す文字列(Signature String)を扱うためのユーティリティ・クラス
(より正確に言うと, Signature String をパースしてフィールドの型情報を返す関数を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldType.hpp))
    // A FieldType is used to determine the type of a field from a signature string.
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldType.hpp))
    class FieldType: public AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

(ciField や ciObjectFactory, SystemDictionary, constantPoolOopDesc, fieldDescriptor, などなど)

### 内部構造(Internal structure)
定義されている public メソッドは, 以下の通り.

(Signature String を表す Symbol オブジェクトを受け取って何らかの情報を返すメソッドばかり)


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldType.hpp))
      // Return basic type
      static BasicType basic_type(Symbol* signature);
    
      // Testing
      static bool is_array(Symbol* signature) { return signature->utf8_length() > 1 && signature->byte_at(0) == '[' && is_valid_array_signature(signature); }
    
      static bool is_obj(Symbol* signature) {
         int sig_length = signature->utf8_length();
         // Must start with 'L' and end with ';'
         return (sig_length >= 2 &&
                 (signature->byte_at(0) == 'L') &&
                 (signature->byte_at(sig_length - 1) == ';'));
      }
    
      // Parse field and extract array information. Works for T_ARRAY only.
      static BasicType get_array_info(Symbol* signature, FieldArrayInfo& ai, TRAPS);
```

### 備考(Notes)
コメントによると, 
  FieldType は SignatureIterator のサブクラスにした方がいい. あるいは逆でもいい.
  どちらにせよ, FieldType というデータ構造についてはいつかの時点で考え直す必要がある, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldType.hpp))
    // Note: FieldType should be based on the SignatureIterator (or vice versa).
    //       In any case, this structure should be re-thought at some point.
```




### 詳細(Details)
See: [here](../doxygen/classFieldType.html) for details

---
## <a name="noMLJwYHL2" id="noMLJwYHL2">FieldArrayInfo</a>

### 概要(Summary)
FieldType クラス用の補助クラス
(より正確に言うと FieldType::get_array_info() メソッド用の補助クラス).

配列型を表す Signature String から型情報を取得する作業中に使われる一時オブジェクト(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldType.hpp))
    // Information returned by get_array_info, which is scoped to decrement
    // reference count if a Symbol is created in the case of T_OBJECT
    class FieldArrayInfo : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. FieldArrayInfo 型の局所変数を宣言する.
   
2. その FieldArrayInfo オブジェクトを FieldType::get_array_info() で初期化する 
   (指定した Signature String の情報が FieldArrayInfo オブジェクト内に格納される).
    
3. FieldArrayInfo クラスの各メソッドで配列の型情報にアクセスできる

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ciObjectFactory::get_unloaded_klass()
* SystemDictionary::resolve_array_class_or_null()
* SystemDictionary::find_instance_or_array_klass()
* SystemDictionary::find_constrained_instance_or_array_klass()
* SystemDictionary::add_loader_constraint()

### 内部構造(Internal structure)
定義されているフィールドはこれだけ
(配列の次元数, 及び配列の要素の型).


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldType.hpp))
      int       _dimension;
      Symbol*   _object_key;
```




### 詳細(Details)
See: [here](../doxygen/classFieldArrayInfo.html) for details

---
