---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： C++ Interpreter での処理
---
[Up](no2114EV0.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： C++ Interpreter での処理

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
BytecodeInterpreter::run()   or  BytecodeInterpreter::runWithChecks()
-&gt; OrderAccess::release_store()   (← Barrier Set の該当箇所を dirty 化する)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)







