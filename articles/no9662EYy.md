---
layout: default
title: 同期排他処理 ： ロック確保処理 ： fast-path の処理 ： synchronized method のエントリ部の処理 ： Template Interpreter での処理  
---
[Up](noCvhNUCUL.html) [Top](../index.html)

#### 同期排他処理 ： ロック確保処理 ： fast-path の処理 ： synchronized method のエントリ部の処理 ： Template Interpreter での処理  

--- 
## 概要(Summary)
処理は以下の関数が生成するコードによって行われる. また CPU 種別によって生成されるコードは異なる.

  * ネイティブメソッドではない場合: 

    InterpreterGenerator::generate_normal_entry() (See: [here](no2935G1h.html) for details)

  * ネイティブメソッドの場合: 

    InterpreterGenerator::generate_native_entry() (See: [here](no3059asZ.html) for details)

ただしどの場合も処理の流れは同じ.
具体的には, BasicObjectLock を確保して oop を書き込んだ後, 
InterpreterMacroAssembler::lock_object() が生成するコードでロック確保を行う.

確保が失敗した場合は, InterpreterRuntime::monitorenter() による slow-path 処理にフォールバックする (See: [here](no9662DsH.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
#### native method ではない場合
<div class="flow-abst"><pre>
InterpreterGenerator::generate_normal_entry() が生成したコード (See: <a href="no2935G1h.html">here</a> for details)
-&gt; InterpreterGenerator::lock_method() が生成したコード
   -&gt; InterpreterMacroAssembler::lock_object() が生成したコード
      -&gt; MacroAssembler::biased_locking_enter() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -&gt; InterpreterRuntime::monitorenter()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -&gt; (See: <a href="no9662DsH.html">here</a> for details)
</pre></div>

#### native method の場合
<div class="flow-abst"><pre>
InterpreterGenerator::generate_native_entry() が生成したコード (See: <a href="no3059asZ.html">here</a> for details)
-&gt; InterpreterGenerator::lock_method() が生成したコード
   -&gt; (同上)
</pre></div>

### x86_64 の場合
#### native method ではない場合
<div class="flow-abst"><pre>
InterpreterGenerator::generate_normal_entry() が生成したコード (See: <a href="no2935G1h.html">here</a> for details)
-&gt; InterpreterGenerator::lock_method() が生成したコード
   -&gt; InterpreterMacroAssembler::lock_object() が生成したコード
      -&gt; MacroAssembler::biased_locking_enter() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -&gt; InterpreterRuntime::monitorenter()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -&gt; (See: <a href="no9662DsH.html">here</a> for details)
</pre></div>

#### native method の場合
<div class="flow-abst"><pre>
InterpreterGenerator::generate_native_entry() が生成したコード (See: <a href="no3059asZ.html">here</a> for details)
-&gt; InterpreterGenerator::lock_method() が生成したコード
   -&gt; (同上)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterGenerator::lock_method() (sparc の場合)
See: [here](no4230Cky.html) for details
### InterpreterMacroAssembler::add_monitor_to_stack() (sparc の場合)
See: [here](no4230B4H.html) for details
### InterpreterMacroAssembler::lock_object() (sparc の場合)
See: [here](no4230OCO.html) for details
### MacroAssembler::biased_locking_enter()  (sparc の場合)
See: [here](no28916A4o.html) for details

### InterpreterGenerator::lock_method() (x86_64 の場合)
See: [here](no4230bMU.html) for details
### InterpreterMacroAssembler::lock_object() (x86_64 の場合)
See: [here](no4230oWa.html) for details
### MacroAssembler::biased_locking_enter() (x86_64 の場合)
See: [here](no28916aM1.html) for details






