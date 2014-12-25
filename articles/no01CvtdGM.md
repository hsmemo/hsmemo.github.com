---
layout: default
title: VerificationType クラス 
---
[Top](../index.html)

#### VerificationType クラス 



---
## <a name="noU_rVRh7I" id="noU_rVRh7I">VerificationType</a>

### 概要(Summary)
クラスファイルの verify 処理で使用される補助クラス(ValueObjクラス) (See: [here](no7882amm.html) for details).

JVM 上での「型」を表す.
  

```cpp
    ((cite: hotspot/src/share/vm/classfile/verificationType.hpp))
    class VerificationType VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
ClassVerifier クラス, 及び StackMapTable/StackMapFrame/StackMapReader クラス内で(のみ)使用されている模様 (? #TODO).

### 内部構造(Internal structure)
以下の union 型のフィールド(_u)のみからなる. このフィールドは以下のように使われる.

* (オブジェクトや配列など) reference 型の場合: _sym にその型を表す Symbol を入れる
* primitive 型の場合: _data に型を表す定数値を入れる

reference か primitive かは最下位1bitの0/1で判断できるようにしている模様 (Symbol* なら 0).


```cpp
    ((cite: hotspot/src/share/vm/classfile/verificationType.hpp))
        // Least significant bits of _handle are always 0, so we use these as
        // the indicator that the _handle is valid.  Otherwise, the _data field
        // contains encoded data (as specified below).  Should the VM change
        // and the lower bits on oops aren't 0, the assert in the constructor
        // will catch this and we'll have to add a descriminator tag to this
        // structure.
        union {
          Symbol*   _sym;
          uintptr_t _data;
        } _u;
```

(なお, primitive 型の場合に _data フィールドに格納される定数値は, 以下のように定義されている)


```cpp
    ((cite: hotspot/src/share/vm/classfile/verificationType.hpp))
    enum {
      // As specifed in the JVM spec
      ITEM_Top = 0,
      ITEM_Integer = 1,
      ITEM_Float = 2,
      ITEM_Double = 3,
      ITEM_Long = 4,
      ITEM_Null = 5,
      ITEM_UninitializedThis = 6,
      ITEM_Object = 7,
      ITEM_Uninitialized = 8,
      ITEM_Bogus = (uint)-1
    };
```


```cpp
    ((cite: hotspot/src/share/vm/classfile/verificationType.hpp))
        enum {
          // These rest are not found in classfiles, but used by the verifier
          ITEM_Boolean = 9, ITEM_Byte, ITEM_Short, ITEM_Char,
          ITEM_Long_2nd, ITEM_Double_2nd
        };
    
        // Enum for the _data field
        enum {
          // Bottom two bits determine if the type is a reference, primitive,
          // uninitialized or a query-type.
          TypeMask           = 0x00000003,
    
          // Topmost types encoding
          Reference          = 0x0,        // _sym contains the name
          Primitive          = 0x1,        // see below for primitive list
          Uninitialized      = 0x2,        // 0x00ffff00 contains bci
          TypeQuery          = 0x3,        // Meta-types used for category testing
    
          // Utility flags
          ReferenceFlag      = 0x00,       // For reference query types
          Category1Flag      = 0x01,       // One-word values
          Category2Flag      = 0x02,       // First word of a two-word value
          Category2_2ndFlag  = 0x04,       // Second word of a two-word value
    
          // special reference values
          Null               = 0x00000000, // A reference with a 0 sym is null
    
          // Primitives categories (the second byte determines the category)
          Category1          = (Category1Flag     << 1 * BitsPerByte) | Primitive,
          Category2          = (Category2Flag     << 1 * BitsPerByte) | Primitive,
          Category2_2nd      = (Category2_2ndFlag << 1 * BitsPerByte) | Primitive,
    
          // Primitive values (type descriminator stored in most-signifcant bytes)
          Bogus              = (ITEM_Bogus      << 2 * BitsPerByte) | Category1,
          Boolean            = (ITEM_Boolean    << 2 * BitsPerByte) | Category1,
          Byte               = (ITEM_Byte       << 2 * BitsPerByte) | Category1,
          Short              = (ITEM_Short      << 2 * BitsPerByte) | Category1,
          Char               = (ITEM_Char       << 2 * BitsPerByte) | Category1,
          Integer            = (ITEM_Integer    << 2 * BitsPerByte) | Category1,
          Float              = (ITEM_Float      << 2 * BitsPerByte) | Category1,
          Long               = (ITEM_Long       << 2 * BitsPerByte) | Category2,
          Double             = (ITEM_Double     << 2 * BitsPerByte) | Category2,
          Long_2nd           = (ITEM_Long_2nd   << 2 * BitsPerByte) | Category2_2nd,
          Double_2nd         = (ITEM_Double_2nd << 2 * BitsPerByte) | Category2_2nd,
    
          // Used by Uninitialized (second and third bytes hold the bci)
          BciMask            = 0xffff << 1 * BitsPerByte,
          BciForThis         = ((u2)-1),   // A bci of -1 is an Unintialized-This
    
          // Query values
          ReferenceQuery     = (ReferenceFlag     << 1 * BitsPerByte) | TypeQuery,
          Category1Query     = (Category1Flag     << 1 * BitsPerByte) | TypeQuery,
          Category2Query     = (Category2Flag     << 1 * BitsPerByte) | TypeQuery,
          Category2_2ndQuery = (Category2_2ndFlag << 1 * BitsPerByte) | TypeQuery
        };
```




### 詳細(Details)
See: [here](../doxygen/classVerificationType.html) for details

---
