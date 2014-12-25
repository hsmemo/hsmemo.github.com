---
layout: default
title: DirtyCardQueue クラス関連のクラス (CardTableEntryClosure, DirtyCardQueue, DirtyCardQueueSet)
---
[Top](../index.html)

#### DirtyCardQueue クラス関連のクラス (CardTableEntryClosure, DirtyCardQueue, DirtyCardQueueSet)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, Barrier Set の働きを補佐するためのクラス (= Remembered Set を適切に維持するためのクラス)
(See: [here](no2114EV0.html) and [here](no2935dGZ.html) for details).


### クラス一覧(class list)

  * [DirtyCardQueue](#no5X3QOOrp)
  * [DirtyCardQueueSet](#noSucKFZ9R)
  * [CardTableEntryClosure](#noO-s1FjVb)


---
## <a name="no5X3QOOrp" id="no5X3QOOrp">DirtyCardQueue</a>

### 概要(Summary)
Barrier Set の働きを補佐するための (= Remembered Set を適切に維持するための) PtrQueue クラス
(論文中では "remembered set log (RS log)").

Java プログラムによってポインタフィールドが変更された際には, 
Barrier Set が dirty 化されるとともに, 
このキューに「どこを dirty にしたか」という情報が記録される
(See: [here](no2114EV0.html) and [here](no2935dGZ.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.hpp))
    // A ptrQueue whose elements are "oops", pointers to object heads.
    class DirtyCardQueue: public PtrQueue {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
write barrier 処理の中でキューにポインタが追加される
(なお, 追加されるポインタは「変更箇所に対応する card のアドレス (= card table 中の位置)」).

キューが一杯になった場合は DirtyCardQueueSet::handle_zero_index_for_thread() が呼び出され, 
一杯になったキューが DirtyCardQueueSet に登録されるとともに, 
新しいキューが用意される.

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている
(ポインタ型ではないフィールドの一覧).

* 各 JavaThread オブジェクトの _dirty_card_queue フィールド

  write barrier 処理でポインタが記録されるフィールド
  (= そのスレッドが変更したポインタを格納するキュー).

* 各 DirtyCardQueueSet オブジェクトの _shared_dirty_card_queue フィールド

* 各 G1ParScanThreadState オブジェクトの _dcq フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, 後ろの 3つは局所変数としての生成なので一時的なオブジェクト).

* (JavaThread クラスの _dirty_card_queue フィールドは, ポインタ型ではなく実体なので,
  JavaThread オブジェクトの生成時に一緒に生成される)

* (DirtyCardQueueSet クラスの _shared_dirty_card_queue フィールドは, ポインタ型ではなく実体なので,
  DirtyCardQueueSet オブジェクトの生成時に一緒に生成される)

* (G1ParScanThreadState クラスの _dcq フィールドは, ポインタ型ではなく実体なので,
  G1ParScanThreadState オブジェクトの生成時に一緒に生成される)

* G1CollectedHeap::remove_self_forwarding_pointers() 内 (局所変数として生成)

* G1RemSet::oops_into_collection_set_do() 内 (局所変数として生成)

* G1RemSet::prepare_for_verify() 内 (局所変数として生成)




### 詳細(Details)
See: [here](../doxygen/classDirtyCardQueue.html) for details

---
## <a name="noSucKFZ9R" id="noSucKFZ9R">DirtyCardQueueSet</a>

### 概要(Summary)
DirtyCardQueue クラス用の PtrQueueSet クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.hpp))
    class DirtyCardQueueSet: public PtrQueueSet {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている
(ポインタ型ではないフィールドの一覧).

* JavaThread クラスの _dirty_card_queue_set フィールド (static フィールド)

  各 JavaThread オブジェクト内の ObjPtrQueue から共有されている)

* 各 G1CollectedHeap オブジェクトの _dirty_card_queue_set フィールド (「各」と言っても1つしかいないが...)
  
  処理を後で行う(遅延する)ポインタを格納しておくためのキュー
  (See: G1DeferredRSUpdate).

* 各 G1CollectedHeap オブジェクトの _into_cset_dirty_card_queue_set フィールド (「各」と言っても1つしかいないが...)
  
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* (JavaThread クラスの _dirty_card_queue_set フィールドは, ポインタ型ではなく実体なので,
  初期段階で自動的に生成される)

* (G1CollectedHeap クラスの _dirty_card_queue_set フィールドは, ポインタ型ではなく実体なので,
  G1CollectedHeap オブジェクトの生成時に一緒に生成される)

* (G1CollectedHeap クラスの _into_cset_dirty_card_queue_set フィールドは, ポインタ型ではなく実体なので,
  G1CollectedHeap オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classDirtyCardQueueSet.html) for details

---
## <a name="noO-s1FjVb" id="noO-s1FjVb">CardTableEntryClosure</a>

### 概要(Summary)
DirtyCardQueue クラス用の補助クラス.

DirtyCardQueue 内の card に対して何らかの処理を行う Closure クラスの基底クラス
(より正確に言うと, 「card のアドレス (= card table 内を指すポインタ)」に対して何らかの処理を行う Closure).
なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

(なお, "Closure" という名前だが StackObj ではなく CHeapObj クラスになっている.
このため, スタック以外の箇所に確保する使い方も可能)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.hpp))
    // A closure class for processing card table entries.  Note that we don't
    // require these closure objects to be stack-allocated.
    class CardTableEntryClosure: public CHeapObj {
```

### 使われ方(Usage)
使用する際には, do_card_ptr() メソッドをオーバーライドしたサブクラスを作ればいい.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.hpp))
      // Process the card whose card table entry is "card_ptr".  If returns
      // "false", terminate the iteration early.
      virtual bool do_card_ptr(jbyte* card_ptr, int worker_i = 0) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classCardTableEntryClosure.html) for details

---
