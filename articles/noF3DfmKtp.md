---
layout: default
title: TenuredGeneration クラス 
---
[Top](../index.html)

#### TenuredGeneration クラス 



---
## <a name="no6mr9d4gm" id="no6mr9d4gm">TenuredGeneration</a>

### 概要(Summary)
GenCollectedHeap 使用時において, Old Generation の管理を担当するクラスの 1つ (See: [here](no3718kvd.html) for details).

このクラスは, GC アルゴリズムが CMS ではない場合用 (つまり Serial Old GC 用) (See: ConcurrentMarkSweepGeneration).


```cpp
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.hpp))
    // TenuredGeneration models the heap containing old (promoted/tenured) objects.
```


```cpp
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.hpp))
    class TenuredGeneration: public OneContigSpaceCardGeneration {
```


### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 GenCollectedHeap オブジェクトの _gens フィールドに(のみ)格納されている.

(正確には, このフィールドは Generation のポインタの配列を格納するフィールド.
この中に, その GenCollectedHeap 内で使用される全ての Generation オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
GenerationSpec::init() 内で(のみ)生成されている.

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(略) (See: <a href="no2114gVH.html">here</a> for details)
-&gt; GenCollectedHeap::initialize()
   -&gt; GenerationSpec::init()
</pre></div>

#### 使用箇所(where its instances are used)
...(#TODO)

このクラスの TenuredGeneration::collect() メソッドが Serial Old GC 処理のエントリポイントになっている (See: [here](no2114hPa.html) for details).




### 詳細(Details)
See: [here](../doxygen/classTenuredGeneration.html) for details

---
