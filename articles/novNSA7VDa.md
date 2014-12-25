---
layout: default
title: Location クラス 
---
[Top](../index.html)

#### Location クラス 



---
## <a name="no6CKP5Akh" id="no6CKP5Akh">Location</a>

### 概要(Summary)
DebugInformationRecorder に格納されるデバッグ情報を表すクラス.

どういう型の値がどこに入っているか(レジスタか(in_register), スタック上か(on_stack))、という情報を表す.

なお, 内部的には32bit の値にエンコーディングされる.


```cpp
    ((cite: hotspot/src/share/vm/code/location.hpp))
    // A Location describes a concrete machine variable location
    // (such as integer or floating point register or a stack-held
    // variable). Used when generating debug-information for nmethods.
    //
    // Encoding:
    //
    // bits (use low bits for best compression):
    //  Type:   [3..0]
    //  Where:  [4]
    //  Offset: [31..5]
    
    class Location VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
内部的には juint 型のフィールドを一つだけ持ち(_value),
場所や型情報などを 32 ビット内にエンコードして格納している.


```cpp
    ((cite: hotspot/src/share/vm/code/location.hpp))
      juint _value;
    ...
    
      inline void set(Where where_, Type type_, unsigned offset_) {
        _value = (juint) ((where_  << WHERE_SHIFT) |
                          (type_   << TYPE_SHIFT)  |
                          ((offset_ << OFFSET_SHIFT) & OFFSET_MASK));
      }
```


なお, Where や Type として指定できる値には以下のようなものがある.


```cpp
    ((cite: hotspot/src/share/vm/code/location.hpp))
      enum Where {
        on_stack,
        in_register
      };
    
      enum Type {
        invalid,                    // Invalid location
        normal,                     // Ints, floats, double halves
        oop,                        // Oop (please GC me!)
        int_in_long,                // Integer held in long register
        lng,                        // Long held in one register
        float_in_dbl,               // Float held in double register
        dbl,                        // Double held in one register
        addr,                       // JSR return address
        narrowoop                   // Narrow Oop (please GC me!)
      };
```




### 詳細(Details)
See: [here](../doxygen/classLocation.html) for details

---
