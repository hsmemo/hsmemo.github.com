---
layout: default
title: SimpleThresholdPolicy クラス 
---
[Top](../index.html)

#### SimpleThresholdPolicy クラス 



---
## <a name="noDHrZ_a3O" id="noDHrZ_a3O">SimpleThresholdPolicy</a>

### 概要(Summary)
C1/C2 JIT Compiler 用の補助クラス (#ifdef TIERED 時にしか定義されない).

Tiered Compilation 用の CompilationPolicy クラス(= メソッドをコンパイルするかどうかの条件/閾値を管理するクラス)
(See: [here](noi9gh3rMo.html) and [here](no3420O-k.html) for details).

なお, このクラスは CompilationPolicyChoice オプションが 2 の場合にのみ使用される
(See: compilationPolicy_init(), Arguments::set_tiered_flags()).


```cpp
    ((cite: hotspot/src/share/vm/runtime/simpleThresholdPolicy.hpp))
    class SimpleThresholdPolicy : public CompilationPolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
CompilationPolicy クラスの _policy フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
compilationPolicy_init() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; compilationPolicy_init()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classSimpleThresholdPolicy.html) for details

---
