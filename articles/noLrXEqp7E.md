---
layout: default
title: IntHistogram クラス 
---
[Top](../index.html)

#### IntHistogram クラス 



---
## <a name="nohku_48GN" id="nohku_48GN">IntHistogram</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (現状では特定のマクロ定数が #define されている場合にしか使用されない) (See: CARD_REPEAT_HISTO, FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT)

名前の通り integer 値に関するヒストグラムを生成するクラス.
どんな int 値が何個存在しているか, という情報を蓄える機能を持つ.

なおコメントによると, 
類似した名前のクラスに Histogram クラスがあるが, 
そちらは String->int というテーブルだったので 
int->int 用を作った, とのこと.

(コメントの追記では, 
template で型を一般化した Histogram クラスを作れば良かったのでは..., 
とも書かれているが...)


```
    ((cite: hotspot/src/share/vm/utilities/intHisto.hpp))
    // This class implements a simple histogram.
    
    // A histogram summarizes a series of "measurements", each of which is
    // assumed (required in this implementation) to have an outcome that is a
    // non-negative integer.  The histogram efficiently maps measurement outcomes
    // to the number of measurements had that outcome.
    
    // To print the results, invoke print() on your Histogram*.
    
    // Note: there is already an existing "Histogram" class, in file
    // histogram.{hpp,cpp}, but to my mind that's not a histogram, it's a table
    // mapping strings to counts.  To be a histogram (IMHO) it needs to map
    // numbers (in fact, integers) to number of occurrences of that number.
    
    // ysr: (i am not sure i agree with the above note.) i suspect we want to have a
    // histogram template that will map an arbitrary type (with a defined order
    // relation) to a count.
    
    
    class IntHistogram : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
IntHistogram::add_entry() で int 値を IntHistogram に入れていく
(こうすることで, どの値が何回 add_entry() されたか, というヒストグラム情報が内部に蓄えられていく).


```
    ((cite: hotspot/src/share/vm/utilities/intHisto.hpp))
      // Add a measurement with the given outcome to the sequence.
      void add_entry(int outcome);
```

結果を見たくなったら IntHistogram::print_on() を呼び出せばいい.
指定の outputStream に現在のヒストグラム情報が出力される.


```
    ((cite: hotspot/src/share/vm/utilities/intHisto.hpp))
      // Print the histogram on the given output stream.
      void print_on(outputStream* st) const;
```

#### インスタンスの格納場所(where its instances are stored)
現状では, 以下の大域変数に(のみ)格納されている. (なお, どちらも G1GC 関連の (より正確に言うと G1RemSet クラス関連の) 変数)

* card_repeat_count

  (ただし, この大域変数は CARD_REPEAT_HISTO が 0 以外の値に #define されていないと定義されない)

* out_of_histo

#### 情報の出力箇所(where the recorded information is output)
集めた結果は, print_statistics() 内で出力されている.

ただし, (デバッグ時であることに加えて) それぞれ CARD_REPEAT_HISTO マクロ定数, FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT マクロ定数が 0 以外の値に #define されていないと出力されない.

### 備考(Notes)
なお, デフォルトではどちらのマクロ定数も 0 に #define されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    #define CARD_REPEAT_HISTO 0
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.inline.hpp))
    #define FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT 0
```




### 詳細(Details)
See: [here](../doxygen/classIntHistogram.html) for details

---
