---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： C++ Interpreter の処理中での Safepoint 停止処理
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： C++ Interpreter の処理中での Safepoint 停止処理

--- 
## 概要(Summary)
C++ Interpreter の場合, 以下の箇所で明示的な SAFEPOINT check を行っている模様.
(<= これらが SAFEPOINT マクロの使用点)

* メソッドのエントリ時
* return 時
* backward branch 時


## 処理の流れ (概要)(Execution Flows : Summary)
### メソッド呼び出し時(invoke* バイトコード), およびリターン時(*return バイトコード)
<div class="flow-abst"><pre>
BytecodeInterpreter::run() もしくは BytecodeInterpreter::runWithChecks()
-&gt; SAFEPOINT マクロ
   -&gt; SafepointSynchronize::is_synchronizing()
   -&gt; SafepointSynchronize::block()
</pre></div>

### backward branch 時
<div class="flow-abst"><pre>
BytecodeInterpreter::run() もしくは BytecodeInterpreter::runWithChecks()
-&gt; DO_BACKEDGE_CHECKS マクロ
   -&gt; SAFEPOINT マクロ
      -&gt; (同上)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### SAFEPOINT マクロ
See: [here](no78824AE.html) for details
### DO_BACKEDGE_CHECKS マクロ
See: [here](no7882FLK.html) for details






