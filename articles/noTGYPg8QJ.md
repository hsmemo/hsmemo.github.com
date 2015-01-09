---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (1) 閾値のチェック処理 ： メソッドの呼び出し回数のチェック処理 ： Template Interpreter の場合
---
[Up](nolR92Ajmf.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (1) 閾値のチェック処理 ： メソッドの呼び出し回数のチェック処理 ： Template Interpreter の場合

--- 
## 概要(Summary)
Template Interpreter の場合, メソッドの呼び出し回数のチェックは以下の箇所で行われる.

  * ネイティブメソッドではない場合: 

    InterpreterGenerator::generate_normal_entry() が生成したコード (See: [here](no2935G1h.html) for details)

  * ネイティブメソッドの場合: 

    InterpreterGenerator::generate_native_entry() が生成したコード (See: [here](no3059asZ.html) for details)

アーキテクチャ(sparc, x86)によって若干処理に違いはあるが, 
どちらも InterpreterGenerator::generate_counter_incr() が生成したコードでカウンタ値をチェックし, 
閾値を超えていれば InterpreterGenerator::generate_counter_overflow() が生成したコードが
InterpreterRuntime::frequency_counter_overflow() を呼び出す.
これにより CompilationPolicy オブジェクトが呼び出され, JIT コンパイル処理が開始される.


## 処理の流れ (概要)(Execution Flows : Summary)
### 実行回数が閾値を越えたメソッド (非 native method) の場合
<div class="flow-abst"><pre>
(See: <a href="no2935G1h.html">here</a> for details)
-&gt; InterpreterGenerator::generate_normal_entry() が生成したコード
   -&gt; (1) JIT コンパイラを使う場合は, invocation counter 値を増加させて値をチェックする
          -&gt; InterpreterGenerator::generate_counter_incr() が生成したコード
             -&gt; * TieredCompilation が有効な場合:
                  -&gt; (1) methodDataOopDesc が付いていれば (&amp; ProfileInterpreter が有効ならば), 
                         methodDataOopDesc 内の _invocation_counter のカウンタ値をインクリメントしてチェックする
                         -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
                     (1) そうでなければ, 
                         methodOopDesc 内の _invocation_counter のカウンタ値をインクリメントしてチェックする
                         -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
                * TieredCompilation が無効な場合:
                  -&gt; (1) methodOopDesc 内の _invocation_counter のカウンタ値をインクリメントする
                     (1) ProfileInterpreter が有効ならば, 
                         methodOopDesc 内の _interpreter_invocation_count のカウンタ値をインクリメントする
                     (1) methodOopDesc 内の _invocation_counter のカウンタ値 (+ _backedge_counter のカウンタ値) をチェックする

      (1) チェックした値が JIT コンパイル対象になる閾値を超えていれば, CompilationPolicy オブジェクトを呼び出す
          -&gt; InterpreterGenerator::generate_counter_overflow() が生成したコード
             -&gt; InterpreterRuntime::frequency_counter_overflow()
                -&gt; InterpreterRuntime::frequency_counter_overflow_inner()
                   -&gt; CompilationPolicy::event()  (を各サブクラスがオーバーライドしたもの)
                      -&gt; (See: <a href="noi9gh3rMo.html">here</a> for details)

      (1) チェックした値が監視対象になる閾値を超えてい (るのにまだ mdp が確保されていなけ) れば, 新しい methodDataOop オブジェクトを methodOop 内にセットする.
          (ただしこの処理は, TieredCompilation が無効 かつ ProfileInterpreter が有効な場合のみ実行される)
          -&gt; InterpreterRuntime::profile_method()
</pre></div>

### 実行回数が閾値を越えたメソッド (native method) の場合
<div class="flow-abst"><pre>
(See: <a href="noPQtTMmO9.html">here</a> for details)
-&gt; InterpreterGenerator::generate_native_entry() が生成したコード
   -&gt; (1) JIT コンパイラを使う場合は, invocation counter 値を増加させて値をチェックする
          -&gt; InterpreterGenerator::generate_counter_incr() が生成したコード
             -&gt; (同上)

      (1) チェックした値が JIT コンパイル対象になる閾値を超えていれば, CompilationPolicy オブジェクトを呼び出す
          -&gt; InterpreterGenerator::generate_counter_overflow() が生成したコード
             -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterGenerator::generate_normal_entry() (sparc の場合)
See: [here](no3718gDz.html) for details
### InterpreterGenerator::generate_counter_incr() (sparc の場合)
See: [here](no3059SDH.html) for details
### InterpreterMacroAssembler::increment_invocation_counter() (sparc の場合)
See: [here](no10981BFy.html) for details
### InterpreterGenerator::generate_counter_overflow() (sparc の場合)
See: [here](no10981wpo.html) for details
### InterpreterRuntime::frequency_counter_overflow()
See: [here](no10981PY0.html) for details
### InterpreterRuntime::frequency_counter_overflow_inner()
See: [here](no10981Pmc.html) for details

### InterpreterGenerator::generate_normal_entry() (x86-64 の場合)
See: [here](no3059T2l.html) for details
### InterpreterGenerator::generate_counter_incr() (x86_64 の場合)
See: [here](no3059Htm.html) for details
### InterpreterGenerator::generate_counter_overflow() (x86_64 の場合)
See: [here](no10981wi0.html) for details






