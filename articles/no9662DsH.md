---
layout: default
title: 同期排他処理 ： ロック確保処理 ： slow-path の処理 (1) ： Interpreter での処理  
---
[Up](no7zlkLkfb.html) [Top](../index.html)

#### 同期排他処理 ： ロック確保処理 ： slow-path の処理 (1) ： Interpreter での処理  

--- 
## 概要(Summary)
Interpreter (Template Interpreter, C++ Interpreter) の場合, 
fast-path が失敗すると InterpreterRuntime::monitorenter() による slow-path 処理が実行される.

ただし, 実際には InterpreterRuntime::monitorenter() 内ではほとんど処理は行っておらず,
単に ObjectSynchronizer::fast_enter() または ObjectSynchronizer::slow_enter() にフォールバックするだけ
(See: [here](no96623Ns.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
InterpreterRuntime::monitorenter()
-&gt; UseBiasedLocking オプションに応じてどちらかを呼び出す
   * UseBiasedLocking オプションが指定されている場合:
     -&gt; ObjectSynchronizer::fast_enter()
        -&gt; (See: <a href="no96623Ns.html">here</a> for details)
   * 〃 が指定されていない場合:
     -&gt; ObjectSynchronizer::slow_enter()
        -&gt; (See: <a href="no96623Ns.html">here</a> for details)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterRuntime::monitorenter()
See: [here](no4230P1s.html) for details






