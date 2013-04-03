---
layout: default
title: NumberSeq クラス関連のクラス (AbsSeq, NumberSeq, TruncatedSeq)
---
[Top](../index.html)

#### NumberSeq クラス関連のクラス (AbsSeq, NumberSeq, TruncatedSeq)

これらは, 収集した数値データを基に, それらの最大値, 平均値, 分散といった統計情報を計算するためのユーティリティ・クラス.


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
    /**
     **  This file contains a few classes that represent number sequence,
     **  x1, x2, x3, ..., xN, and can calculate their avg, max, and sd.
     **
     **  Here's a quick description of the classes:
     **
     **    AbsSeq: abstract superclass
     **    NumberSeq: the sequence is assumed to be very long and the
     **      maximum, avg, sd, davg, and dsd are calculated over all its elements
     **    TruncatedSeq: this class keeps track of the last L elements
     **      of the sequence and calculates avg, max, and sd only over them
     **/
```


### クラス一覧(class list)

  * [AbsSeq](#noh7sIGGFo)
  * [NumberSeq](#noTZqARRXw)
  * [TruncatedSeq](#no0H_nOvZF)


---
## <a name="noh7sIGGFo" id="noh7sIGGFo">AbsSeq</a>

全ての NumberSeq クラスの基底クラス.


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
    class AbsSeq {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 収集した数値データを add() メソッドでどんどん追加していけばよい.


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
      virtual void add(double val); // adds a new element to the sequence
```

2. 統計情報が知りたくなったら以下のメソッドを呼ぶ. (返値として最大値, 平均値, 分散といった統計情報が返される)


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
      virtual double maximum() const = 0; // maximum element in the sequence
      virtual double last() const = 0; // last element added in the sequence
    
      // the number of elements in the sequence
      int num() const { return _num; }
      // the sum of the elements in the sequence
      double sum() const { return _sum; }
    
      double avg() const; // the average of the sequence
      double variance() const; // the variance of the sequence
      double sd() const; // the standard deviation of the sequence
    
      double davg() const; // decaying average
      double dvariance() const; // decaying variance
      double dsd() const; // decaying "standard deviation"
```




### 詳細(Details)
See: [here](../doxygen/classAbsSeq.html) for details

---
## <a name="noTZqARRXw" id="noTZqARRXw">NumberSeq</a>

### 概要(Summary)
AbsSeq クラスの具象サブクラスの1つ.


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
    class NumberSeq: public AbsSeq {
```

### 使われ方(Usage)
現在は G1GC 関連のクラス内で(のみ)使用されている模様.

(主にソフトリアルタイム制約を満たすための予測処理等で使われている)




### 詳細(Details)
See: [here](../doxygen/classNumberSeq.html) for details

---
## <a name="no0H_nOvZF" id="no0H_nOvZF">TruncatedSeq</a>

### 概要(Summary)
特殊な NumberSeq クラス.

add() したデータのうち, 最後の L 個以外を無視した統計情報を返す.


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
    class TruncatedSeq: public AbsSeq {
```

(デフォルトでは最後の 10 個分だけを使用)


```
    ((cite: hotspot/src/share/vm/utilities/numberSeq.hpp))
      enum PrivateConstants {
        DefaultSeqLength = 10
      };
```

### 使われ方(Usage)
現在は G1GC 関連のクラス内で(のみ)使用されている模様.

(主にソフトリアルタイム制約を満たすための予測処理等で使われている)




### 詳細(Details)
See: [here](../doxygen/classTruncatedSeq.html) for details

---
