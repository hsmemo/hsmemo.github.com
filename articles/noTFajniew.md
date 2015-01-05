---
layout: default
title: CompactingPermGen クラス 
---
[Top](../index.html)

#### CompactingPermGen クラス 



---
## <a name="nor0fTo8Km" id="nor0fTo8Km">CompactingPermGen</a>

### 概要(Summary)
PermGen クラスの具象サブクラスの1つ (See: [here](no3718kvd.html) for details).

このクラスは, クラスデータ用の領域が連続領域になっているケース用 
(なお, 連続領域になっていない場合の具象サブクラスは CMSPermGen (See: CMSPermGen)).


```cpp
    ((cite: hotspot/src/share/vm/memory/compactPermGen.hpp))
    // A PermGen implemented with a contiguous space.
    class CompactingPermGen:  public PermGen {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PermanentGenerationSpec::init() 内で(のみ)生成されている.

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(G1CollectedHeap の初期化処理) (See: <a href="no2114tfN.html">here</a> for details)
-&gt; G1CollectedHeap::initialize()
   -&gt; PermanentGenerationSpec::init()

(GenCollectedHeap の初期化処理) (See: <a href="no2114gVH.html">here</a> for details)
-&gt; GenCollectedHeap::initialize()
   -&gt; PermanentGenerationSpec::init()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classCompactingPermGen.html) for details

---
