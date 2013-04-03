---
layout: default
title: PhaseStringOpts クラス (PhaseStringOpts, 及びその補助クラス(StringConcat))
---
[Top](../index.html)

#### PhaseStringOpts クラス (PhaseStringOpts, 及びその補助クラス(StringConcat))

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, 文字列操作(java.lang.Stringの操作)の最適化処理を表すクラス.


### クラス一覧(class list)

  * [PhaseStringOpts](#noDd1JraYU)
  * [StringConcat](#noSNmnezcG)


---
## <a name="noDd1JraYU" id="noDd1JraYU">PhaseStringOpts</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

文字列(java.lang.Stringオブジェクト)の作成処理を最適化する.

(より具体的に言うと, java.lang.StringBuffer.toString() と java.lang.StringBuilder.toString() の最適化を行う模様.
 これらのメソッドで文字列が作成されている場合に, まとめられる concatnate 操作を 1つにまとめて処理を高速化する)


```
    ((cite: hotspot/src/share/vm/opto/stringopts.hpp))
    class PhaseStringOpts : public Phase {
```

### 使われ方(Usage)
Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis ) 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseStringOpts.html) for details

---
## <a name="noSNmnezcG" id="noSNmnezcG">StringConcat</a>

### 概要(Summary)
PhaseStringOpts クラス内で使用される補助クラス(ResourceObjクラス).

PhaseStringOpts が最適化対象とするメソッド呼び出しを表すクラス.
(より具体的に言うと, java.lang.StringBuffer.toString() の呼び出し箇所, 
あるいは java.lang.StringBuilder.toString() の呼び出し箇所を表す).
1つの StringConcat オブジェクトが 1つの呼び出し箇所に対応する.


```
    ((cite: hotspot/src/share/vm/opto/stringopts.cpp))
    class StringConcat : public ResourceObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* PhaseStringOpts::build_candidate()
* StringConcat::merge()




### 詳細(Details)
See: [here](../doxygen/classStringConcat.html) for details

---
