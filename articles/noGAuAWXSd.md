---
layout: default
title: 同期排他処理 ： ロック解放処理 ： slow-path の処理 (1) ： Interpreter での処理
---
[Up](noqn8CuSLG.html) [Top](../index.html)

#### 同期排他処理 ： ロック解放処理 ： slow-path の処理 (1) ： Interpreter での処理

--- 
## 概要(Summary)
Interpreter (Template Interpreter, C++ Interpreter) の場合, 
fast-path が失敗すると InterpreterRuntime::monitorexit() による slow-path 処理が実行される.

ただし, 実際には InterpreterRuntime::monitorexit() 内ではほとんど処理は行っておらず,
単に ObjectSynchronizer::slow_exit にフォールバックするだけ
(See: [here](noS3vRzujM.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
InterpreterRuntime::monitorexit()
-&gt; ObjectSynchronizer::slow_exit()
   -&gt; (See: <a href="noS3vRzujM.html">here</a> for details)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterRuntime::monitorexit()
See: [here](no42301uI.html) for details






