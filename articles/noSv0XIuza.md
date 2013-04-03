---
layout: default
title: Bytes クラス 
---
[Top](../index.html)

#### Bytes クラス 



---
## <a name="noKzqwJGHN" id="noKzqwJGHN">Bytes</a>

### 概要(Summary)
エンディアン(byte order)の違いに対応するためのユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

エンディアンを考慮しながらメモリを読み書きするためのメソッドを提供している.


```
    ((cite: hotspot/src/cpu/x86/vm/bytes_x86.hpp))
    class Bytes: AllStatic {
```

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.


```
    ((cite: hotspot/src/cpu/x86/vm/bytes_x86.hpp))
    #ifndef AMD64
      // Helper function for swap_u8
      static inline u8   swap_u8_base(u4 x, u4 y);        // compiler-dependent implementation
    #endif // AMD64
```


```
    ((cite: hotspot/src/cpu/x86/vm/bytes_x86.hpp))
      // Returns true if the byte ordering used by Java is different from the native byte ordering
      // of the underlying machine. For example, this is true for Intel x86, but false for Solaris
      // on Sparc.
      static inline bool is_Java_byte_ordering_different(){ return true; }
    
    
      // Efficient reading and writing of unaligned unsigned data in platform-specific byte ordering
      // (no special code is needed since x86 CPUs can access unaligned data)
      static inline u2   get_native_u2(address p)         { return *(u2*)p; }
      static inline u4   get_native_u4(address p)         { return *(u4*)p; }
      static inline u8   get_native_u8(address p)         { return *(u8*)p; }
    
      static inline void put_native_u2(address p, u2 x)   { *(u2*)p = x; }
      static inline void put_native_u4(address p, u4 x)   { *(u4*)p = x; }
      static inline void put_native_u8(address p, u8 x)   { *(u8*)p = x; }
    
    
      // Efficient reading and writing of unaligned unsigned data in Java
      // byte ordering (i.e. big-endian ordering). Byte-order reversal is
      // needed since x86 CPUs use little-endian format.
      static inline u2   get_Java_u2(address p)           { return swap_u2(get_native_u2(p)); }
      static inline u4   get_Java_u4(address p)           { return swap_u4(get_native_u4(p)); }
      static inline u8   get_Java_u8(address p)           { return swap_u8(get_native_u8(p)); }
    
      static inline void put_Java_u2(address p, u2 x)     { put_native_u2(p, swap_u2(x)); }
      static inline void put_Java_u4(address p, u4 x)     { put_native_u4(p, swap_u4(x)); }
      static inline void put_Java_u8(address p, u8 x)     { put_native_u8(p, swap_u8(x)); }
    
    
      // Efficient swapping of byte ordering
      static inline u2   swap_u2(u2 x);                   // compiler-dependent implementation
      static inline u4   swap_u4(u4 x);                   // compiler-dependent implementation
      static inline u8   swap_u8(u8 x);
```




### 詳細(Details)
See: [here](../doxygen/classBytes.html) for details

---
