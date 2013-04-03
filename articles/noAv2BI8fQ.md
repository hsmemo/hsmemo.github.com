---
layout: default
title: CardTableRS クラス関連のクラス (CardTableRS, ClearNoncleanCardWrapper, 及びそれらの補助クラス(VerifyCleanCardClosure, VerifyCTSpaceClosure, VerifyCTGenClosure))
---
[Top](../index.html)

#### CardTableRS クラス関連のクラス (CardTableRS, ClearNoncleanCardWrapper, 及びそれらの補助クラス(VerifyCleanCardClosure, VerifyCTSpaceClosure, VerifyCTGenClosure))

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, Remembered Set の一種 (See: [here](no3718kvd.html) for details).

Remembered Set 機能を提供するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavengeHeap 以外の場合に使用される (See: CardTableExtension).

(<= ただし, G1CollectedHeap の場合も主に別のクラスが Remembered Set 機能を担当するので, 
実質的には GenCollectedHeap でしか使われていない模様 #TODO) (See: HeapRegionRemSet, G1RemSet)


### クラス一覧(class list)

  * [CardTableRS](#nobU8pPfUn)
  * [ClearNoncleanCardWrapper](#nodbhl-bJZ)
  * [VerifyCleanCardClosure](#noeBcxVFS2)
  * [VerifyCTSpaceClosure](#noMocKf_jU)
  * [VerifyCTGenClosure](#noXZfoG0rV)


---
## <a name="nobU8pPfUn" id="nobU8pPfUn">CardTableRS</a>

### 概要(Summary)
GenRemSet クラスの具象サブクラス
(= Remembered Set 機能を提供するクラス).

Remembered Set 機能を提供するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavengeHeap 以外の場合に使用される (See: CardTableExtension).

なお, 名前の由来は内部的に "card table" を使用していることから.
(<= 残りの RS は remembered set の略(だと思われる))


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.hpp))
    // This kind of "GenRemSet" uses a card table both as shared data structure
    // for a mod ref barrier set and for the rem set information.
    
    class CardTableRS: public GenRemSet {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 SharedHeap オブジェクトの _rem_set フィールドに(のみ)格納されている.

(実際には SharedHeap は abstract class なので, 
GenCollectedHeap::_rem_set, または G1CollectedHeap::_rem_set に格納されている)

#### 生成箇所(where its instances are created)
CollectorPolicy::create_rem_set() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* G1CollectedHeap オブジェクトの生成処理

  G1CollectedHeap::initialize()
  -> CollectorPolicy::create_rem_set()

* GenCollectedHeap オブジェクトの生成処理

  GenCollectedHeap::initialize()
  -> CollectorPolicy::create_rem_set()
```

### 内部構造(Internal structure)
内部的には, barrier set (CardTableModRefBSForCTRS オブジェクト) を用いて remembered set を実現している.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.hpp))
      CardTableModRefBSForCTRS* _ct_bs;
```

使用する CardTableModRefBSForCTRS オブジェクト内の card table には, ポインタの有無に応じて以下の値が書き込まれる.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.hpp))
      enum ExtendedCardValue {
        youngergen_card   = CardTableModRefBS::CT_MR_BS_last_reserved + 1,
        // These are for parallel collection.
        // There are three P (parallel) youngergen card values.  In general, this
        // needs to be more than the number of generations (including the perm
        // gen) that might have younger_refs_do invoked on them separately.  So
        // if we add more gens, we have to add more values.
        youngergenP1_card  = CardTableModRefBS::CT_MR_BS_last_reserved + 2,
        youngergenP2_card  = CardTableModRefBS::CT_MR_BS_last_reserved + 3,
        youngergenP3_card  = CardTableModRefBS::CT_MR_BS_last_reserved + 4,
        cur_youngergen_and_prev_nonclean_card =
          CardTableModRefBS::CT_MR_BS_last_reserved + 5
      };
```

### 備考(Notes)
通常の write barrier 時には直接 barrier set が dirty 化されているが, 
CardTableRS::inline_write_ref_field_gc() を呼べばこのクラス経由で dirty にすることもできる
(card table の対応する箇所を youngergen_card に変更する処理が行われる).


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.hpp))
      void inline_write_ref_field_gc(void* field, oop new_val) {
        jbyte* byte = _ct_bs->byte_for(field);
        *byte = youngergen_card;
      }
```




### 詳細(Details)
See: [here](../doxygen/classCardTableRS.html) for details

---
## <a name="nodbhl-bJZ" id="nodbhl-bJZ">ClearNoncleanCardWrapper</a>

### 概要(Summary)
CardTableModRefBS クラス内で使用される補助クラス (Closure クラス).

DirtyCardToOopClosure クラスとセットで使用するクラス.
指定したメモリ範囲内の dirty 箇所について指定の DirtyCardToOopClosure オブジェクトを適用する.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.hpp))
    class ClearNoncleanCardWrapper: public MemRegionClosure {
```

### 使われ方(Usage)
以下の関数内で(のみ)使われている.

* CardTableModRefBS::non_clean_card_iterate_possibly_parallel()
* CardTableModRefBS::process_stride()


```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.cpp))
    void CardTableModRefBS::non_clean_card_iterate_possibly_parallel(Space* sp,
                                                                     MemRegion mr,
                                                                     OopsInGenClosure* cl,
                                                                     CardTableRS* ct) {
    ...
          DirtyCardToOopClosure* dcto_cl = sp->new_dcto_cl(cl, precision(),
                                                           cl->gen_boundary());
          ClearNoncleanCardWrapper clear_cl(dcto_cl, ct);
    
          clear_cl.do_MemRegion(mr);
```

### 内部構造(Internal structure)
コンストラクタで DirtyCardToOopClosure オブジェクトを受け取る.

実際に仕事を行うメソッドは ClearNoncleanCardWrapper::do_MemRegion().
引数で指定されたメモリ範囲に対応する card を調べていき,
dirty な領域があれば DirtyCardToOopClosure オブジェクトを呼び出して処理を行う.

(なお, DirtyCardToOopClosure オブジェクト自体も, 引数で渡された OopsInGenClosure を呼び出すだけだったりする.
 (See: DirtyCardToOopClosure))

#### 参考(for your information): ClearNoncleanCardWrapper::do_MemRegion()
See: [here](no2114EqQ.html) for details



### 詳細(Details)
See: [here](../doxygen/classClearNoncleanCardWrapper.html) for details

---
## <a name="noeBcxVFS2" id="noeBcxVFS2">VerifyCleanCardClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.cpp))
    class VerifyCleanCardClosure: public OopClosure {
```

### 使われ方(Usage)
CardTableRS::verify() 内でのみ使用される.

(正確には, CardTableRS::verify() 内で使用される VerifyCTSpaceClosure から呼び出される補助クラス.
 VerifyCTSpaceClosure が呼び出す CardTableRS::verify_space() 内で使用される.)



### 詳細(Details)
See: [here](../doxygen/classVerifyCleanCardClosure.html) for details

---
## <a name="noMocKf_jU" id="noMocKf_jU">VerifyCTSpaceClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.cpp))
    class VerifyCTSpaceClosure: public SpaceClosure {
```

### 使われ方(Usage)
CardTableRS::verify() 内でのみ使用される.

(CardTableRS::verify() 内で直接使用されているほか,
 その中で呼び出される VerifyCTGenClosure::do_generation() の中でも使用されている.)



### 詳細(Details)
See: [here](../doxygen/classVerifyCTSpaceClosure.html) for details

---
## <a name="noXZfoG0rV" id="noXZfoG0rV">VerifyCTGenClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.cpp))
    class VerifyCTGenClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
CardTableRS::verify() 内でのみ使用される.




### 詳細(Details)
See: [here](../doxygen/classVerifyCTGenClosure.html) for details

---
