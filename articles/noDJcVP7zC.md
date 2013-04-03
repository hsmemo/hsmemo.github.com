---
layout: default
title: JNI_FastGetField クラス 
---
[Top](../index.html)

#### JNI_FastGetField クラス 



---
## <a name="noNxyr9N7b" id="noNxyr9N7b">JNI_FastGetField</a>

### 概要(Summary)
JNI の `Get<Primitive>Field()` 関数用のマシン語コードを生成するクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

(See: [here](no5248c5L.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jniFastGetField.hpp))
    // Basic logic of a fast version of jni_Get<Primitive>Field:
    //
    // (See safepoint.hpp for a description of _safepoint_counter)
    //
    // load _safepoint_counter into old_counter
    // IF old_counter is odd THEN
    //   a safepoint is going on, return jni_GetXXXField
    // ELSE
    //   load the primitive field value into result (speculatively)
    //   load _safepoint_counter into new_counter
    //   IF (old_counter == new_counter) THEN
    //     no safepoint happened during the field access, return result
    //   ELSE
    //     a safepoint might have happened in-between, return jni_GetXXXField()
    //   ENDIF
    // ENDIF
    //
    // LoadLoad membars to maintain the load order may be necessary
    // for some platforms.
    //
    // The fast versions don't check for pending suspension request.
    // This is fine since it's totally read-only and doesn't create new race.
    //
    // There is a hypothetical safepoint counter wraparound. But it's not
    // a practical concern.
    
    class JNI_FastGetField : AllStatic {
```

### 内部構造(Internal structure)
内部には, 高速な Get<Primitive>Field() コードを生成するための以下のようなメソッドが定義されている.

```
    ((cite: hotspot/src/share/vm/prims/jniFastGetField.hpp))
      static address generate_fast_get_boolean_field();
      static address generate_fast_get_byte_field();
      static address generate_fast_get_char_field();
      static address generate_fast_get_short_field();
      static address generate_fast_get_int_field();
      static address generate_fast_get_long_field();
      static address generate_fast_get_float_field();
      static address generate_fast_get_double_field();
```




### 詳細(Details)
See: [here](../doxygen/classJNI__FastGetField.html) for details

---
