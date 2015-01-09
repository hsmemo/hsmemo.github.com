---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： -Xcomp (-XX：-UseInterpreter), -XX：+AlwaysCompileLoopMethods による開始処理
---
[Up](no6HzyuMVW.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： -Xcomp (-XX：-UseInterpreter), -XX：+AlwaysCompileLoopMethods による開始処理

--- 
## 概要(Summary)
以下のコマンドラインオプションが指定されている場合, JIT コンパイラの開始契機が増加する.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table10981aLD -->
| Option | Description |
|---|---|
| -Xcomp (または -XX:-UseInterpreter) | 全てのメソッドを実行前にコンパイルする |
| -XX:+AlwaysCompileLoopMethods | ループを持っているメソッドについては、全て実行前にコンパイルする |
<!-- END RECEIVE ORGTBL table10981aLD -->

<!-- 
#+ORGTBL: SEND table10981aLD orgtbl-to-gfm :no-escape t
| Option                              | Description                                                      |
|-------------------------------------+------------------------------------------------------------------|
| -Xcomp (または -XX:-UseInterpreter) | 全てのメソッドを実行前にコンパイルする                           |
| -XX:+AlwaysCompileLoopMethods       | ループを持っているメソッドについては、全て実行前にコンパイルする |
-->
    
内部的には, これらが指定されていると CompilationPolicy::must_be_compiled() が true を返すようになる.
これにより, Constant Pool 中のメソッドの resolve 時やランタイムによるメソッドの呼び出し時に JIT コンパイルが開始される.

## 処理の流れ (概要)(Execution Flows : Summary)
### ConstantPool の resolve 時
<div class="flow-abst"><pre>
(See: <a href="no7882NqI.html">here</a> for details)
-&gt; CallInfo::set_common()
   -&gt; CompileBroker::compile_method()
      -&gt; (See: <a href="nobV3Ayv16.html">here</a> for details)
</pre></div>

### ランタイムによるメソッド呼び出し時
<div class="flow-abst"><pre>
(See: <a href="no3059iJu.html">here</a> for details)
-&gt; JavaCalls::call_helper()
   -&gt; CompileBroker::compile_method()
      -&gt; (See: <a href="nobV3Ayv16.html">here</a> for details)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### CallInfo::set_common()
See: [here](no10981lFT.html) for details
### CompilationPolicy::must_be_compiled()
See: [here](no3059I1h.html) for details
### JavaCalls::call_helper()
See: [here](no30597qb.html) for details






