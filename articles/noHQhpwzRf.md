---
layout: default
title: Rewriter クラス 
---
[Top](../index.html)

#### Rewriter クラス 



---
## <a name="noDWVZURae" id="noDWVZURae">Rewriter</a>

### 概要(Summary)
クラスのリンク時における rewrite 処理で使用される一時オブジェクト(StackObjクラス). 
実際の rewrite 処理を行うクラス (See: [here](no3059AfB.html) for details).

(なお rewrite 処理とは, 高速化などのために bytecode を HotSpot 独自 bytecode へと変換する, あるいは 
bytecode 中の constant pool index を constant pool cache index へと書き換える処理).


```
    ((cite: hotspot/src/share/vm/interpreter/rewriter.hpp))
    // The Rewriter adds caches to the constant pool and rewrites bytecode indices
    // pointing into the constant pool for better interpreter performance.
    
    class Rewriter: public StackObj {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* Rewriter::rewrite(instanceKlassHandle klass, TRAPS)
* Rewriter::rewrite(instanceKlassHandle klass, constantPoolHandle cpool, objArrayHandle methods, TRAPS)

### 内部構造(Internal structure)
このクラスの Rewriter::rewrite() メソッドが rewrite 処理のエントリポイントになっている.

実際の処理のほとんどは Rewriter オブジェクトのコンストラクタ内で行われる.




### 詳細(Details)
See: [here](../doxygen/classRewriter.html) for details

---
