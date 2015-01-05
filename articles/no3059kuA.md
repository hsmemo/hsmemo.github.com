---
layout: default
title: Interpreter に関する処理 ： 初期化処理 (2) ： Interpreter 自体の初期化処理 (1) 
---
[Up](no3059kZk.html) [Top](../index.html)

#### Interpreter に関する処理 ： 初期化処理 (2) ： Interpreter 自体の初期化処理 (1) 

--- 
## 概要(Summary)
Interpreter 自体の初期化処理は, HotSpot の起動時に呼び出される interpreter_init() の中で行われる.

この中では最終的に Interpreter::initialize() で初期化処理が行われるが, 
Interpreter はインタープリタ種別に応じて #define された定数なので, 
実際には TemplateInterpreter::initialize() か CppInterpreter::initialize() が呼び出されることになる (See: [here](no7882AgC.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; interpreter_init()
         -&gt; Interpreter::initialize()   (&lt;= 実際には TemplateInterpreter::initialize() か CppInterpreter::initialize() が呼び出される)
            -&gt; (See: <a href="noGVP1OhVl.html">here</a> for details)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### interpreter_init()
See: [here](no3059lar.html) for details






