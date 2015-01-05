---
layout: default
title: Matcher クラス関連のクラス (Matcher, 及びその補助クラス(MStack))
---
[Top](../index.html)

#### Matcher クラス関連のクラス (Matcher, 及びその補助クラス(MStack))

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, 低レベル中間語(MachNode)の生成処理を表すクラス.


### クラス一覧(class list)

  * [Matcher](#noCsvio94p)
  * [MStack](#noTem9d4nJ)


---
## <a name="noCsvio94p" id="noCsvio94p">Matcher</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

高レベル中間語(Ideal)から低レベル中間語(MachNode)への変換を行う.

### 使われ方(Usage)
Compile::Code_Gen() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classMatcher.html) for details

---
## <a name="noTem9d4nJ" id="noTem9d4nJ">MStack</a>

### 概要(Summary)
Matcher クラス内で使用される補助クラス.

特殊な Node_Stack クラス.
Matcher クラスの処理対象の Node を入れておくために使用される 
(新しい処理対象が出てくるたびに Node をこのスタックに登録していく. スタックが空になったら処理が完了する).

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Matcher::xform()
* Matcher::find_shared()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
Matcher::match()
-&gt; Matcher::find_shared()
-&gt; Matcher::xform()
</pre></div>

### 内部構造(Internal structure)
実際のスタックとしての機能はスーパークラスである Node_Stack によって実現されている.




### 詳細(Details)
See: [here](../doxygen/classMStack.html) for details

---
