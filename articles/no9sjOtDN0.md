---
layout: default
title: Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： signal handler で検出する例外 ： signal handler での処理
---
[Up](noUSEjBfrf.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： signal handler で検出する例外 ： signal handler での処理

--- 
## 概要(Summary)
例外処理に使用するシグナルが生じた場合, 各 OS 用のシグナルハンドラ内では, 
SharedRuntime::continuation_for_implicit_exception() によって適切な戻りアドレスを決定している

(戻りアドレスは, 例えば Interpreter の適切な throw_*_entry() や StubRoutines の適切な throw_*_entry, 等)

最終的には, シグナルハンドラ終了後にそのアドレスに遷移することで例外ハンドラの処理にフォールバックする.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(See: <a href="no30592eE.html">here</a> for details)
-&gt; シグナルハンドラ (JVM_handle_linux_signal() or JVM_handle_solaris_signal() or topLevelExceptionFilter())
    -&gt; SharedRuntime::continuation_for_implicit_exception()
    -&gt; シグナルハンドラ終了後, SharedRuntime::continuation_for_implicit_exception() で取得したアドレスへとジャンプする.
       なお, 以下のようなアドレスが取得される.
       * Interpreter 実行のコード中の場合: ※
         * NullPointerException の場合
           TemplateInterpreter::throw_NullPointerException_entry() (= TemplateInterpreter::_throw_NullPointerException_entry が指しているコード) (See: <a href="no3059y5N.html">here</a> for details)
         * ArithmeticException の場合
           TemplateInterpreter::throw_ArithmeticException_entry() (= TemplateInterpreter::_throw_ArithmeticException_entry が指しているコード) (See: <a href="no3059y5N.html">here</a> for details)
         * StackOverflowError の場合
           TemplateInterpreter::throw_StackOverflowError_entry() (= TemplateInterpreter::_throw_StackOverflowError_entry が指しているコード) (See: <a href="no3059y5N.html">here</a> for details)
       * Interpreter 実行のコードではない場合:
         #TODO
</pre></div>

(※) ただし, C++ Interpreter の場合には signal handler 経由での例外処理は行われないので, Template Interpreter の場合のみ.


## 処理の流れ (詳細)(Execution Flows : Details)
### SharedRuntime::continuation_for_implicit_exception()
(#Under Construction)
See: [here](no3059BgI.html) for details
### TemplateInterpreter::throw_NullPointerException_entry()
See: [here](no27147L6i.html) for details
### TemplateInterpreter::throw_ArithmeticException_entry()
See: [here](no27147YLd.html) for details
### TemplateInterpreter::throw_StackOverflowError_entry()
See: [here](no27147xsK.html) for details





