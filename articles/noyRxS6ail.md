---
layout: default
title: Method に関する処理 ： Java のコードによるメソッド呼び出し処理 (3) ： 呼び出し先(callee 側)での method entry 処理 ： Template Interpreter の場合 ： 特殊なメソッドの場合
---
[Up](noQH79ZxNb.html) [Top](../index.html)

#### Method に関する処理 ： Java のコードによるメソッド呼び出し処理 (3) ： 呼び出し先(callee 側)での method entry 処理 ： Template Interpreter の場合 ： 特殊なメソッドの場合

--- 
## 概要(Summary)
InterpreterGenerator::generate_normal_entry() と
InterpreterGenerator::generate_method_handle_entry() 以外で生成される特殊なメソッドの場合,
エントリ部だけで処理を全て行ってしまい, そのままリターンする
(このため, 行う処理としてはリターン時に行う内容まで含んでいる).


## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
<div class="flow-abst"><pre>
InterpreterGenerator::generate_abstract_entry() が生成したコード
-&gt; MacroAssembler::call_VM() が生成したコード
   -&gt; (See: <a href="no2935dSX.html">here</a> for details)
      -&gt; InterpreterRuntime::throw_AbstractMethodError()

InterpreterGenerator::generate_accessor_entry() が生成したコード
-&gt; * 高速処理が可能な場合:
     -&gt; (この生成コード中でフィールドにアクセスしてリターン)
   * 〃が不可能な場合:
     -&gt; InterpreterGenerator::generate_normal_entry() が生成したコード

InterpreterGenerator::generate_empty_entry() が生成したコード
-&gt; * Safepoint 停止要求が出ていない場合:
     -&gt; (リターンするだけ)
   * 〃が出ていた場合:
     -&gt; InterpreterGenerator::generate_normal_entry() が生成したコード

InterpreterGenerator::generate_Reference_get_entry() が生成したコード
-&gt; * G1GC を使用していて, かつ receiver が NULL ではない場合:
     -&gt; MacroAssembler::g1_write_barrier_pre() が生成したコード
     -&gt; (この生成コード中でフィールドにアクセスしてリターン)
   * G1GC を使用していて, かつ receiver が NULL の場合:
     -&gt; InterpreterGenerator::generate_normal_entry() が生成したコード
   * G1GC ではない場合:
     -&gt; InterpreterGenerator::generate_accessor_entry() が生成したコード
</pre></div>


### x86_64 の場合
<div class="flow-abst"><pre>
InterpreterGenerator::generate_abstract_entry() が生成したコード
-&gt; MacroAssembler::call_VM() が生成したコード
   -&gt; (See: <a href="no2935dSX.html">here</a> for details)
      -&gt; InterpreterRuntime::throw_AbstractMethodError()

InterpreterGenerator::generate_accessor_entry() が生成したコード
-&gt; * 高速処理が可能な場合:
     -&gt; (この生成コード中でフィールドにアクセスしてリターン)
   * 〃が不可能な場合:
     -&gt; InterpreterGenerator::generate_normal_entry() が生成したコード

InterpreterGenerator::generate_empty_entry() が生成したコード
-&gt; * Safepoint 停止要求が出ていない場合:
     -&gt; (リターンするだけ)
   * 〃が出ていた場合:
     -&gt; InterpreterGenerator::generate_normal_entry() が生成したコード

InterpreterGenerator::generate_math_entry() が生成したコード
-&gt; (1) 対応する算術演算を行う

   (1) リターン

InterpreterGenerator::generate_Reference_get_entry() が生成したコード
-&gt; * G1GC を使用していて, かつ receiver が NULL ではない場合:
     -&gt; MacroAssembler::g1_write_barrier_pre() が生成したコード
     -&gt; (この生成コード中でフィールドにアクセスしてリターン)
   * G1GC を使用していて, かつ receiver が NULL の場合:
     -&gt; InterpreterGenerator::generate_normal_entry() が生成したコード
   * G1GC ではない場合:
     -&gt; InterpreterGenerator::generate_accessor_entry() が生成したコード
</pre></div>




## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterGenerator::generate_abstract_entry() (sparc の場合)
See: [here](no3059fUB.html) for details
### InterpreterGenerator::generate_accessor_entry() (sparc の場合)
See: [here](no30595oN.html) for details
### InterpreterGenerator::generate_empty_entry() (sparc の場合)
See: [here](no3059gAs.html) for details
### InterpreterGenerator::generate_Reference_get_entry() (sparc の場合)
See: [here](no9282n7I.html) for details

### InterpreterGenerator::generate_abstract_entry() (x86_64 の場合)
See: [here](no3059seH.html) for details
### InterpreterGenerator::generate_accessor_entry() (x86_64 の場合)
See: [here](no3059T9Z.html) for details
### InterpreterGenerator::generate_empty_entry() (x86_64 の場合)
See: [here](no3059tKy.html) for details
### InterpreterGenerator::generate_math_entry() (x86_64 の場合)
See: [here](no3059GzT.html) for details
### InterpreterGenerator::generate_Reference_get_entry() (x86_64 の場合)
See: [here](no92824XT.html) for details






