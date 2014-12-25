---
layout: default
title: JNITypes クラス 
---
[Top](../index.html)

#### JNITypes クラス 



---
## <a name="noXG8HF6zi" id="noXG8HF6zi">JNITypes</a>

### 概要(Summary)
JavaCalls クラス内 (より正確には JavaCallArguments クラス内) で使用される補助クラス.

JavaCalls クラスを用いて Java メソッドを呼び出す際に, 
ネイティブ形式の引数を JavaCallArguments 内に詰める処理を行うクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/cpu/x86/vm/jniTypes_x86.hpp))
    // This file holds platform-dependent routines used to write primitive jni
    // types to the array of arguments passed into JavaCalls::call
    
    class JNITypes : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JavaCallArguments::push_oop()
* JavaCallArguments::push_int()
* JavaCallArguments::push_double()
* JavaCallArguments::push_long()
* JavaCallArguments::push_float()

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている
(大抵は普通に書き込むだけだが, long と double は endian を反転させることもある).


```cpp
    ((cite: hotspot/src/cpu/x86/vm/jniTypes_x86.hpp))
      // These functions write a java primitive type (in native format)
      // to a java stack slot array to be passed as an argument to JavaCalls:calls.
      // I.e., they are functionally 'push' operations if they have a 'pos'
      // formal parameter.  Note that jlong's and jdouble's are written
      // _in reverse_ of the order in which they appear in the interpreter
      // stack.  This is because call stubs (see stubGenerator_sparc.cpp)
      // reverse the argument list constructed by JavaCallArguments (see
      // javaCalls.hpp).
```


```cpp
    ((cite: hotspot/src/cpu/x86/vm/jniTypes_x86.hpp))
    #ifndef AMD64
      // 32bit Helper routines.
      static inline void    put_int2r(jint *from, intptr_t *to)           { *(jint *)(to++) = from[1];
                                                                            *(jint *)(to  ) = from[0]; }
      static inline void    put_int2r(jint *from, intptr_t *to, int& pos) { put_int2r(from, to + pos); pos += 2; }
    #endif // AMD64
    
    public:
      // Ints are stored in native format in one JavaCallArgument slot at *to.
      static inline void    put_int(jint  from, intptr_t *to)           { *(jint *)(to +   0  ) =  from; }
      static inline void    put_int(jint  from, intptr_t *to, int& pos) { *(jint *)(to + pos++) =  from; }
      static inline void    put_int(jint *from, intptr_t *to, int& pos) { *(jint *)(to + pos++) = *from; }
    
    #ifdef AMD64
      // Longs are stored in native format in one JavaCallArgument slot at
      // *(to+1).
      static inline void put_long(jlong  from, intptr_t *to) {
        *(jlong*) (to + 1) = from;
      }
    
      static inline void put_long(jlong  from, intptr_t *to, int& pos) {
        *(jlong*) (to + 1 + pos) = from;
        pos += 2;
      }
    
      static inline void put_long(jlong *from, intptr_t *to, int& pos) {
        *(jlong*) (to + 1 + pos) = *from;
        pos += 2;
      }
    #else
      // Longs are stored in big-endian word format in two JavaCallArgument slots at *to.
      // The high half is in *to and the low half in *(to+1).
      static inline void    put_long(jlong  from, intptr_t *to)           { put_int2r((jint *)&from, to); }
      static inline void    put_long(jlong  from, intptr_t *to, int& pos) { put_int2r((jint *)&from, to, pos); }
      static inline void    put_long(jlong *from, intptr_t *to, int& pos) { put_int2r((jint *) from, to, pos); }
    #endif // AMD64
    
      // Oops are stored in native format in one JavaCallArgument slot at *to.
      static inline void    put_obj(oop  from, intptr_t *to)           { *(oop *)(to +   0  ) =  from; }
      static inline void    put_obj(oop  from, intptr_t *to, int& pos) { *(oop *)(to + pos++) =  from; }
      static inline void    put_obj(oop *from, intptr_t *to, int& pos) { *(oop *)(to + pos++) = *from; }
    
      // Floats are stored in native format in one JavaCallArgument slot at *to.
      static inline void    put_float(jfloat  from, intptr_t *to)           { *(jfloat *)(to +   0  ) =  from;  }
      static inline void    put_float(jfloat  from, intptr_t *to, int& pos) { *(jfloat *)(to + pos++) =  from; }
      static inline void    put_float(jfloat *from, intptr_t *to, int& pos) { *(jfloat *)(to + pos++) = *from; }
    
    #undef _JNI_SLOT_OFFSET
    #ifdef AMD64
    #define _JNI_SLOT_OFFSET 1
      // Doubles are stored in native word format in one JavaCallArgument
      // slot at *(to+1).
      static inline void put_double(jdouble  from, intptr_t *to) {
        *(jdouble*) (to + 1) = from;
      }
    
      static inline void put_double(jdouble  from, intptr_t *to, int& pos) {
        *(jdouble*) (to + 1 + pos) = from;
        pos += 2;
      }
    
      static inline void put_double(jdouble *from, intptr_t *to, int& pos) {
        *(jdouble*) (to + 1 + pos) = *from;
        pos += 2;
      }
    #else
    #define _JNI_SLOT_OFFSET 0
      // Doubles are stored in big-endian word format in two JavaCallArgument slots at *to.
      // The high half is in *to and the low half in *(to+1).
      static inline void    put_double(jdouble  from, intptr_t *to)           { put_int2r((jint *)&from, to); }
      static inline void    put_double(jdouble  from, intptr_t *to, int& pos) { put_int2r((jint *)&from, to, pos); }
      static inline void    put_double(jdouble *from, intptr_t *to, int& pos) { put_int2r((jint *) from, to, pos); }
    #endif // AMD64
    
    
      // The get_xxx routines, on the other hand, actually _do_ fetch
      // java primitive types from the interpreter stack.
      // No need to worry about alignment on Intel.
      static inline jint    get_int   (intptr_t *from) { return *(jint *)   from; }
      static inline jlong   get_long  (intptr_t *from) { return *(jlong *)  (from + _JNI_SLOT_OFFSET); }
      static inline oop     get_obj   (intptr_t *from) { return *(oop *)    from; }
      static inline jfloat  get_float (intptr_t *from) { return *(jfloat *) from; }
      static inline jdouble get_double(intptr_t *from) { return *(jdouble *)(from + _JNI_SLOT_OFFSET); }
    #undef _JNI_SLOT_OFFSET
```




### 詳細(Details)
See: [here](../doxygen/classJNITypes.html) for details

---
