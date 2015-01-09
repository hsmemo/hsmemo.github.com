---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (1) 閾値のチェック処理 ： ループの実行回数のチェック処理 ： Template Interpreter の場合
---
[Up](no2935sgV.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (1) 閾値のチェック処理 ： ループの実行回数のチェック処理 ： Template Interpreter の場合

--- 
## 概要(Summary)
Template Interpreter の場合, ループの実行回数のチェックは以下のバイトコード(に対応するテンプレート)で行われる.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table10981JDh -->
| Bytecode | Template |
|---|---|
| ifeq | TemplateTable::if_0cmp() |
| ifne | TemplateTable::if_0cmp() |
| iflt | TemplateTable::if_0cmp() |
| ifge | TemplateTable::if_0cmp() |
| ifgt | TemplateTable::if_0cmp() |
| ifle | TemplateTable::if_0cmp() |
| if_icmpeq | TemplateTable::if_icmp() |
| if_icmpne | TemplateTable::if_icmp() |
| if_icmplt | TemplateTable::if_icmp() |
| if_icmpge | TemplateTable::if_icmp() |
| if_icmpgt | TemplateTable::if_icmp() |
| if_icmple | TemplateTable::if_icmp() |
| if_acmpeq | TemplateTable::if_acmp() |
| if_acmpne | TemplateTable::if_acmp() |
| goto | TemplateTable::_goto() |
| ifnull | TemplateTable::if_nullcmp() |
| ifnonnull | TemplateTable::if_nullcmp() |
| goto_w | TemplateTable::goto_w() |
<!-- END RECEIVE ORGTBL table10981JDh -->

<!-- 
#+ORGTBL: SEND table10981JDh orgtbl-to-gfm :no-escape t
| Bytecode  | Template                    |
|-----------+-----------------------------|
| ifeq      | TemplateTable::if_0cmp()    |
| ifne      | TemplateTable::if_0cmp()    |
| iflt      | TemplateTable::if_0cmp()    |
| ifge      | TemplateTable::if_0cmp()    |
| ifgt      | TemplateTable::if_0cmp()    |
| ifle      | TemplateTable::if_0cmp()    |
| if_icmpeq | TemplateTable::if_icmp()    |
| if_icmpne | TemplateTable::if_icmp()    |
| if_icmplt | TemplateTable::if_icmp()    |
| if_icmpge | TemplateTable::if_icmp()    |
| if_icmpgt | TemplateTable::if_icmp()    |
| if_icmple | TemplateTable::if_icmp()    |
| if_acmpeq | TemplateTable::if_acmp()    |
| if_acmpne | TemplateTable::if_acmp()    |
| goto      | TemplateTable::_goto()      |
| ifnull    | TemplateTable::if_nullcmp() |
| ifnonnull | TemplateTable::if_nullcmp() |
| goto_w    | TemplateTable::goto_w()     |
-->

アーキテクチャ(sparc, x86)によって若干処理に違いはあるが, 
どちらも TemplateTable::branch() が生成したコードでカウンタ値をチェックし, 
閾値を超えていれば InterpreterRuntime::frequency_counter_overflow() を呼び出す.
これにより CompilationPolicy オブジェクトが呼び出され, JIT コンパイル処理が開始される.

## 備考(Notes)
なお, 制御の移行に関するバイトコード(See: [here](noS59wryRf.html) for details)のうち, 
以下のものについては Template Interpreter は実行回数をチェックしていない.

(<= まぁ実際問題として, tableswitch や lookupswitch でループしたり  jsr や ret で再帰するプログラムはほぼないだろ...)

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table10981Qmi -->
| Bytecode | Template |
|---|---|
| jsr | TemplateTable::jsr() |
| ret | TemplateTable::ret() |
| tableswitch | TemplateTable::tableswitch() |
| lookupswitch | TemplateTable::lookupswitch() |
| jsr_w | TemplateTable::jsr_w() |
| ret | TemplateTable::wide_ret() |
| fast_linearswitch | TemplateTable::fast_linearswitch() |
| fast_binaryswitch | TemplateTable::fast_binaryswitch() |
<!-- END RECEIVE ORGTBL table10981Qmi -->

<!-- 
#+ORGTBL: SEND table10981Qmi orgtbl-to-gfm :no-escape t
| Bytecode          | Template                           |
|-------------------+------------------------------------|
| jsr               | TemplateTable::jsr()               |
| ret               | TemplateTable::ret()               |
| tableswitch       | TemplateTable::tableswitch()       |
| lookupswitch      | TemplateTable::lookupswitch()      |
| jsr_w             | TemplateTable::jsr_w()             |
| ret               | TemplateTable::wide_ret()          |
| fast_linearswitch | TemplateTable::fast_linearswitch() |
| fast_binaryswitch | TemplateTable::fast_binaryswitch() |
-->

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
TemplateTable::if_0cmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード (における jsr 以外のためのコード生成パス) (See: <a href="noS59wryRf.html">here</a> for details)
      -&gt; * UseCompiler と UseLoopCounter のどちらかが指定されていない場合:
           -&gt; (1) Lbcp の値を変更する
              (2) 次のバイトコードに対応するテンプレートへとジャンプする
                  -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

         * UseCompiler と UseLoopCounter の両方が指定されている場合:
           -&gt; * TieredCompilation が有効な場合:
                -&gt; (1) ブランチの方向が backward でなければ, (4) に進む
                   (2) カウンタ値をインクリメントしてチェックする.
                       閾値を超えていれば, (3) に進む.
                       閾値を超えてなければ, (4) に進む.
                       * methodDataOopDesc が付いていれば (&amp; ProfileInterpreter が有効ならば), 
                         methodDataOopDesc 内の _backedge_counter のカウンタ値をインクリメントしてチェックする.
                         -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
                       * そうでなければ, 
                         methodOopDesc 内の _backedge_counter のカウンタ値をインクリメントしてチェックする
                         -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
                   (3) JIT コンパイルを行う. あるいは, 既に JIT コンパイル済みのコードがあれば, OSR を行ってそのコードに遷移する.
                       (遷移した場合は以降の処理は実行されない)
                       -&gt; InterpreterRuntime::frequency_counter_overflow()
                          -&gt; InterpreterRuntime::frequency_counter_overflow_inner()
                             -&gt; CompilationPolicy::event()  (を各サブクラスがオーバーライドしたもの)
                                -&gt; (See: <a href="noi9gh3rMo.html">here</a> for details)
                       -&gt; SharedRuntime::OSR_migration_begin()
                          -&gt; (See: <a href="norlT6uoVb.html">here</a> for details)
                   (4) Lbcp の値を変更する
                   (5) 次のバイトコードに対応するテンプレートへとジャンプする
                       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード
   
              * TieredCompilation が無効な場合:
                -&gt; (1) ブランチの方向が backward でなければ, (5) に進む
                   (2) methodOopDesc 内の _backedge_counter のカウンタ値をインクリメントする
                       -&gt; InterpreterMacroAssembler::increment_backedge_counter() が生成したコード
                   (3) ProfileInterpreter が有効ならば, 必要に応じて methodDataOop の生成を行う
                       -&gt; InterpreterMacroAssembler::test_invocation_counter_for_mdp() が生成したコード
                   (4) カウンタ値と閾値を比較し, 閾値を超えていれば JIT コンパイルを行う.
                       あるいは, 既に JIT コンパイル済みのコードがあれば, OSR を行ってそのコードに遷移する (遷移した場合は以降の処理は実行されない)
                       -&gt; InterpreterMacroAssembler::test_backedge_count_for_osr() が生成したコード
                          -&gt; InterpreterRuntime::frequency_counter_overflow()
                             -&gt; (同上)
                          -&gt; SharedRuntime::OSR_migration_begin()
                             -&gt; (See: <a href="norlT6uoVb.html">here</a> for details)
                   (5) Lbcp の値を変更する
                   (6) 次のバイトコードに対応するテンプレートへとジャンプする
                       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード
   
TemplateTable::if_icmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::if_acmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::_goto() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::if_nullcmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::goto_w() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)
</pre></div>


