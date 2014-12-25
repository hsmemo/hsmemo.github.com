---
layout: default
title: SATBMarkQueueSet クラス関連のクラス (ObjPtrQueue, SATBMarkQueueSet)
---
[Top](../index.html)

#### SATBMarkQueueSet クラス関連のクラス (ObjPtrQueue, SATBMarkQueueSet)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, write barrier 処理を補佐するためのクラス
(Snapshot-at-the-Beginning 方式の write barrier では変更前のポインタ値をどこかに記録しておく必要があるので, 
 それを実現するためのクラス)
(See: [here](no2114EV0.html) and [here](no2935d4w.html) for details).


### クラス一覧(class list)

  * [ObjPtrQueue](#nokm1W0HbV)
  * [SATBMarkQueueSet](#noujQRebDG)


---
## <a name="nokm1W0HbV" id="nokm1W0HbV">ObjPtrQueue</a>

### 概要(Summary)
write barrier の処理を補佐するための PtrQueue クラス
(論文中では "current marking buffer").

Concurrent Marking Thread の動作中に Java プログラムによってポインタフィールドが変更された際には, 
そのフィールドの変更前の値がこのキューに記録される
(See: [here](no2114EV0.html) and [here](no2935d4w.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/satbQueue.hpp))
    // A ptrQueue whose elements are "oops", pointers to object heads.
    class ObjPtrQueue: public PtrQueue {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
write barrier 処理の中でキューにポインタが追加される.
キューが一杯になった場合は SATBMarkQueueSet::handle_zero_index_for_thread() が呼び出され, 
一杯になったキューが SATBMarkQueueSet に登録されるとともに, 
新しいキューが用意される.

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 JavaThread オブジェクトの _satb_mark_queue フィールド

  write barrier 処理でポインタが記録されるフィールド
  (= そのスレッドが変更したポインタを格納するキュー).

* 各 SATBMarkQueueSet オブジェクトの _shared_satb_queue フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* (JavaThread クラスの _satb_mark_queue フィールドは, ポインタ型ではなく実体なので,
  JavaThread オブジェクトの生成時に一緒に生成される)

* (SATBMarkQueueSet クラスの _shared_satb_queue フィールドは, ポインタ型ではなく実体なので,
  SATBMarkQueueSet オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classObjPtrQueue.html) for details

---
## <a name="noujQRebDG" id="noujQRebDG">SATBMarkQueueSet</a>

### 概要(Summary)
ObjPtrQueue クラス用の PtrQueueSet クラス.

Concurrent Marking 処理中に変更されたポインタフィールドの値を記録するためのクラス.
ここに蓄積されたポインタ情報は Concurrent Marking の調査対象になる (See: [here](no2935d4w.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/satbQueue.hpp))
    class SATBMarkQueueSet: public PtrQueueSet {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
JavaThread クラスの _satb_mark_queue_set フィールド (static フィールド) に(のみ)格納されている
(これが, 各 JavaThread オブジェクト内の ObjPtrQueue から共有される).

#### 生成箇所(where its instances are created)
(JavaThread クラスの _satb_mark_queue_set フィールドは, ポインタ型ではなく実体なので,
 初期段階で自動的に生成される)




### 詳細(Details)
See: [here](../doxygen/classSATBMarkQueueSet.html) for details

---
