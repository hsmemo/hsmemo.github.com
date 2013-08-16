---
layout: default
title: MethodLiveness クラス関連のクラス (MethodLivenessResult, MethodLiveness, MethodLiveness::BasicBlock, 及びそれらの補助クラス(BitCounter))
---
[Top](../index.html)

#### MethodLiveness クラス関連のクラス (MethodLivenessResult, MethodLiveness, MethodLiveness::BasicBlock, 及びそれらの補助クラス(BitCounter))

これらは, JIT コンパイラ用の liveness analysis 機能を実装したクラス (See: [here](no7882MiN.html) for details).

 * MethodLiveness は, liveness analysis を実行するクラス.

 * MethodLivenessResult は, MethodLiveness による解析結果を格納するためのクラス.

 * BitCounter は, デバッグ用(開発時用)の補助クラス.

(#TODO  MethodLiveness と PhaseLive と buildOopMap でやっている liveness analysis はそれぞれどう違う？)


### クラス一覧(class list)

  * [MethodLiveness](#noBL6gC-Uw)
  * [MethodLiveness::BasicBlock](#noEGNEqT3B)
  * [MethodLivenessResult](#nosCKyrIZY)
  * [BitCounter](#noL1ZpMy5u)


---
## <a name="noBL6gC-Uw" id="noBL6gC-Uw">MethodLiveness</a>

### 概要(Summary)
ciMethod クラス内で使用される補助クラス.

liveness analysis 処理中に使用される一時オブジェクト(ResourceObjクラス).
実際の liveness analysis 処理を行う.


```
    ((cite: hotspot/src/share/vm/compiler/methodLiveness.hpp))
    class MethodLiveness : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciMethod オブジェクトの _liveness フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciMethod::raw_liveness_at_bci()
* ciMethod::bci_block_start() (C1 JIT Compiler の場合のみ)

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ciMethod::raw_liveness_at_bci()
* ciMethod::bci_block_start() (C1 JIT Compiler の場合のみ)


```
    ((cite: hotspot/src/share/vm/ci/ciMethod.hpp))
      // Returns a bitmap indicating which locals are required to be
      // maintained as live for deopt.  raw_liveness_at_bci is always the
      // direct output of the liveness computation while liveness_at_bci
      // may mark all locals as live to improve support for debugging Java
      // code by maintaining the state of as many locals as possible.
      MethodLivenessResult raw_liveness_at_bci(int bci);
      MethodLivenessResult liveness_at_bci(int bci);
```

### 内部構造(Internal structure)
MethodLiveness はメソッド内の各バイトコード(bci = bytecode index)における「live な local 変数の集合」を調べる.

結果は MethodLivenessResult という bitmap で返される.

処理としては, メソッドを basic block に分割し, 
後ろから前に def/use 情報を伝搬させていって fixed point に達するまで繰り返すだけ
(ただし, 例外でのコントロールフローも考慮する必要がある).

以上の処理で, 各 basic block 単位の liveness 情報が分かる.
特定の bci での liveness 情報については, 問い合わせがあった際に, 
その bci を含む basic block の liveness 情報に対して, 
その basic block 内での def/use 情報を足し込んで算出している模様.

なお, 以下の理由から, 多少不正確な(保守的な)結果となっている.

  * jsr/ret については保守的に解析している.
  * 例外のフローについては, 個別の命令単位ではなく basic block 単位でまとめてみている.


```
    ((cite: hotspot/src/share/vm/compiler/methodLiveness.cpp))
    // The MethodLiveness class performs a simple liveness analysis on a method
    // in order to decide which locals are live (that is, will be used again) at
    // a particular bytecode index (bci).
    //
    // The algorithm goes:
    //
    // 1. Break the method into a set of basic blocks.  For each basic block we
    //    also keep track of its set of predecessors through normal control flow
    //    and predecessors through exceptional control flow.
    //
    // 2. For each basic block, compute two sets, gen (the set of values used before
    //    they are defined) and kill (the set of values defined before they are used)
    //    in the basic block.  A basic block "needs" the locals in its gen set to
    //    perform its computation.  A basic block "provides" values for the locals in
    //    its kill set, allowing a need from a successor to be ignored.
    //
    // 3. Liveness information (the set of locals which are needed) is pushed backwards through
    //    the program, from blocks to their predecessors.  We compute and store liveness
    //    information for the normal/exceptional exit paths for each basic block.  When
    //    this process reaches a fixed point, we are done.
    //
    // 4. When we are asked about the liveness at a particular bci with a basic block, we
    //    compute gen/kill sets which represent execution from that bci to the exit of
    //    its blocks.  We then compose this range gen/kill information with the normal
    //    and exceptional exit information for the block to produce liveness information
    //    at that bci.
    //
    // The algorithm is approximate in many respects.  Notably:
    //
    // 1. We do not do the analysis necessary to match jsr's with the appropriate ret.
    //    Instead we make the conservative assumption that any ret can return to any
    //    jsr return site.
    // 2. Instead of computing the effects of exceptions at every instruction, we
    //    summarize the effects of all exceptional continuations from the block as
    //    a single set (_exception_exit), losing some information but simplifying the
    //    analysis.
```




### 詳細(Details)
See: [here](../doxygen/classMethodLiveness.html) for details

---
## <a name="noEGNEqT3B" id="noEGNEqT3B">MethodLiveness::BasicBlock</a>

### 概要(Summary)
MethodLiveness クラス内で使用される補助クラス.

基本ブロック(basic block)に関する情報を表す.
1つの MethodLiveness::BasicBlock オブジェクトが 1つの基本ブロックに対応する.


```
    ((cite: hotspot/src/share/vm/compiler/methodLiveness.hpp))
      // The BasicBlock class is used to represent a basic block in the
      // liveness analysis.
      class BasicBlock : public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classMethodLiveness_1_1BasicBlock.html) for details

---
## <a name="nosCKyrIZY" id="nosCKyrIZY">MethodLivenessResult</a>

### 概要(Summary)
MethodLiveness による解析結果を格納するためのクラス.

内部的にはビットマップになっている.


```
    ((cite: hotspot/src/share/vm/compiler/methodLiveness.hpp))
    class MethodLivenessResult : public BitMap {
```




### 詳細(Details)
See: [here](../doxygen/classMethodLivenessResult.html) for details

---
## <a name="noL1ZpMy5u" id="noL1ZpMy5u">BitCounter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

MethodLiveness クラス内で使用される補助クラス.
解析結果に関する統計情報を計算するために使われる
(より具体的に言うと, MethodLivenessResult 内のビットを数えるクラス).

なお, このクラスは (デバッグ時であることに加えて) TimeLivenessAnalysis オプションが指定されている場合にしか使用されない.


```
    ((cite: hotspot/src/share/vm/compiler/methodLiveness.cpp))
    //--------------------------------------------------------------------------
    // The BitCounter class is used for counting the number of bits set in
    // some BitMap.  It is only used when collecting liveness statistics.
    
    #ifndef PRODUCT
    
    class BitCounter: public BitMapClosure {
```

### 使われ方(Usage)
MethodLiveness::get_liveness_at() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classBitCounter.html) for details

---