### x86_64 の場合
<div class="flow-abst"><pre>
TemplateTable::if_0cmp() が生成したコード  (See: <a href="noS59wryRf.html">here</a> for details)
-&gt; TemplateTable::branch() が生成したコード (における jsr 以外のためのコード生成パス) (See: <a href="noS59wryRf.html">here</a> for details)
   -&gt; * UseLoopCounter が指定されていない場合:
        -&gt; (1) r13 の値を変更し, rbx に次のバイトコードのセットする
           (2) 次のバイトコードに対応するテンプレートへとジャンプする
               -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード

      * UseLoopCounter が指定されている場合:
        -&gt; * TieredCompilation が有効な場合:
             -&gt; (1) r13 の値を変更する
                (2) ブランチの方向が backward でなければ, (5) に進む
                (3) カウンタ値をインクリメントしてチェックする.
                    閾値を超えていれば, (4) に進む.
                    閾値を超えてなければ, (5) に進む.
                    * methodDataOopDesc が付いていれば (&amp; ProfileInterpreter が有効ならば), 
                      methodDataOopDesc 内の _backedge_counter のカウンタ値をインクリメントしてチェックする.
                      -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
                    * そうでなければ, 
                      methodOopDesc 内の _backedge_counter のカウンタ値をインクリメントしてチェックする
                      -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
                (4) JIT コンパイルを行う. あるいは, 既に JIT コンパイル済みのコードがあれば, OSR を行ってそのコードに遷移する.
                    (遷移した場合は以降の処理は実行されない)
                    -&gt; InterpreterRuntime::frequency_counter_overflow()
                       -&gt; InterpreterRuntime::frequency_counter_overflow_inner()
                          -&gt; CompilationPolicy::event()  (を各サブクラスがオーバーライドしたもの)
                             -&gt; (See: <a href="noi9gh3rMo.html">here</a> for details)
                    -&gt; SharedRuntime::OSR_migration_begin()
                       -&gt; (See: <a href="norlT6uoVb.html">here</a> for details)
                (5) rbx に次のバイトコードのセットする
                (6) 次のバイトコードに対応するテンプレートへとジャンプする
                    -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード

           * TieredCompilation が無効な場合:
             -&gt; (1) r13 の値を変更する
                (2) ブランチの方向が backward でなければ, (6) に進む
                (3) methodOopDesc 内の _backedge_counter のカウンタ値をインクリメントする
                    -&gt; InterpreterMacroAssembler::increment_backedge_counter() が生成したコード
                (4) ProfileInterpreter が有効ならば, 必要に応じて methodDataOop の生成を行う
                    -&gt; InterpreterMacroAssembler::test_invocation_counter_for_mdp() が生成したコード
                (5) カウンタ値と閾値を比較し, 閾値を超えていれば JIT コンパイルを行う.
                    あるいは, 既に JIT コンパイル済みのコードがあれば, OSR を行ってそのコードに遷移する (遷移した場合は以降の処理は実行されない)
                    -&gt; InterpreterMacroAssembler::test_backedge_count_for_osr() が生成したコード
                       -&gt; InterpreterRuntime::frequency_counter_overflow()
                          -&gt; (同上)
                       -&gt; SharedRuntime::OSR_migration_begin()
                          -&gt; (See: <a href="norlT6uoVb.html">here</a> for details)
                (6) rbx に次のバイトコードのセットする
                (7) 次のバイトコードに対応するテンプレートへとジャンプする
                    -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード
   
