---
layout: default
title: LiveRange クラス 
---
[Top](../index.html)

#### LiveRange クラス 



---
## <a name="no6YSF0VOW" id="no6YSF0VOW">LiveRange</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

GC 処理を高速化するための補助クラス. phase 2 から phase 4 の処理で使用される

より具体的に言うと, コンパクション処理の際に死んでいるオブジェクト(dead object)をスキップできるようにするクラス.
これにより毎回領域全体を見る必要がなくなり処理が高速化される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/liveRange.hpp))
    // This is a shared helper class used during phase 3 and 4 to move all the objects
    // Dead regions in a Space are linked together to keep track of the live regions
    // so that the live data can be traversed quickly without having to look at each
    // object.
    
    class LiveRange: public MemRegion {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
Mark Sweep Compact 型の phase 2 から phase 4 に対応する以下のマクロ／関数の中で使用されている.

* phase 2 : 

  SCAN_AND_FORWARD マクロ 

  (ParallelScavenge の場合は PSMarkSweepDecorator::precompact())

* phase 3 : 

  SCAN_AND_ADJUST_POINTERS マクロ 

  (ParallelScavenge の場合は PSMarkSweepDecorator::adjust_pointers())

* phase 4 : 

  SCAN_AND_COMPACT マクロ

  (ParallelScavenge の場合は PSMarkSweepDecorator::compact())


(なお, phase 3 と phase 4 ではソースコード中に LiveRange というシンボルは出てこないが実際には使用されている)

#### 使用方法の概要(how to use)
1. phase2 の段階で, live object と dead object の境目に当たる箇所全てに LiveRange オブジェクトを埋め込んでおく.

   (より正確に言うと, live object の「後ろ」に dead object が来る箇所全てに埋め込む)

   (具体的な処理としては, 「live object が1つ以上連続しその後ろに dead object が存在する」という箇所全てについて,
   その dead object の先頭部分を LiveRange オブジェクトで上書きする)

2. そして, phase2 の段階で, 埋め込んだ LiveRange に以下の 2つの情報を書き込んでおく.

   1. 次の live object の位置
   2. その次の dead object の位置次の live object の位置   (<= こちらは何に使われる?? 使われていないような... #TODO)

3. phase 3 や phase 4 の処理では, 埋め込んだ LiveRange を使用して dead object をスキップする.

   先頭から処理していって dead object に到達した場合,
   そこに埋め込まれた LiveRange 内の "1." によって次の live object が分かる
   (これにより, dead object をスキップして処理時間を短縮できる).




### 詳細(Details)
See: [here](../doxygen/classLiveRange.html) for details

---
