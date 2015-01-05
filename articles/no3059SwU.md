---
layout: default
title: Interpreter に関する処理 ： 初期化処理 (3) ： Interpreter 自体の初期化処理 (2) ： Template Interpreter の場合  
---
[Up](noGVP1OhVl.html) [Top](../index.html)

#### Interpreter に関する処理 ： 初期化処理 (3) ： Interpreter 自体の初期化処理 (2) ： Template Interpreter の場合  

--- 
## 概要(Summary)
Interpreter 自体の初期化処理は, HotSpot の起動時に呼び出される Interpreter::initialize() の中で行われる. (See: [here](no3059kuA.html) for details).
Template Interpreter 使用時には Interpreter マクロ定数は TemplateInterpreter に #define されているので, 
実際に呼び出されるのは TemplateInterpreter::initialize() になる.

TemplateInterpreter::initialize() 内では TemplateInterpreter の構築処理が行われる.
実際の構築処理のほとんどは InterpreterGenerator のコンストラクタ内に実装されている.

## 備考(Notes)
InterpreterGenerator のコンストラクタ内の処理は cpu/ ごとに定義されている.
ただし, どの定義も (スーパークラスとして CppInterpreterGenerator か TemplateInterpreterGenerator のコンストラクタを呼んだ後)
TemplateInterpreterGenerator::generate_all() を呼び出すだけになっている.
(なぜ share/ 以下で共通化していない? #TODO)


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(See: <a href="no3059kuA.html">here</a> for details)
-&gt; TemplateInterpreter::initialize()
   -&gt; (1) 基底クラスの初期化メソッドを呼び出して, InvocationCounter 等の初期化を行う.
    
          -&gt; AbstractInterpreter::initialize()
             -&gt; InvocationCounter::reinitialize()
                -&gt; InvocationCounter::def()
    
      (2) 各バイトコードに対応する Template オブジェクトを生成する
          (なお, ここでは Template オブジェクトを作るだけ.
          これらの Template オブジェクトによって実際にコードが生成されるのはもう少し先)
    
          -&gt; TemplateTable::initialize()
             -&gt; TemplateTable::def()
                -&gt; Template::initialize()
             -&gt; TemplateTable::pd_initialize()
    
      (3) 実際にインタープリタを構築する
    
          -&gt; InterpreterGenerator::InterpreterGenerator()
             -&gt; TemplateInterpreterGenerator::generate_all()
    
                (1) ネイティブメソッド用の generic (slow) signature handler の生成
    
                    -&gt; AbstractInterpreterGenerator::generate_all()
                       -&gt; AbstractInterpreterGenerator::generate_slow_signature_handler()
                          -&gt; (See: <a href="no3059asZ.html">here</a> for details)
    
                (1) 実行時エラー用のコード (TOS 状態がおかしい時などに使用するコード) を生成
    
                    -&gt; TemplateInterpreterGenerator::generate_error_exit()
    
                (1) デバッグ用のバイトコードトレースコードを生成 (#ifndef PRODUCT にのみ生成)
    
                    -&gt; TemplateInterpreterGenerator::generate_trace_code()
    
                (1) return entry 用のコードを生成
    
                    -&gt; TemplateInterpreterGenerator::generate_return_entry_for()
                       -&gt; (See: <a href="no3059YlB.html">here</a> for details)
    
                (1) JVMTI の ForceEarlyReturn 処理用のコードレットを生成
    
                    -&gt; TemplateInterpreterGenerator::generate_earlyret_entry_for()
                       -&gt; (See: <a href="no3059azN.html">here</a> for details)
    
                (1) Deoptimization 処理用の deopt entry を生成
    
                    -&gt; TemplateInterpreterGenerator::generate_deopt_entry_for()
                       -&gt; (See: Deoptimization)
    
                (1) ネイティブメソッド用の result handler を作成
    
                    -&gt; TemplateInterpreterGenerator::generate_result_handler_for()
                       -&gt; (See: <a href="no3059asZ.html">here</a> for details)
    
                (1) 各種 invoke 命令 (invokevirtual, invokeinterface etc) 用の return entry を取得する
    
                    -&gt; Interpreter::return_entry()
    
                (1) ??#TODO の生成
    
                    -&gt; TemplateInterpreterGenerator::generate_continuation_for()
    
                (1) Template Interpreter を Safepoint 停止させるためのコードを生成
    
                    -&gt; TemplateInterpreterGenerator::generate_safept_entry_for()
    
                (1) 例外発生時の例外送出処理やスタックの巻き戻し処理コードの生成
    
                    -&gt; TemplateInterpreterGenerator::generate_throw_exception()
                       -&gt; (See: <a href="no30593YX.html">here</a> for details)
                    -&gt; TemplateInterpreterGenerator::generate_${例外名}_handler()
                       -&gt; (See: <a href="no3059y5N.html">here</a> for details)
    
                (1) メソッドのエントリ部処理用コードの生成
    
                    -&gt; AbstractInterpreterGenerator::generate_method_entry()
                       -&gt; (See: <a href="no3059n2f.html">here</a> for details)
    
                (1) 各バイトコードに対応するコードの生成
    
                    -&gt; TemplateInterpreterGenerator::set_entry_points_for_all_bytes()
                       -&gt; (See: <a href="noEgWr8prQ.html">here</a> for details)
    
                (1) Safepoint 停止用の dispatch table(TemplateInterpreter::_safept_table) の生成
    
                    -&gt; TemplateInterpreterGenerator::set_safepoints_for_all_bytes()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateInterpreter::initialize()
See: [here](no3059ykx.html) for details
### AbstractInterpreter::initialize()
See: [here](no3059x4G.html) for details
### InvocationCounter::reinitialize()
(#Under Construction)
See: [here](no3059-CN.html) for details
### InvocationCounter::def()
See: [here](no3059LNT.html) for details
### TemplateTable::initialize()
See: [here](no3059YXZ.html) for details
### TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(), char filler)
See: [here](no3059lhf.html) for details
### TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(int arg), int arg)
See: [here](no3059yrl.html) for details
### TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(Operation op), Operation op)
See: [here](no3059_1r.html) for details
### TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(bool arg    ), bool arg)
See: [here](no3059MAy.html) for details
### TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(TosState tos), TosState tos)
See: [here](no3059-JB.html) for details
### TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(Condition cc), Condition cc)
See: [here](no3059LUH.html) for details
### Template::initialize()
See: [here](no3059YeN.html) for details
### TemplateTable::pd_initialize() (sparc の場合)
See: [here](no3059loT.html) for details
### TemplateTable::pd_initialize() (x86_64 の場合)
See: [here](no3059yyZ.html) for details

### InterpreterGenerator::InterpreterGenerator()  (sparc の場合)
See: [here](no3059_8f.html) for details
### InterpreterGenerator::InterpreterGenerator()  (x86_64 の場合)
See: [here](no3059MHm.html) for details
### TemplateInterpreterGenerator::TemplateInterpreterGenerator()
See: [here](no3059ZRs.html) for details
### TemplateInterpreterGenerator::generate_all()
See: [here](no3059mby.html) for details
### AbstractInterpreterGenerator::generate_all()
See: [here](no3059_DU.html) for details
### TemplateInterpreterGenerator::generate_error_exit()
See: [here](no3059MOa.html) for details
### TemplateInterpreterGenerator::generate_trace_code()  (sparc の場合)
See: [here](no3059ZYg.html) for details
### TemplateInterpreterGenerator::generate_trace_code()  (x86_64 の場合)
See: [here](no3059mim.html) for details
### SharedRuntime::trace_bytecode()
See: [here](no2747Ixf.html) for details
### TemplateInterpreter::return_entry()
See: [here](no3059zss.html) for details
### TemplateInterpreterGenerator::generate_continuation_for()
(#Under Construction)

### TemplateInterpreterGenerator::generate_safept_entry_for()  (sparc の場合)
See: [here](no3059A3y.html) for details
### TemplateInterpreterGenerator::generate_safept_entry_for()  (x86_64 の場合)
See: [here](no3059yAC.html) for details
### TemplateInterpreterGenerator::set_safepoints_for_all_bytes()
See: [here](no3059ZfU.html) for details






