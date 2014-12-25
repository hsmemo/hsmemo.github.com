---
layout: default
title: G1SATBCardTableModRefBS クラス関連のクラス (G1SATBCardTableModRefBS, G1SATBCardTableLoggingModRefBS)
---
[Top](../index.html)

#### G1SATBCardTableModRefBS クラス関連のクラス (G1SATBCardTableModRefBS, G1SATBCardTableLoggingModRefBS)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, BarrierSet の一種 (See: [here](no3718kvd.html) for details).

なお, これらの Barrier Set を使用する場合, 
write barrier 処理も元の値を退避しておく処理に変わる
(G1GC の Concurrent Mark (snapshot-at-the-beginning 式) がきちんと動くようにするため)
(See: [here](no2114EV0.html) for details).

なお, これらのクラスは以下のような継承関係を持つ
(実際に使用されるのは G1SATBCardTableLoggingModRefBS クラス).

  * CardTableModRefBSForCTRS (<= これは別ファイルで定義されているクラス)
      * G1SATBCardTableModRefBS (abstract class)
          * G1SATBCardTableLoggingModRefBS


### クラス一覧(class list)

  * [G1SATBCardTableModRefBS](#noyg0IiUXT)
  * [G1SATBCardTableLoggingModRefBS](#nombI7ln7T)


---
## <a name="noyg0IiUXT" id="noyg0IiUXT">G1SATBCardTableModRefBS</a>

### 概要(Summary)
G1GC 用の Barrier Set の基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.hpp))
    // This barrier is specialized to use a logging barrier to support
    // snapshot-at-the-beginning marking.
    
    class G1SATBCardTableModRefBS: public CardTableModRefBSForCTRS {
```

### 内部構造(Internal structure)
コンストラクタは (_kind を G1SATBCT にしている点以外は) CardTableModRefBSForCTRS と同じ.

#### 参考(for your information): G1SATBCardTableModRefBS::G1SATBCardTableModRefBS()
See: [here](no3420vNv.html) for details
### 備考(Notes)
G1SATBCardTableModRefBS::enqueue() メソッドは, 
ポインタ変更時の write barrier 処理を行う.

ただし, これは jni_GetObjectField() や sun.misc.Unsafe から使用されるもので, 
通常の write barrier 処理では使用されない (See: [here](no2114EV0.html) for details).




### 詳細(Details)
See: [here](../doxygen/classG1SATBCardTableModRefBS.html) for details

---
## <a name="nombI7ln7T" id="nombI7ln7T">G1SATBCardTableLoggingModRefBS</a>

### 概要(Summary)
G1SATBCardTableModRefBS クラスの具象サブクラス.
実際に Barrier Set として使用されるのはこのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.hpp))
    // Adds card-table logging to the post-barrier.
    // Usual invariant: all dirty cards are logged in the DirtyCardQueueSet.
    class G1SATBCardTableLoggingModRefBS: public G1SATBCardTableModRefBS {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CardTableRS オブジェクトの _ct_bs フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
CardTableRS::CardTableRS() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(CardTableModRefBS の内部構造も参照)

コンストラクタは (_kind を G1SATBCT にしている点以外は) CardTableModRefBSForCTRS と同じ.

#### 参考(for your information): G1SATBCardTableLoggingModRefBS::G1SATBCardTableLoggingModRefBS()
See: [here](no34208X1.html) for details



### 詳細(Details)
See: [here](../doxygen/classG1SATBCardTableLoggingModRefBS.html) for details

---
