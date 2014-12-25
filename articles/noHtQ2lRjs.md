---
layout: default
title: BitMap クラス関連のクラス (BitMap, BitMap2D, BitMapClosure)
---
[Top](../index.html)

#### BitMap クラス関連のクラス (BitMap, BitMap2D, BitMapClosure)

これらは, ビットマップを表すユーティリティ・クラス.


### クラス一覧(class list)

  * [BitMap](#no_g3HyIxI)
  * [BitMap2D](#noQJTCB6YL)
  * [BitMapClosure](#no-Rfg0SbU)


---
## <a name="no_g3HyIxI" id="no_g3HyIxI">BitMap</a>

### 概要(Summary)
ビットマップを表すユーティリティ・クラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/bitMap.hpp))
    // Operations for bitmaps represented as arrays of unsigned integers.
    // Bit offsets are numbered from 0 to size-1.
    
    class BitMap VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

(remembered set, コンパイラでの liveness analysis, etc).




### 詳細(Details)
See: [here](../doxygen/classBitMap.html) for details

---
## <a name="noQJTCB6YL" id="noQJTCB6YL">BitMap2D</a>

### 概要(Summary)
二次元のビットマップを表すユーティリティ・クラス.
コメントによると, 各スロットに複数のビットを持つような BitMap, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/utilities/bitMap.hpp))
    // Convenience class wrapping BitMap which provides multiple bits per slot.
    class BitMap2D VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
?? (C1 JIT Compiler 関係でしか使われていないように見える... #TODO)




### 詳細(Details)
See: [here](../doxygen/classBitMap2D.html) for details

---
## <a name="no-Rfg0SbU" id="no-Rfg0SbU">BitMapClosure</a>

### 概要(Summary)
BitMap クラス用のユーティリティ・クラス.

ビットマップに対する iterate 処理のための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/bitMap.hpp))
    // Closure for iterating over BitMaps
    
    class BitMapClosure VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classBitMapClosure.html) for details

---