TemplateTable::if_icmp() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::if_acmp() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::_goto() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::if_nullcmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::goto_w() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateTable::if_0cmp() (sparc の場合)
See: [here](no10981XKl.html) for details
### InterpreterMacroAssembler::if_cmp() (sparc の場合)
See: [here](no10981RZM.html) for details
### TemplateTable::if_icmp() (sparc の場合)
See: [here](no1098198M.html) for details
### TemplateTable::if_acmp() (sparc の場合)
See: [here](no10981MCt.html) for details
### TemplateTable::_goto() (sparc の場合)
See: [here](no10981AAi.html) for details
### TemplateTable::if_nullcmp() (sparc の場合)
See: [here](no10981ZMz.html) for details
### TemplateTable::goto_w() (sparc の場合)
See: [here](no10981ne0.html) for details
### TemplateTable::branch() (sparc の場合)
See: [here](no10981cyM.html) for details
### InterpreterMacroAssembler::increment_backedge_counter() (sparc の場合)
See: [here](no109817Nm.html) for details
### InterpreterMacroAssembler::test_invocation_counter_for_mdp() (sparc の場合)
See: [here](no10981v9y.html) for details
### InterpreterMacroAssembler::test_backedge_count_for_osr() (sparc の場合)
See: [here](no10981SYp.html) for details
### InterpreterRuntime::frequency_counter_overflow()
See: [here](no10981PY0.html) for details
### InterpreterRuntime::frequency_counter_overflow_inner()
See: [here](no10981Pmc.html) for details

### TemplateTable::if_0cmp() (x86_64 の場合)          
See: [here](no1098167W.html) for details
### TemplateTable::if_icmp() (x86_64 の場合)          
See: [here](no1098171p.html) for details
### TemplateTable::if_acmp() (x86_64 の場合)          
See: [here](no10981VK2.html) for details
### TemplateTable::_goto() (x86_64 の場合)
See: [here](no10981AAi.html) for details
### TemplateTable::if_nullcmp() (x86_64 の場合)       
See: [here](no10981IAw.html) for details
### TemplateTable::goto_w() (x86_64 の場合)
See: [here](no10981ne0.html) for details
### TemplateTable::branch() (x86_64 の場合)
See: [here](no10981VfS.html) for details






