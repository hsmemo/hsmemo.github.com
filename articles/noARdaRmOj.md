---
layout: default
title: 同期排他処理 ： ロック確保処理 ： fast-path の処理 ： monitorexit 命令の処理 ： Template Interpreter での処理
---
[Up](noQFS71luo.html) [Top](../index.html)

#### 同期排他処理 ： ロック確保処理 ： fast-path の処理 ： monitorexit 命令の処理 ： Template Interpreter での処理

--- 
## 概要(Summary)
処理は TemplateTable::monitorexit() が生成するコードによって行われる.

CPU 種別によって生成されるコードは異なるがどちらも処理の流れは同じ.
具体的には, InterpreterMacroAssembler::unlock_object() が生成するコードでロック解放を行う.

解放が fast-path で終わらない場合は, InterpreterRuntime::monitorexit() による slow-path 処理にフォールバックする
(See: [here](noGAuAWXSd.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
<div class="flow-abst"><pre>
TemplateTable::monitorexit() が生成したコード
-&gt; InterpreterMacroAssembler::unlock_object() が生成したコード
   -&gt; MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
   -&gt; InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
      -&gt; (See: <a href="noGAuAWXSd.html">here</a> for details)
</pre></div>

### x86_64 の場合
<div class="flow-abst"><pre>
TemplateTable::monitorexit() が生成したコード
-&gt; InterpreterMacroAssembler::unlock_object() が生成したコード
   -&gt; MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
   -&gt; InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
      -&gt; (See: <a href="noGAuAWXSd.html">here</a> for details)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateTable::monitorexit() (sparc の場合)
See: [here](no4230cGn.html) for details
### InterpreterMacroAssembler::unlock_object() (sparc の場合)
See: [here](no4230pQt.html) for details
### MacroAssembler::biased_locking_exit() (sparc の場合)
See: [here](no28916NCv.html) for details

### TemplateTable::monitorexit() (x86_64 の場合)
See: [here](no42302az.html) for details
### InterpreterMacroAssembler::unlock_object() (x86_64 の場合)
See: [here](no4230okC.html) for details
### MacroAssembler::biased_locking_exit() (x86_64 の場合)
See: [here](no28916MWE.html) for details






