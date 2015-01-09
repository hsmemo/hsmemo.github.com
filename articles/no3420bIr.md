---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理 ： SimpleCompPolicy の場合 
---
[Up](noi9gh3rMo.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理 ： SimpleCompPolicy の場合 

--- 
## 概要(Summary)
SimpleCompPolicy クラスの CompilationPolicy オブジェクトの場合, 
CompilationPolicy::event() が呼ばれると NonTieredCompPolicy::event() が実行される.

JIT コンパイルを行うかどうかの判断は,
上記の関数から呼び出される SimpleCompPolicy::method_invocation_event() または SimpleCompPolicy::method_back_branch_event() で行われる.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
SimpleCompPolicy::event() (= NonTieredCompPolicy::event())
-&gt; * メソッドのコンパイルの場合:
     -&gt; SimpleCompPolicy::method_invocation_event()
        -&gt; CompileBroker::compile_method()
           -&gt; (See: <a href="nobV3Ayv16.html">here</a> for details)
   * ループのコンパイルの場合:
     -&gt; SimpleCompPolicy::method_back_branch_event()
        -&gt; CompileBroker::compile_method()
           -&gt; (See: <a href="nobV3Ayv16.html">here</a> for details)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### NonTieredCompPolicy::event()
See: [here](no6348YGY.html) for details
### SimpleCompPolicy::method_invocation_event()
See: [here](no6348sBI.html) for details
### CompilationPolicy::is_compilation_enabled()
See: [here](no6348HJz.html) for details
### SimpleCompPolicy::method_back_branch_event()
See: [here](no186823Zo.html) for details
### CompilationPolicy::can_be_compiled()
See: [here](no28564oC0.html) for details
### methodOopDesc::is_not_compilable()
See: [here](no28564uku.html) for details
### methodOopDesc::is_not_c1_compilable()
See: [here](no28564VRp.html) for details
### methodOopDesc::is_not_c2_compilable()
See: [here](no28564ibv.html) for details
### methodOopDesc::is_not_osr_compilable()
See: [here](no28564IOX.html) for details






