---
layout: default
title: Histogram クラス関連のクラス (HistogramElement, Histogram)
---
[Top](../index.html)

#### Histogram クラス関連のクラス (HistogramElement, Histogram)

これらは, デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).


```cpp
    ((cite: hotspot/src/share/vm/utilities/histogram.hpp))
    #ifdef ASSERT
```

### 概要(Summary)
これらは, 様々な統計情報を集めるためのクラス.

といっても現状の実装では「何らかのイベントが発生した回数を記録する」という処理に特化している
(典型的な使われ方は, 指定の関数が呼び出された回数を記録する, というもの).

簡単な使い方は以下の通り
(なおこの例も, 特定の関数が何度呼び出されたかを記録する, というもの.
記録した結果は Histogram::print_on() で出力できる.)

  1. Histogram* 型の変数を宣言する.

  2. HistogramElement クラスのサブクラスを作る.

     ("1." で作った Histogram* 型の変数は, このサブクラスの中で遅延初期化すればいい)

        class MyHistogramElement : public HistogramElement {
          public:
            MyHistogramElement(char* name);
        };

        MyHistogramElement::MyHistogramElement(char* elementName) {
          _name = elementName;

          if(MyHistogram == NULL)
            MyHistogram = new Histogram("My Call Counts",100);

          MyHistogram->add_element(this);
        }

  3. "2." で作った HistogramElement のサブクラスを使うためのマクロを定義しておく.
      
        #define MyCountWrapper(arg) static MyHistogramElement* e = new MyHistogramElement(arg); e->increment_count()
  
  4. 呼び出し回数を記録したい関数の先頭箇所に, "3."で作ったマクロを記述する.

        void a_function_that_is_being_counted() {
          MyCountWrapper("FunctionName");
          ...
        }


```cpp
    ((cite: hotspot/src/share/vm/utilities/histogram.hpp))
    // This class provides a framework for collecting various statistics.
    // The current implementation is oriented towards counting invocations
    // of various types, but that can be easily changed.
    //
    // To use it, you need to declare a Histogram*, and a subtype of
    // HistogramElement:
    //
    //  HistogramElement* MyHistogram;
    //
    //  class MyHistogramElement : public HistogramElement {
    //    public:
    //      MyHistogramElement(char* name);
    //  };
    //
    //  MyHistogramElement::MyHistogramElement(char* elementName) {
    //    _name = elementName;
    //
    //    if(MyHistogram == NULL)
    //      MyHistogram = new Histogram("My Call Counts",100);
    //
    //    MyHistogram->add_element(this);
    //  }
    //
    //  #define MyCountWrapper(arg) static MyHistogramElement* e = new MyHistogramElement(arg); e->increment_count()
    //
    // This gives you a simple way to count invocations of specfic functions:
    //
    // void a_function_that_is_being_counted() {
    //   MyCountWrapper("FunctionName");
    //   ...
    // }
    //
    // To print the results, invoke print() on your Histogram*.
```



### クラス一覧(class list)

  * [HistogramElement](#no8RhGBpmF)
  * [Histogram](#no-HB3Uvyb)


---
## <a name="no8RhGBpmF" id="no8RhGBpmF">HistogramElement</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

Histogram クラス内で使用される補助クラス
(Histogram 内に格納される要素を表すクラスの基底クラス).

1つの HistogramElement オブジェクトが 1つの統計情報に対応する.


```cpp
    ((cite: hotspot/src/share/vm/utilities/histogram.hpp))
    class HistogramElement : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classHistogramElement.html) for details

---
## <a name="no-HB3Uvyb" id="no-HB3Uvyb">Histogram</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

様々な統計データの収集/出力を行うためのユーティリティ・クラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/histogram.hpp))
    class Histogram : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
現状では, 以下の大域変数に(のみ)格納されている.

* RuntimeHistogram
* JNIHistogram
* JVMHistogram

#### 情報の出力箇所(where the recorded information is output)
集めた結果は, print_statistics() 内で出力されている.

ただし, (デバッグ時であることに加えて) それぞれ CountRuntimeCalls オプション, CountJNICalls オプション, CountJVMCalls オプションが指定されている場合にしか出力処理は行われない.




### 詳細(Details)
See: [here](../doxygen/classHistogram.html) for details

---
