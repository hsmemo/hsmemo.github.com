---
layout: default
title: BufferingOopClosure クラス関連のクラス (BufferingOopClosure, BufferingOopsInGenClosure, BufferingOopsInHeapRegionClosure)
---
[Top](../index.html)

#### BufferingOopClosure クラス関連のクラス (BufferingOopClosure, BufferingOopsInGenClosure, BufferingOopsInHeapRegionClosure)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, OopClosure の機能を補うための補助クラス (root からの参照を辿る処理と実際の OopClosure の処理を分離するためのクラス).

### 概要(Summary)
これらのクラスは, GC 時の OopClosure による処理の際に, 
root からの参照を辿る処理と実際の OopClosure の処理の時間を分離するために使用される.

BufferingOopClosure はコンストラクタ引数として OopClosure を1つ受け取る.
BufferingOopClosure の処理が呼び出されると,
root から辿れるポインタを配列に格納していき,
配列がいっぱいになったらまとめて OopClosure に掛けて Closure 処理を行う
(ついでに, それぞれの処理時間も計測している模様).

なお, root の種類によって以下の3種類が用意されている.

  * BufferingOopClosure

    Java Heap 以外の root を処理する.

  * BufferingOopsInGenClosure

    Java Heap 内にある root を処理する.

  * BufferingOopsInHeapRegionClosure

    このクラスは使用箇所が見当たらないが... (Java Heap 内にある root を処理する??)
    

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/bufferingOopClosure.hpp))
    // A BufferingOops closure tries to separate out the cost of finding roots
    // from the cost of applying closures to them.  It maintains an array of
    // ref-containing locations.  Until the array is full, applying the closure
    // to an oop* merely records that location in the array.  Since this
    // closure app cost is small, an elapsed timer can approximately attribute
    // all of this cost to the cost of finding the roots.  When the array fills
    // up, the wrapped closure is applied to all elements, keeping track of
    // this elapsed time of this process, and leaving the array empty.
    // The caller must be sure to call "done" to process any unprocessed
    // buffered entriess.
```



### クラス一覧(class list)

  * [BufferingOopClosure](#no2aup0Chp)
  * [BufferingOopsInGenClosure](#noVH1VrQBO)
  * [BufferingOopsInHeapRegionClosure](#noo2YPKevq)


---
## <a name="no2aup0Chp" id="no2aup0Chp">BufferingOopClosure</a>

### 概要(Summary)
他の OopClosure と組み合わせて使用される Closure クラス.

G1GC の処理において, root からの参照を辿る処理と実際の OopClosure の処理の時間を分離するために使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/bufferingOopClosure.hpp))
    class BufferingOopClosure: public OopClosure {
```

### 使われ方(Usage)
G1CollectedHeap::g1_process_strong_roots() 内で(のみ)使用されている
(G1CollectedHeap::g1_process_strong_roots() 内の局所変数として直接使用されているほか,
その中で使用される BufferingOopsInGenClosure 内のフィールドとしても使用されている).  (See: [here](no2935YzN.html) for details)




### 詳細(Details)
See: [here](../doxygen/classBufferingOopClosure.html) for details

---
## <a name="noVH1VrQBO" id="noVH1VrQBO">BufferingOopsInGenClosure</a>

### 概要(Summary)
他の OopClosure と組み合わせて使用される Closure クラス.

G1GC の処理において, root からの参照を辿る処理と実際の OopClosure の処理の時間を分離するために使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/bufferingOopClosure.hpp))
    class BufferingOopsInGenClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
G1CollectedHeap::g1_process_strong_roots() 内で(のみ)使用されている (See: [here](no2935YzN.html) for details).




### 詳細(Details)
See: [here](../doxygen/classBufferingOopsInGenClosure.html) for details

---
## <a name="noo2YPKevq" id="noo2YPKevq">BufferingOopsInHeapRegionClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/bufferingOopClosure.hpp))
    class BufferingOopsInHeapRegionClosure: public OopsInHeapRegionClosure {
```

### 使われ方(Usage)
(現状では使われていない)




### 詳細(Details)
See: [here](../doxygen/classBufferingOopsInHeapRegionClosure.html) for details

---
