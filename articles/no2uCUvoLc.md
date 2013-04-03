---
layout: default
title: CardTableExtension クラス (CardTableExtension, 及びその補助クラス(CheckForUnmarkedOops, CheckForUnmarkedObjects, CheckForPreciseMarks))
---
[Top](../index.html)

#### CardTableExtension クラス (CardTableExtension, 及びその補助クラス(CheckForUnmarkedOops, CheckForUnmarkedObjects, CheckForPreciseMarks))

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, BarrierSet の一種 (なお Remembered Set の機能も兼ねる) (See: [here](no3718kvd.html) for details).


### クラス一覧(class list)

  * [CardTableExtension](#noAz9WC9zA)
  * [CheckForUnmarkedObjects](#noH0JL2Fv-)
  * [CheckForUnmarkedOops](#noohAX281m)
  * [CheckForPreciseMarks](#noHZnVS40r)


---
## <a name="noAz9WC9zA" id="noAz9WC9zA">CardTableExtension</a>

### 概要(Summary)
ParallelScavengeHeap 用の BarrierSet クラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.hpp))
    class CardTableExtension : public CardTableModRefBS {
```

### 使われ方(Usage)
card table には, ポインタの有無に応じて以下のような値が書き込まれる.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.hpp))
      enum ExtendedCardValue {
        youngergen_card   = CardTableModRefBS::CT_MR_BS_last_reserved + 1,
        verify_card       = CardTableModRefBS::CT_MR_BS_last_reserved + 5
      };
```

例えば New 領域を指しているポインタを見つけた場合は, inline_write_ref_field_gc() を呼ぶことで
card table の対応する箇所を youngergen_card に変更できる.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.hpp))
      // Card marking
      void inline_write_ref_field_gc(void* field, oop new_val) {
        jbyte* byte = byte_for(field);
        *byte = youngergen_card;
      }
```

また, card table の値は以下のメソッドで調べることが可能.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.hpp))
      // Testers for entries
      static bool card_is_dirty(int value)      { return value == dirty_card; }
      static bool card_is_newgen(int value)     { return value == youngergen_card; }
      static bool card_is_clean(int value)      { return value == clean_card; }
      static bool card_is_verify(int value)     { return value == verify_card; }
```




### 詳細(Details)
See: [here](../doxygen/classCardTableExtension.html) for details

---
## <a name="noH0JL2Fv-" id="noH0JL2Fv-">CheckForUnmarkedObjects</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される) 
(See: VerifyRememberedSets).

Remembered Set がおかしな状態になっていない(old->young のポインタの見落としがない)ことを確認する Closure.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.cpp))
    // Checks all objects for the existance of some type of mark,
    // precise or imprecise, dirty or newgen.
    class CheckForUnmarkedObjects : public ObjectClosure {
```

### 使われ方(Usage)
CardTableExtension::verify_all_young_refs_imprecise() 内で(のみ)使用されている.

(なお, CardTableExtension::verify_all_young_refs_imprecise() は
 PSScavenge::invoke_no_policy() 内で使用される verify 用の関数.
 ただし, VerifyRememberedSets オプションが指定されている場合にしか呼び出されない.
 (See: PSScavenge::invoke_no_policy()))




### 詳細(Details)
See: [here](../doxygen/classCheckForUnmarkedObjects.html) for details

---
## <a name="noohAX281m" id="noohAX281m">CheckForUnmarkedOops</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される) 
(See: VerifyRememberedSets).

(同じくトラブルシューティング用のクラスである) CheckForUnmarkedObjects クラス内で使用される補助クラス
(See: CheckForUnmarkedObjects).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.cpp))
    // Checks an individual oop for missing precise marks. Mark
    // may be either dirty or newgen.
    class CheckForUnmarkedOops : public OopClosure {
```

### 使われ方(Usage)
CheckForUnmarkedObjects::do_object() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCheckForUnmarkedOops.html) for details

---
## <a name="noHZnVS40r" id="noHZnVS40r">CheckForPreciseMarks</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される) 
(See: VerifyRememberedSets).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.cpp))
    // Checks for precise marking of oops as newgen.
    class CheckForPreciseMarks : public OopClosure {
```

### 使われ方(Usage)
CardTableExtension::verify_all_young_refs_precise() 内で(のみ)使用されている.

(なお, CardTableExtension::verify_all_young_refs_precise() 自体は, 現状はどこからも使用されていない.
 PSScavenge::invoke_no_policy() 内に呼び出しコードがあるがコメントアウトされている.
 PSScavenge::invoke_no_policy() 内の該当箇所では,
 「現状では false positives が出るので直るまで CardTableExtension::verify_all_young_refs_imprecise() を使う」
 と書かれている.
 (See: PSScavenge::invoke_no_policy()))




### 詳細(Details)
See: [here](../doxygen/classCheckForPreciseMarks.html) for details

---
