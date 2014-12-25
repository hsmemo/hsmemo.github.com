---
layout: default
title: Atomic クラス 
---
[Top](../index.html)

#### Atomic クラス 



---
## <a name="nocVOWPu5e" id="nocVOWPu5e">Atomic</a>

### 概要(Summary)
HotSpot 内でのアトミックなメモリ書き換え操作(inc, xchg, 等)用のユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/runtime/atomic.hpp))
    class Atomic : AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
具体的な処理は os や cpu に依存するため, ほとんどのメソッドは os_cpu/ 以下で定義されている.

share/ 以下で定義されているメソッドは以下の3つだけ
(しかも, これらも内部では cpu 依存な処理にフォールバックしている).


```cpp
    ((cite: hotspot/src/share/vm/runtime/atomic.cpp))
    jbyte Atomic::cmpxchg(jbyte exchange_value, volatile jbyte* dest, jbyte compare_value) {
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/atomic.cpp))
    unsigned Atomic::xchg(unsigned int exchange_value, volatile unsigned int* dest) {
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/atomic.cpp))
    unsigned Atomic::cmpxchg(unsigned int exchange_value,
                             volatile unsigned int* dest, unsigned int compare_value) {
```

### 備考(Notes)
cmpxchg 系のメソッドは release/acquire barrier も張ることを仮定している
(だから本当は "cmpxchg" じゃなく "fence_cmpxchg_acquire" なんだ, とのこと. 
 といっても x86 や sparc ではほとんど関係ないが...).


```cpp
    ((cite: hotspot/src/share/vm/runtime/atomic.hpp))
      // Performs atomic compare of *dest and compare_value, and exchanges *dest with exchange_value
      // if the comparison succeeded.  Returns prior value of *dest.  Guarantees a two-way memory
      // barrier across the cmpxchg.  I.e., it's really a 'fence_cmpxchg_acquire'.
      static jbyte    cmpxchg    (jbyte    exchange_value, volatile jbyte*    dest, jbyte    compare_value);
      static jint     cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value);
      static jlong    cmpxchg    (jlong    exchange_value, volatile jlong*    dest, jlong    compare_value);
    
      static unsigned int cmpxchg(unsigned int exchange_value,
                                  volatile unsigned int* dest,
                                  unsigned int compare_value);
    
      static intptr_t cmpxchg_ptr(intptr_t exchange_value, volatile intptr_t* dest, intptr_t compare_value);
      static void*    cmpxchg_ptr(void*    exchange_value, volatile void*     dest, void*    compare_value);
```




### 詳細(Details)
See: [here](../doxygen/classAtomic.html) for details

---
