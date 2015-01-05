---
layout: default
title: 同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： 例外による unwind 時 ： Template Interpreter での処理
---
[Up](noA8GKT6vy.html) [Top](../index.html)

#### 同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： 例外による unwind 時 ： Template Interpreter での処理

--- 
## 概要(Summary)
処理は TemplateInterpreterGenerator::generate_throw_exception() が生成するコードによって行われる. 
なお CPU 種別によって生成されるコードは異なる (See: [here](no30593YX.html) for details).

ただしどの場合も処理の流れは同じ.
具体的には, InterpreterMacroAssembler::unlock_object() が生成するコードでロック解放を行う.

解放が fast-path で終わらない場合は, InterpreterRuntime::monitorexit() による slow-path 処理にフォールバックする
(See: [here](noGAuAWXSd.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
<div class="flow-abst"><pre>
TemplateInterpreterGenerator::generate_throw_exception() が生成したコード (See: <a href="no30593YX.html">here</a> for details)
-&gt; InterpreterMacroAssembler::unlock_if_synchronized_method() が生成したコード
   -&gt; InterpreterMacroAssembler::unlock_object() が生成したコード
      -&gt; MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -&gt; InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -&gt; (See: <a href="noGAuAWXSd.html">here</a> for details)
</pre></div>

### x86_64 の場合
<div class="flow-abst"><pre>
TemplateInterpreterGenerator::generate_throw_exception() が生成したコード (See: <a href="no30593YX.html">here</a> for details)
-&gt; InterpreterMacroAssembler::remove_activation() が生成したコード (See: <a href="no30590Am.html">here</a> for details)
   -&gt; InterpreterMacroAssembler::unlock_object() が生成したコード
      -&gt; MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -&gt; InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -&gt; (See: <a href="noGAuAWXSd.html">here</a> for details)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterMacroAssembler::unlock_if_synchronized_method() (sparc の場合)
See: [here](no30590Ha.html) for details
### InterpreterMacroAssembler::unlock_object() (sparc の場合)
See: [here](no4230pQt.html) for details
### MacroAssembler::biased_locking_exit() (sparc の場合)
See: [here](no28916NCv.html) for details

### InterpreterMacroAssembler::unlock_object() (x86_64 の場合)
See: [here](no4230okC.html) for details
### MacroAssembler::biased_locking_exit() (x86_64 の場合)
See: [here](no28916MWE.html) for details






