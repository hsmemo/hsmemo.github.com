---
layout: default
title: oopDesc クラス 
---
[Top](../index.html)

#### oopDesc クラス 



---
## <a name="noCg_H-g57" id="noCg_H-g57">oopDesc</a>

### 概要(Summary)
全ての oopDesc クラスの基底クラス.

なお, "oop" とは "ordinary object pointer" の略らしい.
「GC の対象になるヒープ中で管理されるオブジェクト (を指すポインタ)」 を表す.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/oop.hpp))
    // oopDesc is the top baseclass for objects classes.  The {name}Desc classes describe
    // the format of Java objects so the fields can be accessed from C++.
    // oopDesc is abstract.
    // (see oopHierarchy for complete oop class hierarchy)
    //
    // no virtual functions allowed
```


```cpp
    ((cite: hotspot/src/share/vm/oops/oop.hpp))
    class oopDesc {
```

### 内部構造(Internal structure)
oopDesc は以下の様な構造をしている (サブクラスではさらにフィールドが追加されている).

* 先頭 1word が markOop. これでオブジェクトのロック管理や GC 処理を行う. (See: markOopDesc)

* 次の 1word (UseCompressedOop 時なら32bit)が klassOopDesc.
  その oopDesc オブジェクトのクラスを示す (e.g. instanceOopDesc, arrayOopDesc, methodOopDesc, etc).
  (See: klassOopDesc)


```cpp
    ((cite: hotspot/src/share/vm/oops/oop.hpp))
      volatile markOop  _mark;
      union _metadata {
        wideKlassOop    _klass;
        narrowOop       _compressed_klass;
      } _metadata;
```

### 備考(Notes)
なお, 実際の使用箇所では oop という別名(もしくはラッパークラス)で使われることが多い (See: oop).

### 備考(Notes)
このクラスのサブクラスは以下のような継承関係を持つ.

         oopDesc                    -- 全ての oopDesc の基底. abstract class
           instanceOopDesc          -- 「Java のインスタンスオブジェクト」を表す oopDesc
           methodOopDesc            --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           constMethodOopDesc       --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           methodDataOopDesc        --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           arrayOopDesc             -- 「Java の配列」を表す oopDesc. abstract class
             objArrayOopDesc        -- 「Java のポインタ型の配列」を表す oopDesc. 
             typeArrayOopDesc       -- 「Java の primitive 型の配列」を表す oopDesc. 
           constantPoolOopDesc      --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           constantPoolCacheOopDesc --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           klassOopDesc             --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           markOopDesc              --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)
           compiledICHolderOopDesc  --     (Java の世界に直接は関係しない oopDesc. HotSpot の内部処理用)



### 詳細(Details)
See: [here](../doxygen/classoopDesc.html) for details

---
