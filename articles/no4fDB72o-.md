---
layout: default
title: Parse 関連のクラス (InlineTree, Parse, Parse::Block, Parse::BytecodeParseHistogram)
---
[Top](../index.html)

#### Parse 関連のクラス (InlineTree, Parse, Parse::Block, Parse::BytecodeParseHistogram)

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, 高レベル中間語(Ideal)の生成処理を表すクラス.


### クラス一覧(class list)

  * [Parse](#no71uEPW4r)
  * [InlineTree](#noE0uMJyV_)
  * [Parse::Block](#no_-pqVInY)
  * [Parse::BytecodeParseHistogram](#no-EMme7IT)


---
## <a name="no71uEPW4r" id="no71uEPW4r">Parse</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

バイトコードから高レベル中間語(Ideal)への変換を行う.


```
    ((cite: hotspot/src/share/vm/opto/parse.hpp))
    //-----------------------------------------------------------------------------
    //------------------------------Parse------------------------------------------
    // Parse bytecodes, build a Graph
    class Parse : public GraphKit {
```

### 使われ方(Usage)
ParseGenerator::generate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classParse.html) for details

---
## <a name="noE0uMJyV_" id="noE0uMJyV_">InlineTree</a>

### 概要(Summary)
Parse クラス及び Compile クラス用の補助クラス(ResourceObjクラス).

メソッドのインライン展開処理用の補助クラス.
1つの InlineTree オブジェクトが 1つのメソッドに対応する.


```
    ((cite: hotspot/src/share/vm/opto/parse.hpp))
    //------------------------------InlineTree-------------------------------------
    class InlineTree : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
InlineTree::ok_to_inline() でインライン展開すべきかどうかを判定できる (?? #TODO).

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Compile オブジェクトの _ilt フィールド

* 各 InlineTree オブジェクトの _subtrees フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Compile::call_generator()
* InlineTree::build_inline_tree_root()
* InlineTree::build_inline_tree_for_callee()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Compile()
-> InlineTree::build_inline_tree_root()

Parse::Parse()
-> Parse::do_all_blocks()
   -> Parse::do_one_block()
      -> Parse::do_call()
         -> Compile::call_generator()
            -> CallGenerator::for_method_handle_inline()
               -> Compile::call_generator()
            -> InlineTree::ok_to_inline()
               -> InlineTree::build_inline_tree_for_callee()
            -> InlineTree::find_subtree_from_root()
               -> InlineTree::build_inline_tree_for_callee()

Parse::Parse()
-> InlineTree::find_subtree_from_root()  (ただし #ifndef PRODUCT 時にのみ呼び出す)
   -> (同上)

Parse::show_parse_info()  (ただし, これは #ifndef PRODUCT 時にのみ定義される関数)
-> InlineTree::find_subtree_from_root()
   -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classInlineTree.html) for details

---
## <a name="no_-pqVInY" id="no_-pqVInY">Parse::Block</a>

### 概要(Summary)
Parse クラス内で使用される補助クラス.

メソッド内の各基本ブロックの情報を管理するクラス.
1つの Parse::Block オブジェクトが 1つの基本ブロックに対応する.


```
    ((cite: hotspot/src/share/vm/opto/parse.hpp))
      // Per-block information needed by the parser:
      class Block {
```


### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Parse オブジェクトの _blocks フィールドに(のみ)格納されている.

(正確には, このフィールドは Parse::Block の配列を格納するフィールド.
この中に, その Parse 内で使用される全ての Parse::Block オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
配列用のメモリ領域は Parse::init_blocks() 内で(のみ)確保されている. 




### 詳細(Details)
See: [here](../doxygen/classParse_1_1Block.html) for details

---
## <a name="no-EMme7IT" id="no-EMme7IT">Parse::BytecodeParseHistogram</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

Parse クラス内で使用される補助クラス.
パース処理に関する統計情報を記録するためのクラス.


```
    ((cite: hotspot/src/share/vm/opto/parse.hpp))
    #ifndef PRODUCT
      // BytecodeParseHistogram collects number of bytecodes parsed, nodes constructed, and transformations.
      class BytecodeParseHistogram : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Parse オブジェクトの _parse_histogram フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
Parse::Parse() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classParse_1_1BytecodeParseHistogram.html) for details

---
