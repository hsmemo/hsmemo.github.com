---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイムの呼び出し時 ： InterpreterRuntime の呼び出し時 (IRT_*)  
---
[Up](no3059XPe.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイムの呼び出し時 ： InterpreterRuntime の呼び出し時 (IRT_*)  

--- 
## 概要(Summary)
InterpreterRuntime の関数の中で
InterpreterRuntime 外から呼び出される可能性があるもの (= InterpreterRuntime 外からの entry point になるもの) については,
呼び出し時に明示的に SafepointSynchronize::_state のチェックが行われる.
この時点で Safepoint が開始されていた場合, そのスレッドはその場で停止する.

より具体的に言うと, 上記のような関数は定義時に以下のマクロを用いて定義されている.

  * IRT_ENTRY マクロ
  * IRT_END マクロ

これらのマクロが生成するコードから ThreadInVMfromJava のコンストラクタ/デストラクタが呼び出され,
その中で SafepointSynchronize::_state の値を確認している.

## 備考(Notes)
非常に簡単な InterpreterRuntime の関数用に, Safepoint チェックを行わない IRT_LEAF というマクロも用意されている.

また, InterpreterRuntime::monitorenter() と InterpreterRuntime::monitorexit() 用に
IRT_ENTRY_NO_ASYNC というマクロも用意されている.
IRT_ENTRY マクロとの違いは, ThreadInVMfromJava ではなく ThreadInVMfromJavaNoAsyncException を用いる点
(これは, JavaThread::handle_special_runtime_exit_condition() 内での
 asynchronous exception 用の処理を省略した高速版).

(ついでに IRT_ENTRY_FOR_NMETHOD というマクロも存在しているが, これは使用箇所が見当たらない...#TODO)

## 処理の流れ (概要)(Execution Flows : Summary)
### IRT_ENTRY マクロ ~ IRT_END マクロ
<div class="flow-abst"><pre>
IRT_ENTRY マクロ
-&gt; ThreadInVMfromJava::ThreadInVMfromJava()
   -&gt; (See: <a href="no8p2E6iLf.html">here</a> for details)
</pre></div>

<div class="flow-abst"><pre>
IRT_END マクロ
(IRT_ENTRY マクロで宣言されていた ThreadInVMfromJava のデストラクタが呼ばれる)
-&gt; ThreadInVMfromJava::~ThreadInVMfromJava()
   -&gt; (See: <a href="no8p2E6iLf.html">here</a> for details)
</pre></div>


### IRT_LEAF マクロ ~ IRT_END マクロ
<div class="flow-abst"><pre>
(Safepoint チェック処理はない)
</pre></div>


### IRT_ENTRY_NO_ASYNC マクロ ~ IRT_END マクロ
<div class="flow-abst"><pre>
IRT_ENTRY_NO_ASYNC マクロ
-&gt; ThreadInVMfromJavaNoAsyncException::ThreadInVMfromJavaNoAsyncException()
   -&gt; (See: <a href="no8p2E6iLf.html">here</a> for details)
</pre></div>

<div class="flow-abst"><pre>
IRT_END マクロ
(IRT_ENTRY_NO_ASYNC マクロで宣言されていた ThreadInVMfromJavaNoAsyncException のデストラクタが呼ばれる)
-&gt; ThreadInVMfromJavaNoAsyncException::~ThreadInVMfromJavaNoAsyncException()
   -&gt; (See: <a href="no8p2E6iLf.html">here</a> for details)
</pre></div>


### IRT_ENTRY_FOR_NMETHOD マクロ ~ IRT_END マクロ
<div class="flow-abst"><pre>
IRT_ENTRY_FOR_NMETHOD マクロ
-&gt; ThreadInVMfromJavaNoAsyncException::ThreadInVMfromJavaNoAsyncException()
   -&gt; (See: <a href="no8p2E6iLf.html">here</a> for details)
</pre></div>

<div class="flow-abst"><pre>
IRT_END マクロ
(IRT_ENTRY_FOR_NMETHOD マクロで宣言されていた ThreadInVMfromJavaNoAsyncException のデストラクタが呼ばれる)
-&gt; ThreadInVMfromJavaNoAsyncException::~ThreadInVMfromJavaNoAsyncException()
   -&gt; (See: <a href="no8p2E6iLf.html">here</a> for details)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### IRT_ENTRY マクロ
See: [here](no78824rn.html) for details
### __ENTRY マクロ
See: [here](no7882ScE.html) for details
### IRT_END マクロ
See: [here](no7882SHo.html) for details

### IRT_LEAF マクロ
See: [here](no7882uk2.html) for details

### IRT_ENTRY_NO_ASYNC マクロ
See: [here](no7882guF.html) for details

### IRT_ENTRY_FOR_NMETHOD マクロ
See: [here](no7882t4L.html) for details






