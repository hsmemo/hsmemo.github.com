---
layout: default
title: CardTableModRefBS クラス関連のクラス (CardTableModRefBS, CardTableModRefBSForCTRS)
---
[Top](../index.html)

#### CardTableModRefBS クラス関連のクラス (CardTableModRefBS, CardTableModRefBSForCTRS)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, BarrierSet の一種 (See: [here](no3718kvd.html) for details).

CollectedHeap 内のポインタフィールドについて, 変更された箇所を "card" という単位で記録する.

なおコメントによると, 
「変更されたポインタフィールドを含む card」を dirty 化するのではなく, 
「変更されたポインタフィールドを含むオブジェクトの先頭アドレスが含まれる card」を dirty 化する
(BlockOffsetArray から使用する場合にはこちらの方が使いやすい) ので, 
MemRegionClosures でスキャンする際には注意するように, 
とのこと.


```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.hpp))
    // This kind of "BarrierSet" allows a "CollectedHeap" to detect and
    // enumerate ref fields that have been modified (since the last
    // enumeration.)
    
    // As it currently stands, this barrier is *imprecise*: when a ref field in
    // an object "o" is modified, the card table entry for the card containing
    // the head of "o" is dirtied, not necessarily the card containing the
    // modified field itself.  For object arrays, however, the barrier *is*
    // precise; only the card containing the modified element is dirtied.
    // Any MemRegionClosures used to scan dirty cards should take these
    // considerations into account.
```


### クラス一覧(class list)

  * [CardTableModRefBS](#nondjm7o2K)
  * [CardTableModRefBSForCTRS](#noqBjj8OX5)


---
## <a name="nondjm7o2K" id="nondjm7o2K">CardTableModRefBS</a>

### 概要(Summary)
card 単位で記録を行う BarrierSet クラス (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.hpp))
    class CardTableModRefBS: public ModRefBarrierSet {
```

### 内部構造(Internal structure)
内部的には, jbyte の配列で "card table" を実装している.

(card 1枚に付き jbyte 値1つが対応.
 以下の _byte_map フィールドに jbyte 配列の先頭アドレスが格納され, _byte_map_size フィールドに配列の長さが格納されている)


```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.hpp))
      const size_t    _byte_map_size;    // in bytes
      jbyte*          _byte_map;         // the card marking array
```

アドレスに対応する card table のエントリ(のアドレス)を取得したい場合は, byte_for() メソッドを使えばいい.

```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.hpp))
      // Mapping from address to card marking array entry
      jbyte* byte_for(const void* p) const {
```

逆に, card table のエントリ(を指すアドレス)から対応するメモリアドレスを取得したい場合は, addr_for() メソッドを使えばいい.

```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.hpp))
      // Mapping from card marking array entry to address of first word
      HeapWord* addr_for(const jbyte* p) const {
```




### 詳細(Details)
See: [here](../doxygen/classCardTableModRefBS.html) for details

---
## <a name="noqBjj8OX5" id="noqBjj8OX5">CardTableModRefBSForCTRS</a>

### 概要(Summary)
CardTableModRefBS クラスの具象サブクラスの1つ
(= BarrierSet 機能を提供するクラス).


```
    ((cite: hotspot/src/share/vm/memory/cardTableModRefBS.hpp))
    // A specialization for the CardTableRS gen rem set.
    class CardTableModRefBSForCTRS: public CardTableModRefBS {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CardTableRS オブジェクトの _ct_bs フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/memory/cardTableRS.hpp))
    class CardTableRS: public GenRemSet {
    ...
      CardTableModRefBSForCTRS* _ct_bs;
```




### 詳細(Details)
See: [here](../doxygen/classCardTableModRefBSForCTRS.html) for details

---
