---
layout: default
title: ByteSize クラスおよび WordSize クラス (ByteSize, WordSize)
---
[Top](../index.html)

#### ByteSize クラスおよび WordSize クラス (ByteSize, WordSize)

これらは, デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外にはクラスとしては定義されず単なる typedef になる).

### 概要(Summary)
これらは, HotSpot 内で使用される様々な「メモリ量を表す値」(e.g. 'sizes', 'offsets', etc) が
「byte 単位」なのか「word 単位」なのかを型レベルで明示的に扱うためのクラス.

(要するに F# の "Units of Measure" のようなもの.
 実質上はどちらも単なる数値データだが, 単位が違うものが混同されにくくすることでバグの混入を抑えられる)

ByteSize は byte 単位での大きさ, WordSize はプロセッサの machine word 単位での大きさを示す
(machine word は環境依存なので 32bit だったり 64bit だったりする).


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
    // The following two classes are used to represent 'sizes' and 'offsets' in the VM;
    // they serve as 'unit' types. ByteSize is used for sizes measured in bytes, while
    // WordSize is used for sizes measured in machine words (i.e., 32bit or 64bit words
    // depending on platform).
    //
    // The classes are defined with friend functions operating on them instead of member
    // functions so that they (the classes) can be re-#define'd to int types in optimized
    // mode. This allows full type checking and maximum safety in debug mode, and full
    // optimizations (constant folding) and zero overhead (time and space wise) in the
    // optimized build (some compilers do not optimize one-element value classes but
    // instead create an object in memory - thus the overhead may be significant).
    //
    // Note: 1) DO NOT add new overloaded friend functions that do not have a unique function
    //          function name but require signature types for resolution. This will not work
    //          in optimized mode as both, ByteSize and WordSize are mapped to the same type
    //          and thus the distinction would not be possible anymore (=> compiler errors).
    //
    //       2) DO NOT add non-static member functions as they cannot be mapped so something
    //          compilable in the optimized build. Static member functions could be added
    //          but require a corresponding class definition in the optimized build.
    //
    // These classes should help doing a transition from (currently) word-size based offsets
    // to byte-size based offsets in the VM (this will be important if we desire to pack
    // objects more densely in the VM for 64bit machines). Such a transition should proceed
    // in two steps to minimize the risk of introducing hard-to-find bugs:
    //
    // a) first transition the whole VM into a form where all sizes are strongly typed
    // b) change all WordSize's to ByteSize's where desired and fix the compilation errors
```

なお, #ifdef ASSERT 時以外にはどちらも単なる int の typedef にしている.
このため, int に typedef すると動かない member function ではなく friend function で実装している.
(コメントによると, int の typedef で動かなくなるような変更はしないように, とのこと).


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
    #ifdef ASSERT
    ...
    #else // ASSERT
    
    // The following definitions must match the corresponding friend declarations
    // in the Byte/WordSize classes if they are typedef'ed to be int. This will
    // be the case in optimized mode to ensure zero overhead for these types.
    //
    // Note: If a compiler does not inline these function calls away, one may
    //       want to use #define's to make sure full optimization (constant
    //       folding in particular) is possible.
    
    typedef int ByteSize;
    inline ByteSize in_ByteSize(int size)                 { return size; }
    inline int      in_bytes   (ByteSize x)               { return x; }
    
    typedef int WordSize;
    inline WordSize in_WordSize(int size)                 { return size; }
    inline int      in_words   (WordSize x)               { return x; }
    
    #endif // ASSERT
```


### クラス一覧(class list)

  * [ByteSize](#noCOLPzE-Q)
  * [WordSize](#noS8bIWwIq)


---
## <a name="noCOLPzE-Q" id="noCOLPzE-Q">ByteSize</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外にはクラスとしては定義されず単なる typedef になる).

「byte 単位」でのメモリ量を表す数値型.


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
    class ByteSize VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
      int _size;
```

(このフィールドへのアクセサは in_bytes() 関数)

```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
      // accessors
      inline friend int in_bytes(ByteSize x);
```

また, 四則演算用に以下の friend function が定義されている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
      // operators
      friend ByteSize operator + (ByteSize x, ByteSize y) { return ByteSize(in_bytes(x) + in_bytes(y)); }
      friend ByteSize operator - (ByteSize x, ByteSize y) { return ByteSize(in_bytes(x) - in_bytes(y)); }
      friend ByteSize operator * (ByteSize x, int      y) { return ByteSize(in_bytes(x) * y          ); }
    
      // comparison
      friend bool operator == (ByteSize x, ByteSize y)    { return in_bytes(x) == in_bytes(y); }
      friend bool operator != (ByteSize x, ByteSize y)    { return in_bytes(x) != in_bytes(y); }
      friend bool operator <  (ByteSize x, ByteSize y)    { return in_bytes(x) <  in_bytes(y); }
      friend bool operator <= (ByteSize x, ByteSize y)    { return in_bytes(x) <= in_bytes(y); }
      friend bool operator >  (ByteSize x, ByteSize y)    { return in_bytes(x) >  in_bytes(y); }
      friend bool operator >= (ByteSize x, ByteSize y)    { return in_bytes(x) >= in_bytes(y); }
```



### 詳細(Details)
See: [here](../doxygen/classByteSize.html) for details

---
## <a name="noS8bIWwIq" id="noS8bIWwIq">WordSize</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外にはクラスとしては定義されず単なる typedef になる).

「word 単位」でのメモリ量を表す数値型 (なお word は環境依存の単位であり 32bit のこともあれば 64bit のこともある).


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
    class WordSize VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
      int _size;
```

(このフィールドへのアクセサは in_words() 関数)

```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
      // accessors
      inline friend int in_words(WordSize x);
```

また, 四則演算用に以下の friend function が定義されている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/sizes.hpp))
      // operators
      friend WordSize operator + (WordSize x, WordSize y) { return WordSize(in_words(x) + in_words(y)); }
      friend WordSize operator - (WordSize x, WordSize y) { return WordSize(in_words(x) - in_words(y)); }
      friend WordSize operator * (WordSize x, int      y) { return WordSize(in_words(x) * y          ); }
    
      // comparison
      friend bool operator == (WordSize x, WordSize y)    { return in_words(x) == in_words(y); }
      friend bool operator != (WordSize x, WordSize y)    { return in_words(x) != in_words(y); }
      friend bool operator <  (WordSize x, WordSize y)    { return in_words(x) <  in_words(y); }
      friend bool operator <= (WordSize x, WordSize y)    { return in_words(x) <= in_words(y); }
      friend bool operator >  (WordSize x, WordSize y)    { return in_words(x) >  in_words(y); }
      friend bool operator >= (WordSize x, WordSize y)    { return in_words(x) >= in_words(y); }
```




### 詳細(Details)
See: [here](../doxygen/classWordSize.html) for details

---
