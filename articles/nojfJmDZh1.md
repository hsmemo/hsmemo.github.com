---
layout: default
title: fieldDescriptor クラス 
---
[Top](../index.html)

#### fieldDescriptor クラス 



---
## <a name="noLOIua6lp" id="noLOIua6lp">fieldDescriptor</a>

### 概要(Summary)
Java のクラスのフィールド情報にアクセスするためのユーティリティ・クラス.

1つの fieldDescriptor オブジェクトが 1つのフィールドに対応する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldDescriptor.hpp))
    // A fieldDescriptor describes the attributes of a single field (instance or class variable).
    // It needs the class constant pool to work (because it only holds indices into the pool
    // rather than the actual info).
    
    class fieldDescriptor VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. fieldDescriptor 型の局所変数を宣言する.
   
2. その fieldDescriptor オブジェクトを以下の関数で初期化する 
   (指定したフィールドの情報が fieldDescriptor オブジェクト内に格納される).
    
   * fieldDescriptor::initialize()
   * instanceKlass::find_field()
   * instanceKlass::find_local_field()
   * instanceKlass::find_field_from_offset()
   * instanceKlass::find_local_field_from_offset()
   * Reflection::resolve_field()
   * JvmtiEnv::get_field_descriptor()
   
3. fieldDescriptor クラスの各メソッドでフィールドの情報にアクセスできる

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

(_access_flags, _offset, _index 以外のものは, 情報そのものではなく Constant Pool 内を指す index)


```cpp
    ((cite: hotspot/src/share/vm/runtime/fieldDescriptor.hpp))
      AccessFlags         _access_flags;
      int                 _name_index;
      int                 _signature_index;
      int                 _initial_value_index;
      int                 _offset;
      int                 _generic_signature_index;
      int                 _index; // index into fields() array
      constantPoolHandle  _cp;
```




### 詳細(Details)
See: [here](../doxygen/classfieldDescriptor.html) for details

---
