---
layout: default
title: JIT Compiler ： 補足 ： C2 JIT コンパイラ用の前処理 ： ADLC による AD ファイルのコンパイル処理
---
[Up](noVxQtU9lk.html) [Top](../index.html)

#### JIT Compiler ： 補足 ： C2 JIT コンパイラ用の前処理 ： ADLC による AD ファイルのコンパイル処理

--- 
## 概要(Summary)
adlc コマンドにより、AD ファイルから C++ コードが生成される.

## 備考(Notes)
adlc は以下のようなクラスから構成される. (#Under Construction)

* FileBuff クラスは, ADL ファイルの内容をファイルから読んでくるクラス.
* ADLParser クラスは, FileBuff クラスが読んだ内容をパースするクラス.
* ADLParser クラスのパース処理の結果として, ArchDesc クラス(パース結果を格納するクラス)のオブジェクトが生成される.

  (なお, ADLParser クラスがパースした段階ではそれぞれの要素が ArchDesc クラス内に分類して納められている段階.
  main() 関数内でその後に呼び出される ArchDesc::generateMatchLists() 等の中で match rule から DFA 等を作る作業が行われている模様)

* ADL ファイル中のそれぞれの要素は
  パースによって Form クラス(のそれぞれの要素が対応するサブクラス)のオブジェクトになり,
  ArchDesc オブジェクト内に格納されている.

* OutputMap クラス(というか, そのサブクラス(?))は, 結果を cpp/hpp ファイルに書き出す際の補助クラス?

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
main()
-&gt; コマンドラインオプションの処理
-&gt; ADLParser::parse()
-&gt; ArchDesc::generateMatchLists()
-&gt; ArchDesc::identify_unique_operands()
-&gt; ArchDesc::identify_cisc_spill_instructions()
-&gt; ArchDesc::identify_short_branches()
-&gt; ファイルへの出力処理
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### main()
See: [here](no4230RMX.html) for details
### ADLParser::parse()
(#Under Construction)

### ArchDesc::generateMatchLists()
(#Under Construction)

### ArchDesc::identify_unique_operands()
(#Under Construction)

### ArchDesc::identify_cisc_spill_instructions()
(#Under Construction)

### ArchDesc::identify_short_branches()
(#Under Construction)






