---
layout: default
title: LibraryIntrinsic クラス関連のクラス (LibraryIntrinsic, LibraryCallKit)
---
[Top](../index.html)

#### LibraryIntrinsic クラス関連のクラス (LibraryIntrinsic, LibraryCallKit)

これらは, C2 JIT Compiler 用のクラス.
より具体的に言うと, 標準ライブラリ内のよく使われるメソッドについて最適化(インライン展開)を行うためのクラス.


### クラス一覧(class list)

  * [LibraryIntrinsic](#no583oNdpy)
  * [LibraryCallKit](#noRDtOrw6O)


---
## <a name="no583oNdpy" id="no583oNdpy">LibraryIntrinsic</a>

### 概要(Summary)
InlineCallGenerator クラスの具象サブクラスの1つ.

標準ライブラリ内の特定のメソッドについて, その中身を呼び出し点に展開する (= メソッドに特化したインライン展開処理を行う) クラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    class LibraryIntrinsic : public InlineCallGenerator {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Compile::make_vm_intrinsic() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::call_generator()
-> Compile::find_intrinsic()
   -> Compile::make_vm_intrinsic()

Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis )
-> Compile::find_intrinsic()
   -> (同上)
```

### 備考(Notes)
現在このクラスのインライン展開対象になっているメソッドは以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/classfile/vmSymbols.hpp))
    // Here are all the intrinsics known to the runtime and the CI.
    // Each intrinsic consists of a public enum name (like _hashCode),
    // followed by a specification of its klass, name, and signature:
    //    template(<id>,  <klass>,  <name>, <sig>, <FCODE>)
    //
    // If you add an intrinsic here, you must also define its name
    // and signature as members of the VM symbols.  The VM symbols for
    // the intrinsic name and signature may be defined above.
    //
    // Because the VM_SYMBOLS_DO macro makes reference to VM_INTRINSICS_DO,
    // you can also define an intrinsic's name and/or signature locally to the
    // intrinsic, if this makes sense.  (It often does make sense.)
    //
    // For example:
    //    do_intrinsic(_foo,  java_lang_Object,  foo_name, foo_signature, F_xx)
    //     do_name(     foo_name, "foo")
    //     do_signature(foo_signature, "()F")
    // klass      = vmSymbols::java_lang_Object()
    // name       = vmSymbols::foo_name()
    // signature  = vmSymbols::foo_signature()
    //
    // The name and/or signature might be a "well known" symbol
    // like "equal" or "()I", in which case there will be no local
    // re-definition of the symbol.
    //
    // The do_class, do_name, and do_signature calls are all used for the
    // same purpose:  Define yet another VM symbol.  They could all be merged
    // into a common 'do_symbol' call, but it seems useful to record our
    // intentions here about kinds of symbols (class vs. name vs. signature).
    //
    // The F_xx is one of the Flags enum; see below.
    //
    // for Emacs: (let ((c-backslash-column 120) (c-backslash-max-column 120)) (c-backslash-region (point) (point-max) nil t))
    #define VM_INTRINSICS_DO(do_intrinsic, do_class, do_name, do_signature, do_alias)                                       \
      do_intrinsic(_hashCode,                 java_lang_Object,       hashCode_name, void_int_signature,             F_R)   \
       do_name(     hashCode_name,                                   "hashCode")                                            \
      do_intrinsic(_getClass,                 java_lang_Object,       getClass_name, void_class_signature,           F_R)   \
       do_name(     getClass_name,                                   "getClass")                                            \
      do_intrinsic(_clone,                    java_lang_Object,       clone_name, void_object_signature,             F_R)   \
       do_name(     clone_name,                                      "clone")                                               \
                                                                                                                            \
      /* Math & StrictMath intrinsics are defined in terms of just a few signatures: */                                     \
      do_class(java_lang_Math,                "java/lang/Math")                                                             \
      do_class(java_lang_StrictMath,          "java/lang/StrictMath")                                                       \
      do_signature(double2_double_signature,  "(DD)D")                                                                      \
      do_signature(int2_int_signature,        "(II)I")                                                                      \
                                                                                                                            \
      /* here are the math names, all together: */                                                                          \
      do_name(abs_name,"abs")       do_name(sin_name,"sin")         do_name(cos_name,"cos")                                 \
      do_name(tan_name,"tan")       do_name(atan2_name,"atan2")     do_name(sqrt_name,"sqrt")                               \
      do_name(log_name,"log")       do_name(log10_name,"log10")     do_name(pow_name,"pow")                                 \
      do_name(exp_name,"exp")       do_name(min_name,"min")         do_name(max_name,"max")                                 \
                                                                                                                            \
      do_intrinsic(_dabs,                     java_lang_Math,         abs_name,   double_double_signature,           F_S)   \
      do_intrinsic(_dsin,                     java_lang_Math,         sin_name,   double_double_signature,           F_S)   \
      do_intrinsic(_dcos,                     java_lang_Math,         cos_name,   double_double_signature,           F_S)   \
      do_intrinsic(_dtan,                     java_lang_Math,         tan_name,   double_double_signature,           F_S)   \
      do_intrinsic(_datan2,                   java_lang_Math,         atan2_name, double2_double_signature,          F_S)   \
      do_intrinsic(_dsqrt,                    java_lang_Math,         sqrt_name,  double_double_signature,           F_S)   \
      do_intrinsic(_dlog,                     java_lang_Math,         log_name,   double_double_signature,           F_S)   \
      do_intrinsic(_dlog10,                   java_lang_Math,         log10_name, double_double_signature,           F_S)   \
      do_intrinsic(_dpow,                     java_lang_Math,         pow_name,   double2_double_signature,          F_S)   \
      do_intrinsic(_dexp,                     java_lang_Math,         exp_name,   double_double_signature,           F_S)   \
      do_intrinsic(_min,                      java_lang_Math,         min_name,   int2_int_signature,                F_S)   \
      do_intrinsic(_max,                      java_lang_Math,         max_name,   int2_int_signature,                F_S)   \
                                                                                                                            \
      do_intrinsic(_floatToRawIntBits,        java_lang_Float,        floatToRawIntBits_name,   float_int_signature, F_S)   \
       do_name(     floatToRawIntBits_name,                          "floatToRawIntBits")                                   \
      do_intrinsic(_floatToIntBits,           java_lang_Float,        floatToIntBits_name,      float_int_signature, F_S)   \
       do_name(     floatToIntBits_name,                             "floatToIntBits")                                      \
      do_intrinsic(_intBitsToFloat,           java_lang_Float,        intBitsToFloat_name,      int_float_signature, F_S)   \
       do_name(     intBitsToFloat_name,                             "intBitsToFloat")                                      \
      do_intrinsic(_doubleToRawLongBits,      java_lang_Double,       doubleToRawLongBits_name, double_long_signature, F_S) \
       do_name(     doubleToRawLongBits_name,                        "doubleToRawLongBits")                                 \
      do_intrinsic(_doubleToLongBits,         java_lang_Double,       doubleToLongBits_name,    double_long_signature, F_S) \
       do_name(     doubleToLongBits_name,                           "doubleToLongBits")                                    \
      do_intrinsic(_longBitsToDouble,         java_lang_Double,       longBitsToDouble_name,    long_double_signature, F_S) \
       do_name(     longBitsToDouble_name,                           "longBitsToDouble")                                    \
                                                                                                                            \
      do_intrinsic(_numberOfLeadingZeros_i,   java_lang_Integer,      numberOfLeadingZeros_name,int_int_signature,   F_S)   \
      do_intrinsic(_numberOfLeadingZeros_l,   java_lang_Long,         numberOfLeadingZeros_name,long_int_signature,  F_S)   \
                                                                                                                            \
      do_intrinsic(_numberOfTrailingZeros_i,  java_lang_Integer,      numberOfTrailingZeros_name,int_int_signature,  F_S)   \
      do_intrinsic(_numberOfTrailingZeros_l,  java_lang_Long,         numberOfTrailingZeros_name,long_int_signature, F_S)   \
                                                                                                                            \
      do_intrinsic(_bitCount_i,               java_lang_Integer,      bitCount_name,            int_int_signature,   F_S)   \
      do_intrinsic(_bitCount_l,               java_lang_Long,         bitCount_name,            long_int_signature,  F_S)   \
                                                                                                                            \
      do_intrinsic(_reverseBytes_i,           java_lang_Integer,      reverseBytes_name,        int_int_signature,   F_S)   \
       do_name(     reverseBytes_name,                               "reverseBytes")                                        \
      do_intrinsic(_reverseBytes_l,           java_lang_Long,         reverseBytes_name,        long_long_signature, F_S)   \
        /*  (symbol reverseBytes_name defined above) */                                                                     \
      do_intrinsic(_reverseBytes_c,           java_lang_Character,    reverseBytes_name,        char_char_signature, F_S)   \
        /*  (symbol reverseBytes_name defined above) */                                                                     \
      do_intrinsic(_reverseBytes_s,           java_lang_Short,        reverseBytes_name,        short_short_signature, F_S) \
        /*  (symbol reverseBytes_name defined above) */                                                                     \
                                                                                                                            \
      do_intrinsic(_identityHashCode,         java_lang_System,       identityHashCode_name, object_int_signature,   F_S)   \
       do_name(     identityHashCode_name,                           "identityHashCode")                                    \
      do_intrinsic(_currentTimeMillis,        java_lang_System,       currentTimeMillis_name, void_long_signature,   F_S)   \
                                                                                                                            \
       do_name(     currentTimeMillis_name,                          "currentTimeMillis")                                   \
      do_intrinsic(_nanoTime,                 java_lang_System,       nanoTime_name,          void_long_signature,   F_S)   \
       do_name(     nanoTime_name,                                   "nanoTime")                                            \
                                                                                                                            \
      do_intrinsic(_arraycopy,                java_lang_System,       arraycopy_name, arraycopy_signature,           F_S)   \
       do_name(     arraycopy_name,                                  "arraycopy")                                           \
       do_signature(arraycopy_signature,                             "(Ljava/lang/Object;ILjava/lang/Object;II)V")          \
      do_intrinsic(_isInterrupted,            java_lang_Thread,       isInterrupted_name, isInterrupted_signature,   F_R)   \
       do_name(     isInterrupted_name,                              "isInterrupted")                                       \
       do_signature(isInterrupted_signature,                         "(Z)Z")                                                \
      do_intrinsic(_currentThread,            java_lang_Thread,       currentThread_name, currentThread_signature,   F_S)   \
       do_name(     currentThread_name,                              "currentThread")                                       \
       do_signature(currentThread_signature,                         "()Ljava/lang/Thread;")                                \
                                                                                                                            \
      /* reflective intrinsics, for java/lang/Class, etc. */                                                                \
      do_intrinsic(_isAssignableFrom,         java_lang_Class,        isAssignableFrom_name, class_boolean_signature, F_RN) \
       do_name(     isAssignableFrom_name,                           "isAssignableFrom")                                    \
      do_intrinsic(_isInstance,               java_lang_Class,        isInstance_name, object_boolean_signature,     F_RN)  \
       do_name(     isInstance_name,                                 "isInstance")                                          \
      do_intrinsic(_getModifiers,             java_lang_Class,        getModifiers_name, void_int_signature,         F_RN)  \
       do_name(     getModifiers_name,                               "getModifiers")                                        \
      do_intrinsic(_isInterface,              java_lang_Class,        isInterface_name, void_boolean_signature,      F_RN)  \
       do_name(     isInterface_name,                                "isInterface")                                         \
      do_intrinsic(_isArray,                  java_lang_Class,        isArray_name, void_boolean_signature,          F_RN)  \
       do_name(     isArray_name,                                    "isArray")                                             \
      do_intrinsic(_isPrimitive,              java_lang_Class,        isPrimitive_name, void_boolean_signature,      F_RN)  \
       do_name(     isPrimitive_name,                                "isPrimitive")                                         \
      do_intrinsic(_getSuperclass,            java_lang_Class,        getSuperclass_name, void_class_signature,      F_RN)  \
       do_name(     getSuperclass_name,                              "getSuperclass")                                       \
      do_intrinsic(_getComponentType,         java_lang_Class,        getComponentType_name, void_class_signature,   F_RN)  \
       do_name(     getComponentType_name,                           "getComponentType")                                    \
                                                                                                                            \
      do_intrinsic(_getClassAccessFlags,      sun_reflect_Reflection, getClassAccessFlags_name, class_int_signature, F_SN)  \
       do_name(     getClassAccessFlags_name,                        "getClassAccessFlags")                                 \
      do_intrinsic(_getLength,                java_lang_reflect_Array, getLength_name, object_int_signature,         F_SN)  \
       do_name(     getLength_name,                                   "getLength")                                          \
                                                                                                                            \
      do_intrinsic(_getCallerClass,           sun_reflect_Reflection, getCallerClass_name, getCallerClass_signature, F_SN)  \
       do_name(     getCallerClass_name,                             "getCallerClass")                                      \
       do_signature(getCallerClass_signature,                        "(I)Ljava/lang/Class;")                                \
                                                                                                                            \
      do_intrinsic(_newArray,                 java_lang_reflect_Array, newArray_name, newArray_signature,            F_SN)  \
       do_name(     newArray_name,                                    "newArray")                                           \
       do_signature(newArray_signature,                               "(Ljava/lang/Class;I)Ljava/lang/Object;")             \
                                                                                                                            \
      do_intrinsic(_copyOf,                   java_util_Arrays,       copyOf_name, copyOf_signature,                 F_S)   \
       do_name(     copyOf_name,                                     "copyOf")                                              \
       do_signature(copyOf_signature,             "([Ljava/lang/Object;ILjava/lang/Class;)[Ljava/lang/Object;")             \
                                                                                                                            \
      do_intrinsic(_copyOfRange,              java_util_Arrays,       copyOfRange_name, copyOfRange_signature,       F_S)   \
       do_name(     copyOfRange_name,                                "copyOfRange")                                         \
       do_signature(copyOfRange_signature,        "([Ljava/lang/Object;IILjava/lang/Class;)[Ljava/lang/Object;")            \
                                                                                                                            \
      do_intrinsic(_equalsC,                  java_util_Arrays,       equals_name,    equalsC_signature,             F_S)   \
       do_signature(equalsC_signature,                               "([C[C)Z")                                             \
                                                                                                                            \
      do_intrinsic(_compareTo,                java_lang_String,       compareTo_name, string_int_signature,          F_R)   \
       do_name(     compareTo_name,                                  "compareTo")                                           \
      do_intrinsic(_indexOf,                  java_lang_String,       indexOf_name, string_int_signature,            F_R)   \
       do_name(     indexOf_name,                                    "indexOf")                                             \
      do_intrinsic(_equals,                   java_lang_String,       equals_name, object_boolean_signature,         F_R)   \
                                                                                                                            \
      do_class(java_nio_Buffer,               "java/nio/Buffer")                                                            \
      do_intrinsic(_checkIndex,               java_nio_Buffer,        checkIndex_name, int_int_signature,            F_R)   \
       do_name(     checkIndex_name,                                 "checkIndex")                                          \
                                                                                                                            \
      /* java/lang/ref/Reference */                                                                                         \
      do_intrinsic(_Reference_get,            java_lang_ref_Reference, get_name,    void_object_signature, F_R)             \
                                                                                                                            \
                                                                                                                            \
      do_class(sun_misc_AtomicLongCSImpl,     "sun/misc/AtomicLongCSImpl")                                                  \
      do_intrinsic(_get_AtomicLong,           sun_misc_AtomicLongCSImpl, get_name, void_long_signature,              F_R)   \
      /*   (symbols get_name and void_long_signature defined above) */                                                      \
                                                                                                                            \
      do_intrinsic(_attemptUpdate,            sun_misc_AtomicLongCSImpl, attemptUpdate_name, attemptUpdate_signature, F_R)  \
       do_name(     attemptUpdate_name,                                 "attemptUpdate")                                    \
       do_signature(attemptUpdate_signature,                            "(JJ)Z")                                            \
                                                                                                                            \
      /* support for sun.misc.Unsafe */                                                                                     \
      do_class(sun_misc_Unsafe,               "sun/misc/Unsafe")                                                            \
                                                                                                                            \
      do_intrinsic(_allocateInstance,         sun_misc_Unsafe,        allocateInstance_name, allocateInstance_signature, F_RN) \
       do_name(     allocateInstance_name,                           "allocateInstance")                                    \
       do_signature(allocateInstance_signature,   "(Ljava/lang/Class;)Ljava/lang/Object;")                                  \
      do_intrinsic(_copyMemory,               sun_misc_Unsafe,        copyMemory_name, copyMemory_signature,         F_RN)  \
       do_name(     copyMemory_name,                                 "copyMemory")                                          \
       do_signature(copyMemory_signature,         "(Ljava/lang/Object;JLjava/lang/Object;JJ)V")                             \
      do_intrinsic(_park,                     sun_misc_Unsafe,        park_name, park_signature,                     F_RN)  \
       do_name(     park_name,                                       "park")                                                \
       do_signature(park_signature,                                  "(ZJ)V")                                               \
      do_intrinsic(_unpark,                   sun_misc_Unsafe,        unpark_name, unpark_signature,                 F_RN)  \
       do_name(     unpark_name,                                     "unpark")                                              \
       do_alias(    unpark_signature,                               /*(LObject;)V*/ object_void_signature)                  \
                                                                                                                            \
      /* unsafe memory references (there are a lot of them...) */                                                           \
      do_signature(getObject_signature,       "(Ljava/lang/Object;J)Ljava/lang/Object;")                                    \
      do_signature(putObject_signature,       "(Ljava/lang/Object;JLjava/lang/Object;)V")                                   \
      do_signature(getBoolean_signature,      "(Ljava/lang/Object;J)Z")                                                     \
      do_signature(putBoolean_signature,      "(Ljava/lang/Object;JZ)V")                                                    \
      do_signature(getByte_signature,         "(Ljava/lang/Object;J)B")                                                     \
      do_signature(putByte_signature,         "(Ljava/lang/Object;JB)V")                                                    \
      do_signature(getShort_signature,        "(Ljava/lang/Object;J)S")                                                     \
      do_signature(putShort_signature,        "(Ljava/lang/Object;JS)V")                                                    \
      do_signature(getChar_signature,         "(Ljava/lang/Object;J)C")                                                     \
      do_signature(putChar_signature,         "(Ljava/lang/Object;JC)V")                                                    \
      do_signature(getInt_signature,          "(Ljava/lang/Object;J)I")                                                     \
      do_signature(putInt_signature,          "(Ljava/lang/Object;JI)V")                                                    \
      do_signature(getLong_signature,         "(Ljava/lang/Object;J)J")                                                     \
      do_signature(putLong_signature,         "(Ljava/lang/Object;JJ)V")                                                    \
      do_signature(getFloat_signature,        "(Ljava/lang/Object;J)F")                                                     \
      do_signature(putFloat_signature,        "(Ljava/lang/Object;JF)V")                                                    \
      do_signature(getDouble_signature,       "(Ljava/lang/Object;J)D")                                                     \
      do_signature(putDouble_signature,       "(Ljava/lang/Object;JD)V")                                                    \
                                                                                                                            \
      do_name(getObject_name,"getObject")           do_name(putObject_name,"putObject")                                     \
      do_name(getBoolean_name,"getBoolean")         do_name(putBoolean_name,"putBoolean")                                   \
      do_name(getByte_name,"getByte")               do_name(putByte_name,"putByte")                                         \
      do_name(getShort_name,"getShort")             do_name(putShort_name,"putShort")                                       \
      do_name(getChar_name,"getChar")               do_name(putChar_name,"putChar")                                         \
      do_name(getInt_name,"getInt")                 do_name(putInt_name,"putInt")                                           \
      do_name(getLong_name,"getLong")               do_name(putLong_name,"putLong")                                         \
      do_name(getFloat_name,"getFloat")             do_name(putFloat_name,"putFloat")                                       \
      do_name(getDouble_name,"getDouble")           do_name(putDouble_name,"putDouble")                                     \
                                                                                                                            \
      do_intrinsic(_getObject,                sun_misc_Unsafe,        getObject_name, getObject_signature,           F_RN)  \
      do_intrinsic(_getBoolean,               sun_misc_Unsafe,        getBoolean_name, getBoolean_signature,         F_RN)  \
      do_intrinsic(_getByte,                  sun_misc_Unsafe,        getByte_name, getByte_signature,               F_RN)  \
      do_intrinsic(_getShort,                 sun_misc_Unsafe,        getShort_name, getShort_signature,             F_RN)  \
      do_intrinsic(_getChar,                  sun_misc_Unsafe,        getChar_name, getChar_signature,               F_RN)  \
      do_intrinsic(_getInt,                   sun_misc_Unsafe,        getInt_name, getInt_signature,                 F_RN)  \
      do_intrinsic(_getLong,                  sun_misc_Unsafe,        getLong_name, getLong_signature,               F_RN)  \
      do_intrinsic(_getFloat,                 sun_misc_Unsafe,        getFloat_name, getFloat_signature,             F_RN)  \
      do_intrinsic(_getDouble,                sun_misc_Unsafe,        getDouble_name, getDouble_signature,           F_RN)  \
      do_intrinsic(_putObject,                sun_misc_Unsafe,        putObject_name, putObject_signature,           F_RN)  \
      do_intrinsic(_putBoolean,               sun_misc_Unsafe,        putBoolean_name, putBoolean_signature,         F_RN)  \
      do_intrinsic(_putByte,                  sun_misc_Unsafe,        putByte_name, putByte_signature,               F_RN)  \
      do_intrinsic(_putShort,                 sun_misc_Unsafe,        putShort_name, putShort_signature,             F_RN)  \
      do_intrinsic(_putChar,                  sun_misc_Unsafe,        putChar_name, putChar_signature,               F_RN)  \
      do_intrinsic(_putInt,                   sun_misc_Unsafe,        putInt_name, putInt_signature,                 F_RN)  \
      do_intrinsic(_putLong,                  sun_misc_Unsafe,        putLong_name, putLong_signature,               F_RN)  \
      do_intrinsic(_putFloat,                 sun_misc_Unsafe,        putFloat_name, putFloat_signature,             F_RN)  \
      do_intrinsic(_putDouble,                sun_misc_Unsafe,        putDouble_name, putDouble_signature,           F_RN)  \
                                                                                                                            \
      do_name(getObjectVolatile_name,"getObjectVolatile")   do_name(putObjectVolatile_name,"putObjectVolatile")             \
      do_name(getBooleanVolatile_name,"getBooleanVolatile") do_name(putBooleanVolatile_name,"putBooleanVolatile")           \
      do_name(getByteVolatile_name,"getByteVolatile")       do_name(putByteVolatile_name,"putByteVolatile")                 \
      do_name(getShortVolatile_name,"getShortVolatile")     do_name(putShortVolatile_name,"putShortVolatile")               \
      do_name(getCharVolatile_name,"getCharVolatile")       do_name(putCharVolatile_name,"putCharVolatile")                 \
      do_name(getIntVolatile_name,"getIntVolatile")         do_name(putIntVolatile_name,"putIntVolatile")                   \
      do_name(getLongVolatile_name,"getLongVolatile")       do_name(putLongVolatile_name,"putLongVolatile")                 \
      do_name(getFloatVolatile_name,"getFloatVolatile")     do_name(putFloatVolatile_name,"putFloatVolatile")               \
      do_name(getDoubleVolatile_name,"getDoubleVolatile")   do_name(putDoubleVolatile_name,"putDoubleVolatile")             \
                                                                                                                            \
      do_intrinsic(_getObjectVolatile,        sun_misc_Unsafe,        getObjectVolatile_name, getObject_signature,   F_RN)  \
      do_intrinsic(_getBooleanVolatile,       sun_misc_Unsafe,        getBooleanVolatile_name, getBoolean_signature, F_RN)  \
      do_intrinsic(_getByteVolatile,          sun_misc_Unsafe,        getByteVolatile_name, getByte_signature,       F_RN)  \
      do_intrinsic(_getShortVolatile,         sun_misc_Unsafe,        getShortVolatile_name, getShort_signature,     F_RN)  \
      do_intrinsic(_getCharVolatile,          sun_misc_Unsafe,        getCharVolatile_name, getChar_signature,       F_RN)  \
      do_intrinsic(_getIntVolatile,           sun_misc_Unsafe,        getIntVolatile_name, getInt_signature,         F_RN)  \
      do_intrinsic(_getLongVolatile,          sun_misc_Unsafe,        getLongVolatile_name, getLong_signature,       F_RN)  \
      do_intrinsic(_getFloatVolatile,         sun_misc_Unsafe,        getFloatVolatile_name, getFloat_signature,     F_RN)  \
      do_intrinsic(_getDoubleVolatile,        sun_misc_Unsafe,        getDoubleVolatile_name, getDouble_signature,   F_RN)  \
      do_intrinsic(_putObjectVolatile,        sun_misc_Unsafe,        putObjectVolatile_name, putObject_signature,   F_RN)  \
      do_intrinsic(_putBooleanVolatile,       sun_misc_Unsafe,        putBooleanVolatile_name, putBoolean_signature, F_RN)  \
      do_intrinsic(_putByteVolatile,          sun_misc_Unsafe,        putByteVolatile_name, putByte_signature,       F_RN)  \
      do_intrinsic(_putShortVolatile,         sun_misc_Unsafe,        putShortVolatile_name, putShort_signature,     F_RN)  \
      do_intrinsic(_putCharVolatile,          sun_misc_Unsafe,        putCharVolatile_name, putChar_signature,       F_RN)  \
      do_intrinsic(_putIntVolatile,           sun_misc_Unsafe,        putIntVolatile_name, putInt_signature,         F_RN)  \
      do_intrinsic(_putLongVolatile,          sun_misc_Unsafe,        putLongVolatile_name, putLong_signature,       F_RN)  \
      do_intrinsic(_putFloatVolatile,         sun_misc_Unsafe,        putFloatVolatile_name, putFloat_signature,     F_RN)  \
      do_intrinsic(_putDoubleVolatile,        sun_misc_Unsafe,        putDoubleVolatile_name, putDouble_signature,   F_RN)  \
                                                                                                                            \
      /* %%% these are redundant except perhaps for getAddress, but Unsafe has native methods for them */                   \
      do_signature(getByte_raw_signature,     "(J)B")                                                                       \
      do_signature(putByte_raw_signature,     "(JB)V")                                                                      \
      do_signature(getShort_raw_signature,    "(J)S")                                                                       \
      do_signature(putShort_raw_signature,    "(JS)V")                                                                      \
      do_signature(getChar_raw_signature,     "(J)C")                                                                       \
      do_signature(putChar_raw_signature,     "(JC)V")                                                                      \
      do_signature(putInt_raw_signature,      "(JI)V")                                                                      \
          do_alias(getLong_raw_signature,    /*(J)J*/ long_long_signature)                                                  \
          do_alias(putLong_raw_signature,    /*(JJ)V*/ long_long_void_signature)                                            \
      do_signature(getFloat_raw_signature,    "(J)F")                                                                       \
      do_signature(putFloat_raw_signature,    "(JF)V")                                                                      \
          do_alias(getDouble_raw_signature,  /*(J)D*/ long_double_signature)                                                \
      do_signature(putDouble_raw_signature,   "(JD)V")                                                                      \
          do_alias(getAddress_raw_signature, /*(J)J*/ long_long_signature)                                                  \
          do_alias(putAddress_raw_signature, /*(JJ)V*/ long_long_void_signature)                                            \
                                                                                                                            \
       do_name(    getAddress_name,           "getAddress")                                                                 \
       do_name(    putAddress_name,           "putAddress")                                                                 \
                                                                                                                            \
      do_intrinsic(_getByte_raw,              sun_misc_Unsafe,        getByte_name, getByte_raw_signature,           F_RN)  \
      do_intrinsic(_getShort_raw,             sun_misc_Unsafe,        getShort_name, getShort_raw_signature,         F_RN)  \
      do_intrinsic(_getChar_raw,              sun_misc_Unsafe,        getChar_name, getChar_raw_signature,           F_RN)  \
      do_intrinsic(_getInt_raw,               sun_misc_Unsafe,        getInt_name, long_int_signature,               F_RN)  \
      do_intrinsic(_getLong_raw,              sun_misc_Unsafe,        getLong_name, getLong_raw_signature,           F_RN)  \
      do_intrinsic(_getFloat_raw,             sun_misc_Unsafe,        getFloat_name, getFloat_raw_signature,         F_RN)  \
      do_intrinsic(_getDouble_raw,            sun_misc_Unsafe,        getDouble_name, getDouble_raw_signature,       F_RN)  \
      do_intrinsic(_getAddress_raw,           sun_misc_Unsafe,        getAddress_name, getAddress_raw_signature,     F_RN)  \
      do_intrinsic(_putByte_raw,              sun_misc_Unsafe,        putByte_name, putByte_raw_signature,           F_RN)  \
      do_intrinsic(_putShort_raw,             sun_misc_Unsafe,        putShort_name, putShort_raw_signature,         F_RN)  \
      do_intrinsic(_putChar_raw,              sun_misc_Unsafe,        putChar_name, putChar_raw_signature,           F_RN)  \
      do_intrinsic(_putInt_raw,               sun_misc_Unsafe,        putInt_name, putInt_raw_signature,             F_RN)  \
      do_intrinsic(_putLong_raw,              sun_misc_Unsafe,        putLong_name, putLong_raw_signature,           F_RN)  \
      do_intrinsic(_putFloat_raw,             sun_misc_Unsafe,        putFloat_name, putFloat_raw_signature,         F_RN)  \
      do_intrinsic(_putDouble_raw,            sun_misc_Unsafe,        putDouble_name, putDouble_raw_signature,       F_RN)  \
      do_intrinsic(_putAddress_raw,           sun_misc_Unsafe,        putAddress_name, putAddress_raw_signature,     F_RN)  \
                                                                                                                            \
      do_intrinsic(_compareAndSwapObject,     sun_misc_Unsafe,        compareAndSwapObject_name, compareAndSwapObject_signature, F_RN) \
       do_name(     compareAndSwapObject_name,                       "compareAndSwapObject")                                \
       do_signature(compareAndSwapObject_signature,  "(Ljava/lang/Object;JLjava/lang/Object;Ljava/lang/Object;)Z")          \
      do_intrinsic(_compareAndSwapLong,       sun_misc_Unsafe,        compareAndSwapLong_name, compareAndSwapLong_signature, F_RN) \
       do_name(     compareAndSwapLong_name,                         "compareAndSwapLong")                                  \
       do_signature(compareAndSwapLong_signature,                    "(Ljava/lang/Object;JJJ)Z")                            \
      do_intrinsic(_compareAndSwapInt,        sun_misc_Unsafe,        compareAndSwapInt_name, compareAndSwapInt_signature, F_RN) \
       do_name(     compareAndSwapInt_name,                          "compareAndSwapInt")                                   \
       do_signature(compareAndSwapInt_signature,                     "(Ljava/lang/Object;JII)Z")                            \
      do_intrinsic(_putOrderedObject,         sun_misc_Unsafe,        putOrderedObject_name, putOrderedObject_signature, F_RN) \
       do_name(     putOrderedObject_name,                           "putOrderedObject")                                    \
       do_alias(    putOrderedObject_signature,                     /*(LObject;JLObject;)V*/ putObject_signature)           \
      do_intrinsic(_putOrderedLong,           sun_misc_Unsafe,        putOrderedLong_name, putOrderedLong_signature, F_RN)  \
       do_name(     putOrderedLong_name,                             "putOrderedLong")                                      \
       do_alias(    putOrderedLong_signature,                       /*(Ljava/lang/Object;JJ)V*/ putLong_signature)          \
      do_intrinsic(_putOrderedInt,            sun_misc_Unsafe,        putOrderedInt_name, putOrderedInt_signature,   F_RN)  \
       do_name(     putOrderedInt_name,                              "putOrderedInt")                                       \
       do_alias(    putOrderedInt_signature,                        /*(Ljava/lang/Object;JI)V*/ putInt_signature)           \
                                                                                                                            \
      /* prefetch_signature is shared by all prefetch variants */                                                           \
      do_signature( prefetch_signature,        "(Ljava/lang/Object;J)V")                                                    \
                                                                                                                            \
      do_intrinsic(_prefetchRead,             sun_misc_Unsafe,        prefetchRead_name, prefetch_signature,         F_RN)  \
       do_name(     prefetchRead_name,                               "prefetchRead")                                        \
      do_intrinsic(_prefetchWrite,            sun_misc_Unsafe,        prefetchWrite_name, prefetch_signature,        F_RN)  \
       do_name(     prefetchWrite_name,                              "prefetchWrite")                                       \
      do_intrinsic(_prefetchReadStatic,       sun_misc_Unsafe,        prefetchReadStatic_name, prefetch_signature,   F_SN)  \
       do_name(     prefetchReadStatic_name,                         "prefetchReadStatic")                                  \
      do_intrinsic(_prefetchWriteStatic,      sun_misc_Unsafe,        prefetchWriteStatic_name, prefetch_signature,  F_SN)  \
       do_name(     prefetchWriteStatic_name,                        "prefetchWriteStatic")                                 \
        /*== LAST_COMPILER_INLINE*/                                                                                         \
        /*the compiler does have special inlining code for these; bytecode inline is just fine */                           \
                                                                                                                            \
      do_intrinsic(_fillInStackTrace,         java_lang_Throwable, fillInStackTrace_name, void_throwable_signature,  F_RNY) \
                                                                                                                              \
      do_intrinsic(_StringBuilder_void,   java_lang_StringBuilder, object_initializer_name, void_method_signature,     F_R)   \
      do_intrinsic(_StringBuilder_int,    java_lang_StringBuilder, object_initializer_name, int_void_signature,        F_R)   \
      do_intrinsic(_StringBuilder_String, java_lang_StringBuilder, object_initializer_name, string_void_signature,     F_R)   \
                                                                                                                              \
      do_intrinsic(_StringBuilder_append_char,   java_lang_StringBuilder, append_name, char_StringBuilder_signature,   F_R)   \
      do_intrinsic(_StringBuilder_append_int,    java_lang_StringBuilder, append_name, int_StringBuilder_signature,    F_R)   \
      do_intrinsic(_StringBuilder_append_String, java_lang_StringBuilder, append_name, String_StringBuilder_signature, F_R)   \
                                                                                                                              \
      do_intrinsic(_StringBuilder_toString, java_lang_StringBuilder, toString_name, void_string_signature,             F_R)   \
                                                                                                                              \
      do_intrinsic(_StringBuffer_void,   java_lang_StringBuffer, object_initializer_name, void_method_signature,       F_R)   \
      do_intrinsic(_StringBuffer_int,    java_lang_StringBuffer, object_initializer_name, int_void_signature,          F_R)   \
      do_intrinsic(_StringBuffer_String, java_lang_StringBuffer, object_initializer_name, string_void_signature,       F_R)   \
                                                                                                                              \
      do_intrinsic(_StringBuffer_append_char,   java_lang_StringBuffer, append_name, char_StringBuffer_signature,      F_Y)   \
      do_intrinsic(_StringBuffer_append_int,    java_lang_StringBuffer, append_name, int_StringBuffer_signature,       F_Y)   \
      do_intrinsic(_StringBuffer_append_String, java_lang_StringBuffer, append_name, String_StringBuffer_signature,    F_Y)   \
                                                                                                                              \
      do_intrinsic(_StringBuffer_toString,  java_lang_StringBuffer, toString_name, void_string_signature,              F_Y)   \
                                                                                                                              \
      do_intrinsic(_Integer_toString,      java_lang_Integer, toString_name, int_String_signature,                     F_S)   \
                                                                                                                              \
      do_intrinsic(_String_String, java_lang_String, object_initializer_name, string_void_signature,                   F_R)   \
                                                                                                                              \
      do_intrinsic(_Object_init,              java_lang_Object, object_initializer_name, void_method_signature,        F_R)   \
      /*    (symbol object_initializer_name defined above) */                                                                 \
                                                                                                                              \
      do_intrinsic(_invoke,                   java_lang_reflect_Method, invoke_name, object_object_array_object_signature, F_R) \
      /*   (symbols invoke_name and invoke_signature defined above) */                                                      \
      do_intrinsic(_checkSpreadArgument,      java_lang_invoke_MethodHandleNatives, checkSpreadArgument_name, checkSpreadArgument_signature, F_S) \
       do_name(    checkSpreadArgument_name,       "checkSpreadArgument")                                                   \
       do_name(    checkSpreadArgument_signature,  "(Ljava/lang/Object;I)V")                                                \
      do_intrinsic(_invokeExact,              java_lang_invoke_MethodHandle, invokeExact_name,   object_array_object_signature, F_RN) \
      do_intrinsic(_invokeGeneric,            java_lang_invoke_MethodHandle, invokeGeneric_name, object_array_object_signature, F_RN) \
      do_intrinsic(_invokeVarargs,            java_lang_invoke_MethodHandle, invokeVarargs_name, object_array_object_signature, F_R)  \
      do_intrinsic(_invokeDynamic,            java_lang_invoke_InvokeDynamic, star_name,         object_array_object_signature, F_SN) \
                                                                                                                            \
      /* unboxing methods: */                                                                                               \
      do_intrinsic(_booleanValue,             java_lang_Boolean,      booleanValue_name, void_boolean_signature, F_R)       \
       do_name(     booleanValue_name,       "booleanValue")                                                                \
      do_intrinsic(_byteValue,                java_lang_Byte,         byteValue_name, void_byte_signature, F_R)             \
       do_name(     byteValue_name,          "byteValue")                                                                   \
      do_intrinsic(_charValue,                java_lang_Character,    charValue_name, void_char_signature, F_R)             \
       do_name(     charValue_name,          "charValue")                                                                   \
      do_intrinsic(_shortValue,               java_lang_Short,        shortValue_name, void_short_signature, F_R)           \
       do_name(     shortValue_name,         "shortValue")                                                                  \
      do_intrinsic(_intValue,                 java_lang_Integer,      intValue_name, void_int_signature, F_R)               \
       do_name(     intValue_name,           "intValue")                                                                    \
      do_intrinsic(_longValue,                java_lang_Long,         longValue_name, void_long_signature, F_R)             \
       do_name(     longValue_name,          "longValue")                                                                   \
      do_intrinsic(_floatValue,               java_lang_Float,        floatValue_name, void_float_signature, F_R)           \
       do_name(     floatValue_name,         "floatValue")                                                                  \
      do_intrinsic(_doubleValue,              java_lang_Double,       doubleValue_name, void_double_signature, F_R)         \
       do_name(     doubleValue_name,        "doubleValue")                                                                 \
                                                                                                                            \
      /* boxing methods: */                                                                                                 \
       do_name(    valueOf_name,              "valueOf")                                                                    \
      do_intrinsic(_Boolean_valueOf,          java_lang_Boolean,      valueOf_name, Boolean_valueOf_signature, F_S)         \
       do_name(     Boolean_valueOf_signature,                       "(Z)Ljava/lang/Boolean;")                              \
      do_intrinsic(_Byte_valueOf,             java_lang_Byte,         valueOf_name, Byte_valueOf_signature, F_S)            \
       do_name(     Byte_valueOf_signature,                          "(B)Ljava/lang/Byte;")                                 \
      do_intrinsic(_Character_valueOf,        java_lang_Character,    valueOf_name, Character_valueOf_signature, F_S)       \
       do_name(     Character_valueOf_signature,                     "(C)Ljava/lang/Character;")                            \
      do_intrinsic(_Short_valueOf,            java_lang_Short,        valueOf_name, Short_valueOf_signature, F_S)           \
       do_name(     Short_valueOf_signature,                         "(S)Ljava/lang/Short;")                                \
      do_intrinsic(_Integer_valueOf,          java_lang_Integer,      valueOf_name, Integer_valueOf_signature, F_S)         \
       do_name(     Integer_valueOf_signature,                       "(I)Ljava/lang/Integer;")                              \
      do_intrinsic(_Long_valueOf,             java_lang_Long,         valueOf_name, Long_valueOf_signature, F_S)            \
       do_name(     Long_valueOf_signature,                          "(J)Ljava/lang/Long;")                                 \
      do_intrinsic(_Float_valueOf,            java_lang_Float,        valueOf_name, Float_valueOf_signature, F_S)           \
       do_name(     Float_valueOf_signature,                         "(F)Ljava/lang/Float;")                                \
      do_intrinsic(_Double_valueOf,           java_lang_Double,       valueOf_name, Double_valueOf_signature, F_S)          \
       do_name(     Double_valueOf_signature,                        "(D)Ljava/lang/Double;")                               \
                                                                                                                            \
        /*end*/
```




### 詳細(Details)
See: [here](../doxygen/classLibraryIntrinsic.html) for details

---
## <a name="noRDtOrw6O" id="noRDtOrw6O">LibraryCallKit</a>

### 概要(Summary)
LibraryIntrinsic クラス内で使用される補助クラス.

LibraryIntrinsic 用の Ideal 生成処理を行う GraphKit クラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
    // Local helper class for LibraryIntrinsic:
    class LibraryCallKit : public GraphKit {
```

### 使われ方(Usage)
LibraryIntrinsic::generate() 内で(のみ)使用されている.

### 内部構造(Internal structure)
LibraryIntrinsic::try_to_inline() で全てのインライン展開処理が行われている (See: [here](no7882AZO.html) for details).





### 詳細(Details)
See: [here](../doxygen/classLibraryCallKit.html) for details

---
