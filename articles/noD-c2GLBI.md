---
layout: default
title: HotSpot 内で広く利用されるユーティリティ・クラス (HeapWord, Padded, Padded01, JavaValue)
---
[Top](../index.html)

#### HotSpot 内で広く利用されるユーティリティ・クラス (HeapWord, Padded, Padded01, JavaValue)



### クラス一覧(class list)

  * [HeapWord](#noHW-OBwbR)
  * [Padded](#noGCRvC7dC)
  * [Padded01](#noQZ5cfE5T)
  * [JavaValue](#noUzZmfIAm)


---
## <a name="noHW-OBwbR" id="noHW-OBwbR">HeapWord</a>

### 概要(Summary)
実行対象のアーキテクチャにおける "word size" の大きさを示すためのユーティリティ・クラス.

HeapWord* 型が 「word-aligned なアドレスを指す汎用的なポインタ型」 として HotSpot 内の様々な箇所で使用されている.

なおコメントによると, 
「HotSpot 内ではオブジェクトの大きさも HeapWord 単位で表すことになっている.
このため, 以下のようなコードでヒープ中のオブジェクトを先頭から辿っていくことができる」 とのこと
(foo() はオブジェクトの大きさを HeapWord 単位で返すメソッドと想定).

            HeapWord* hw;
            hw += oop(hw)->foo();


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // An opaque struct of heap-word width, so that HeapWord* can be a generic
    // pointer into the heap.  We require that object sizes be measured in
    // units of heap words, so that that
    //   HeapWord* hw;
    //   hw += oop(hw)->foo();
    // works, where foo is a method (like size or scavenge) that returns the
    // object size.
    class HeapWord {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.

(このフィールドも特に使われるわけではなく, 単に HeapWord オブジェクトのアドレスを word-aligned させるためだけのもの)


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
      char* i;
```




### 詳細(Details)
See: [here](../doxygen/classHeapWord.html) for details

---
## <a name="noGCRvC7dC" id="noGCRvC7dC">Padded</a>

### 概要(Summary)
既存のクラスを基に, (cache line の競合を避けるためのパディングを入れた) 新しいクラスを生成するユーティリティ・クラス.


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // Templates to create a subclass padded to avoid cache line sharing.  These are
    // effective only when applied to derived-most (leaf) classes.
```


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // When no args are passed to the base ctor.
    template <class T, size_t alignment = DEFAULT_CACHE_LINE_SIZE>
    class Padded: public T {
```


### 使われ方(Usage)
#### 使用方法の概要(how to use)
テンプレート引数で基にしたいクラスを指定すると,
指定したクラスのサブクラスとして (パディングを入れた) 新しいクラスが生成される.

#### 使用箇所(where its instances are used)
現在は, OopTaskQueue にパディングを入れた Padded<OopTaskQueue> クラスを作るために(のみ)使用されている
(このクラスは ParNew GC や CMS GC で使用されている).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parNew/parNewGeneration.hpp))
    typedef Padded<OopTaskQueue> ObjToScanQueue;
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/parNew/parOopClosures.hpp))
    typedef Padded<OopTaskQueue> ObjToScanQueue;
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.cpp))
          typedef Padded<OopTaskQueue> PaddedOopTaskQueue;
```

### 内部構造(Internal structure)
内部には char 配列を持っている (これによってパディングを入れる).

この配列の長さは以下の 2つに応じて変わる (See: PADDING_SIZE()).

* template 引数で指定されているパディング対象のクラス(T)
* alignment 値


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    private:
      char _pad_buf_[PADDING_SIZE(T, alignment)];
```




### 詳細(Details)
See: [here](../doxygen/classPadded.html) for details

---
## <a name="noQZ5cfE5T" id="noQZ5cfE5T">Padded01</a>

### 概要(Summary)
既存のクラスを基に, (cache line の競合を避けるためのパディングを入れた) 新しいクラスを生成するユーティリティ・クラス.

?? (Padded と異なり, こっちは使用箇所が見当たらないが...)


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // Templates to create a subclass padded to avoid cache line sharing.  These are
    // effective only when applied to derived-most (leaf) classes.
```


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // When either 0 or 1 args may be passed to the base ctor.
    template <class T, typename Arg1T, size_t alignment = DEFAULT_CACHE_LINE_SIZE>
    class Padded01: public T {
```

(Padded との違いは, パディング対象のクラス(T)のベースコンストラクタに指定の引数を渡せる点)


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
      Padded01(): T() { }
      Padded01(Arg1T arg1): T(arg1) { }
```

### 使われ方(Usage)
(現状では使われていない)




### 詳細(Details)
See: [here](../doxygen/classPadded01.html) for details

---
## <a name="noUzZmfIAm" id="noUzZmfIAm">JavaValue</a>

### 概要(Summary)
JVM 仕様で定められている任意の値を格納できるクラス
(全部の型の tagged union のようなクラス).


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // JavaValue serves as a container for arbitrary Java values.
    
    class JavaValue {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.

* _type フィールド : 保持している値の型を示すフィールド
* _value フィールド : 値そのものを格納するフィールド


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
      BasicType _type;
      JavaCallValue _value;
```

なお, JavaCallValue は以下のような union 型になっている.


```
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
      typedef union JavaCallValue {
        jfloat   f;
        jdouble  d;
        jint     i;
        jlong    l;
        jobject  h;
      } JavaCallValue;
```




### 詳細(Details)
See: [here](../doxygen/classJavaValue.html) for details

---
