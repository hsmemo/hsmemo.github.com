---
layout: default
title: NativeLookup クラス 
---
[Top](../index.html)

#### NativeLookup クラス 



---
## <a name="nonSM9tSBB" id="nonSM9tSBB">NativeLookup</a>

### 概要(Summary)
JNI 機能(より具体的に言うと, ネイティブメソッドに対するダイナミックリンク機能)を実現するためのクラス.

(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```
    ((cite: hotspot/src/share/vm/prims/nativeLookup.hpp))
    // NativeLookup provides an interface for finding DLL entry points for
    // Java native functions.
    
    class NativeLookup : AllStatic {
```

(なおダイナミックリンク以外に,
 1.3.1 以前のバージョンでのみ使用される JVM_SetPrimitiveFieldValues() と JVM_GetPrimitiveFieldValues()
 用の補助関数をバインドする処理も行っている.
 ただし, JDK 1.4 以降の場合にはこれらの関数は使われないのでバインド処理も行われない.)

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* CompileBroker::compile_method()

  これは, メソッドの JIT コンパイル処理を行う関数.
  対象のメソッドがネイティブメソッドの場合, NativeLookup::lookup() が呼び出される.
  (See: [here](no3718SNC.html), [here](no293548G.html) and [here](no3059err.html) for details)

* InterpreterRuntime::prepare_native_call()

  これは, ネイティブメソッドが呼び出された場合の処理を行う関数. (See: [here](no3059asZ.html) and [here](no3059err.html) for details)

* initialize_converter_functions()

  (これは 1.3.1 以前のバージョン用の関数.
   HotSpot の起動中に universe_post_init() 内で呼び出され,
   JVM_SetPrimitiveFieldValues()/JVM_GetPrimitiveFieldValues() 内で使用される補助関数をバインドする.)

  (なお, このバインド処理では NativeLookup::base_library_lookup() が使用されているが,
   普通のルックアップ処理(NativeLookup::lookup())と処理自体は同じで, 少し assert 処理が増えているだけ.)

### 備考(Notes)
initialize_converter_functions() の処理は以下のパスで(のみ)行われる.

```
universe_post_init()
-> initialize_converter_functions()
   -> NativeLookup::base_library_lookup()
      -> NativeLookup::lookup()
```

#### 参考(for your information): initialize_converter_functions()
See: [here](no17119For.html) for details
#### 参考(for your information): NativeLookup::base_library_lookup()
See: [here](no17119WSe.html) for details

### 備考(Notes)
initialize_converter_functions() で初期化される関数ポインタ
(= JVM_SetPrimitiveFieldValues()/JVM_GetPrimitiveFieldValues() 内で使用される関数ポインタ) は, 
以下のフィールドに格納される.


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    static IntBitsToFloatFn   int_bits_to_float_fn   = NULL;
    static LongBitsToDoubleFn long_bits_to_double_fn = NULL;
    static FloatToIntBitsFn   float_to_int_bits_fn   = NULL;
    static DoubleToLongBitsFn double_to_long_bits_fn = NULL;
```

それぞれ以下のように使用されている.


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    JVM_ENTRY(void, JVM_SetPrimitiveFieldValues(JNIEnv *env, jclass cb, jobject obj,
                                                jlongArray fieldIDs, jcharArray typecodes, jbyteArray data))
      assert(!JDK_Version::is_gte_jdk14x_version(), "should only be used in 1.3.1 and earlier");
    ...
          case 'F':
            if (fid != NULL) {
              jint ival = ((dbuf->byte_at(off + 0) & 0xFF) << 24)
                        + ((dbuf->byte_at(off + 1) & 0xFF) << 16)
                        + ((dbuf->byte_at(off + 2) & 0xFF) << 8)
                        + ((dbuf->byte_at(off + 3) & 0xFF) << 0);
              jfloat fval = (*int_bits_to_float_fn)(env, NULL, ival);
              o->float_field_put(field_offset, fval);
            }
            off += 4;
            break;
    ...
          case 'D':
            if (fid != NULL) {
              jlong lval = (((jlong) dbuf->byte_at(off + 0) & 0xFF) << 56)
                         + (((jlong) dbuf->byte_at(off + 1) & 0xFF) << 48)
                         + (((jlong) dbuf->byte_at(off + 2) & 0xFF) << 40)
                         + (((jlong) dbuf->byte_at(off + 3) & 0xFF) << 32)
                         + (((jlong) dbuf->byte_at(off + 4) & 0xFF) << 24)
                         + (((jlong) dbuf->byte_at(off + 5) & 0xFF) << 16)
                         + (((jlong) dbuf->byte_at(off + 6) & 0xFF) << 8)
                         + (((jlong) dbuf->byte_at(off + 7) & 0xFF) << 0);
              jdouble dval = (*long_bits_to_double_fn)(env, NULL, lval);
              o->double_field_put(field_offset, dval);
            }
            off += 8;
            break;
    ...
        }
      }
    JVM_END
```


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    JVM_ENTRY(void, JVM_GetPrimitiveFieldValues(JNIEnv *env, jclass cb, jobject obj,
                                jlongArray fieldIDs, jcharArray typecodes, jbyteArray data))
      assert(!JDK_Version::is_gte_jdk14x_version(), "should only be used in 1.3.1 and earlier");
    ...
           case 'F':
             {
               jfloat fval = o->float_field(field_offset);
               jint ival = (*float_to_int_bits_fn)(env, NULL, fval);
               dbuf->byte_at_put(off++, (ival >> 24) & 0xFF);
               dbuf->byte_at_put(off++, (ival >> 16) & 0xFF);
               dbuf->byte_at_put(off++, (ival >> 8)  & 0xFF);
               dbuf->byte_at_put(off++, (ival >> 0)  & 0xFF);
             }
             break;
    ...
           case 'D':
             {
               jdouble dval = o->double_field(field_offset);
               jlong lval = (*double_to_long_bits_fn)(env, NULL, dval);
               dbuf->byte_at_put(off++, (lval >> 56) & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 48) & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 40) & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 32) & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 24) & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 16) & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 8)  & 0xFF);
               dbuf->byte_at_put(off++, (lval >> 0)  & 0xFF);
             }
             break;
    ...
         }
      }
    JVM_END
```




### 詳細(Details)
See: [here](../doxygen/classNativeLookup.html) for details

---
